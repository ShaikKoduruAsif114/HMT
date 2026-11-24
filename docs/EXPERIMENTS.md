# Experiments and resume-claim reconciliation

Evidence is source plus saved notebook outputs, not a new model run. Cell references are one-based physical positions in `recovered/notebooks/HMT_Transformer_v3_6.ipynb`.

## Dataset and split evidence

Cell 7's saved output records 8,543 rows, 79 CSV columns, four fault classes, four/five continuous inputs and 36 targets. A row is a metadata/response-spectrum record, **not a proven unique earthquake event or accelerogram component**. The CSV was not recovered, so counts of unique events/stations/components, quality filters, source composition and row duplication cannot be recomputed. The source explicitly names ESM, not a combined NGA-West corpus.

Cell 9 uses `GroupShuffleSplit` by `EventID`, first holding out 30% of groups, then splitting that group subset in half, both with seed 42. Saved row counts are 5,943 train / 1,335 validation / 1,265 test. Group proportions do not imply exact row proportions. Source intends event-disjoint splits and train-only scaling; missing data means actual ID overlap and provenance cannot be independently validated. There is no station-held-out split, external-region test, repeated-seed uncertainty analysis, or empirical GMPE baseline evaluation. Test-driven model/report iteration across prior history cannot be ruled out.

## Preserve variant lineage

Cell 15's saved output shows a six-input training run early-stopped at epoch 82 with best validation loss 0.1965, then **loads an existing five-input checkpoint** reporting best validation loss 0.2036. This is not a matched fresh retraining of both architectures. The six-input training call does not enable the optional monotonicity penalty.

Cell 17 historical test results:

| Metric | Six-input primary | Five-input comparison |
|---|---:|---:|
| Mean period-wise R² | 0.8500 | 0.8644 |
| Mean period-wise RMSE (log10) | 0.4374 | Per-period table in cell 55 |
| Signed mean period bias (log10) | -0.07086 | -0.04897 |
| 0.01 s PGA-proxy R² | 0.8299 | 0.8385 (cell 55) |
| Best period R² | 0.8962 at 6 s | See cell 55 |
| Worst period R² | 0.7818 at 0.15 s | See cell 55 |

Mean R² is the arithmetic mean of 36 separate `r2_score` values on inverse-scaled log10 PSA targets; it is not flattened spectral R², waveform fit, all-data R² or performance across 111 periods. PGA is a 0.01-second PSA proxy, not a separately loaded PGA target. Cell 55's rounded five-input table yields mean R² approximately 0.864364; this is arithmetic on historical tabulated metrics, not newly evaluated predictions.

The bias is signed prediction-minus-target averaged over records and periods; cancellation can hide period-specific bias. A 76% reduction would require an identified baseline, same split/units, precise period mask and explicit formula. None was recovered. Comparing five versus six input signed means does not establish that claim.

## Claim matrix

| Resume/context claim | Audit disposition |
|---|---|
| 18,500+ ESM/NGA-West records | Not supported by this recovered implementation; saved loader says 8,543 ESM rows. Broader experiment may exist elsewhere but was not recovered. |
| 111 spectral periods | Contradicted for this notebook: exactly 36 explicit output periods. |
| R² = 0.865 | Closest saved mean is five-input 0.8644; not exact rounding to three decimals (0.864). Six-input primary is 0.8500. Treat 0.865 as an unverified separate result, not a reproduced metric. |
| Hierarchical Multi-Token architecture | Scalar-token and period-decoder code exists; separate physical hierarchy is not implemented. Six-input depth–fault attention is real. |
| 76% spectral/long-period bias reduction | Unverified; no baseline experiment, before/after reduction calculation or corresponding results recovered. |
| Outperformed empirical GMPEs | Unverified; no executable GMPE baseline or matched comparison recovered. |
| SHAP and residual decomposition | Implemented; saved SHAP computation message and image payload exist. Does not prove global physical consistency. |
| Physics-informed monotonic attenuation | Optional sampled soft penalty exists, but training calls disable it by omitting scaler. No global guarantee or physics solver. |

## Other historical outputs

Cell 28 runs a separate one-factor sensitivity sweep with resampled training rows (up to 1,000, with replacement), 20 epochs, default d=64, two layers and four heads. It tests learning rate, batch size, width, layer count and dropout against validation loss. It is not a full-model cross-validated optimizer or evidence that the final CFG was selected by a controlled search.

Saved notebook payloads include 16 PNG figures and four HTML tables; independent image/table files listed in report prose were not recovered. HTML tables retain formatted historical statistics, per-period five-input metrics and diagnostic parameters even when text/plain only shows a Styler representation. The audit keeps their lineage; no metric was inferred from chart appearance.

## Defensible description

“Implemented scalar-token Transformer variants with a period-conditioned spectral decoder for 36 ground-motion PSA targets; the recovered notebook records an event-grouped ESM experiment with 8,543 rows and historical five-input mean test R² of 0.8644. Added residual decomposition and Kernel SHAP analysis; full reproduction remains blocked by missing data/checkpoints.”

The result sentence should be presented as historical until the original dataset, split and checkpoint are recovered and the reporting defects are resolved in a separately versioned experiment.
