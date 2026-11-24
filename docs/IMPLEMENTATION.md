# Implementation audit

All cell references below are **one-based physical notebook cells** in `recovered/notebooks/HMT_Transformer_v3_6.ipynb`, not the notebook's C01/F01 labels. Inspection was static; the source was not executed or repaired.

## Data contract and transformations — cells 5, 7, 9

`ESM_filtered.csv` is the sole loaded dataset. Required metadata columns are `Mag`, `R(km)`, `Vs30(m/s)`, `Depth(km)`, `Fault` and `EventID`. The notebook labels `R(km)` as Joyner–Boore distance (Rjb); rupture-distance/Rrup claims are not verified by a data dictionary. It selects 36 explicitly named `U_Hor_*` columns. No accelerogram reading, waveform preprocessing, component combination, NGA-West merge, or 111-period resampling implementation was recovered.

Continuous inputs are magnitude, distance, `ln(R+1)`, and `(ln(Vs30)-ln(760))/ln(2)`. The six-input variant additionally uses `ln(Depth+1)`. Fault codes are decremented to zero-based classes. Targets are `log10(PSA+1e-9)`; figures treat input PSA as cm/s² and sometimes divide by 981 or 980.665 to obtain g. Dataset units cannot be independently confirmed without the CSV.

Train-only `StandardScaler` instances transform the two feature matrices and all target periods. Comments claiming the affine Vs30 re-expression preserves a spread that StandardScaler would otherwise compress are misleading: fitting another StandardScaler largely removes that affine change. Missing-value, positive-domain, category-range and duplicate-record validation are not supplied.

## Encoder and decoder — cell 11

Each continuous scalar passes through Linear(1,d), GELU, Linear(d,d), LayerNorm and dropout. Fault type uses an embedding. The five-input sequence has six tokens including CLS; the six-input sequence has eight, including CLS and an extra depth–fault context from attention over depth/fault. Learned token-position embeddings feed five pre-norm self-attention/SwiGLU blocks with d=160 and eight heads.

The decoder projects normalized log-period coordinates through an MLP, uses four-head cross-attention against the encoded feature tokens, then produces one scalar for each of 36 periods. These are coordinate-conditioned queries with shared learned projection weights, not 36 independent free query parameters. No separate source/path/site hierarchical encoder is implemented. A hierarchy should not be inferred solely from the project title. Saved parameter counts are 2,351,201 and 2,481,281 for the two variants. The architecture drawing in cell 26 incorrectly shows seven tokens for the eight-token model.

## Loss and optimization — cells 5, 13, 15

Batch loss is magnitude-weighted standardized-target MSE plus 0.50 times squared global signed mean error plus 0.05 times the mean squared second difference across ordered period indices. This is a discrete smoothness regularizer, not a physical differential equation; period spacing is nonuniform and differences are not divided by period intervals.

An optional helper samples fixed distance, Vs30 and magnitude pairs, penalizing distance increases, magnitude decreases, and opposite short-/long-period Vs30 ordering. It considers fault classes 0–2 only and uses standardized prediction coordinates. Its three named weights are summed into one common factor rather than separately weighting rules. These soft sampled preferences are not guaranteed physical constraints.

**The actual cell-15 training calls omit `scaler`, so the helper is never activated**, despite saved training messages displaying `[+mono]` after warmup. Passing the existing six-input scaler into this helper would also fail its four-feature input contract. Do not cite the log tag as evidence of active monotonicity training.

Training uses AdamW, gradient clipping at 1.0, 15 warmup epochs, cosine decay, up to 150 epochs and patience 35. EMA updates after batch steps. Validation applies EMA weights and evaluates the full batch `physics_loss`, not MSE alone. After validation it restores ordinary weights before saving `best_state`. At completion it loads that best ordinary state and immediately overwrites it with the **final EMA shadow**, so checkpoint weights are not guaranteed to correspond to the minimum validation loss. Optional monotonicity optimizer steps would not immediately update EMA. Checkpoints store weights/history/best loss but not scalers, split indices, schema or optimizer state. Existing checkpoints bypass training without provenance compatibility checks.

## Evaluation and report defects

- Cell 13 inverse-transforms predictions and computes R², RMSE and signed bias per period. Cell 17 uses six-input predictions as its primary aliases, but cells 31, 33, 35, 55 and 57 explicitly report five-input results. Keep their lineages separate.
- Training loaders shuffle rows. Paired predictions/targets remain aligned for metrics, but returned external training indices do not track that shuffled order; do not join those predictions back to event IDs by index without correcting ordering.
- `predict6` in cells 19 and 37 passes raw depth where training expects `ln(Depth+1)`. Affects sample/sensitivity plots using it, not cell-17 loader-based metrics.
- `predict5` in cell 19 passes raw `ln(Vs30)` where training expects the normalized log feature. Attenuation and magnitude figures calling this helper are not preprocessing-consistent predictions.
- Cell 39 multiplies spectra by `1+0.03*(Mw-6)`. Cell 41 clips/smooths spectra and multiplies shallow/deep predictions by 1.05/0.95. Those plots are modified illustrations, not untouched model outputs or proof of learned depth effects.
- Cell 35 computes between-event sigma from event means repeated per record, making it record-weighted, not an equal-event sample estimate.
- Cell 57's R₀² residual uses `t-K*t`, excluding actual prediction values except through K; the label should not be accepted as a validated external-standard implementation. Values are also floored at zero.
- Cell 59 is **markdown containing Python**, so the comparison chart/summary is not executable in the recovered notebook. Cell 60's completion/file list is prose, not proof files exist.

## Interpretability — cell 49

Gradient importance averages absolute derivatives of summed standardized outputs for up to 500 test examples; fault importance uses changes under class substitution. These quantities have different meanings and are mixed into a normalized bar plot.

Kernel SHAP uses 100 training background rows, 120 test rows, and 50 samples for each of period indices 0 and 19 (0.01 and 1.0 s). It explains **standardized target outputs** because `model6_predict_np` does not inverse-transform them, despite plot titles referring to log10 PSA. Correlated distance/log-distance inputs are perturbed independently. Fault is passed numerically then cast to integer. SHAP is explanatory code, not a baseline bias-reduction method, causality test, or monotonicity certificate. The cell imports `shap` unconditionally and its fallback references `bar_colors` defined only in the SHAP branch, weakening the advertised fallback.
