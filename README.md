# HMT Ground Motion Transformer

A recovered PyTorch research notebook for predicting a response spectrum from earthquake metadata. The implementation uses scalar feature tokens, a Transformer encoder, and a period-conditioned cross-attention decoder. It contains separate five-input and six-input models and report-generation code.

**Status: historical research artifact, not a reproduced benchmark or engineering design tool.** The dataset and trained checkpoints were not recovered. The notebook is preserved rather than silently repaired; known preprocessing, checkpoint-selection and reporting issues are documented below.

## What the recovered implementation actually contains

- `recovered/notebooks/HMT_Transformer_v3_6.ipynb`: the recovered 60-cell notebook (public copy has saved outputs cleared).
- Two architectures: four continuous features plus fault class; a second adds log-depth and a depth–fault context token.
- 36 PSA targets, from 0.01 to 10 seconds, in log10 space.
- Event-grouped train/validation/test splitting and train-fitted scalers.
- AdamW, warmup/cosine learning rate, EMA, magnitude-weighted error, bias and spectral-smoothness penalties.
- Historical report code for residual decomposition, sensitivity studies, gradient importance and Kernel SHAP.

## Historical results and claim boundaries

Saved output in notebook cell 7 reports **8,543 ESM rows and 79 input CSV columns**, not 18,500+ combined ESM/NGA-West records. Cell 17 reports the following event-held-out test metrics, computed in log10 PSA space as arithmetic means over 36 per-period metrics:

| Variant | Mean R² | Mean RMSE | Signed mean spectral bias |
|---|---:|---:|---:|
| Five-input comparison | 0.8644 | See historical per-period table | -0.04897 |
| Six-input primary | 0.8500 | 0.4374 | -0.07086 |

These are **saved historical outputs**, not results reproduced during recovery. The five-input run loads an already-existing checkpoint in the saved training output; that checkpoint is unavailable. Resume claims of 111 output periods, a 76% bias reduction, and superiority over empirical GMPE baselines are not established by the recovered experiment. SHAP code and saved figure output exist, but explanation plots do not prove physical consistency.

## Reading guide

- [Implementation and known defects](docs/IMPLEMENTATION.md)
- [Experiments, splits and claim reconciliation](docs/EXPERIMENTS.md)
- [Recovery provenance and missing assets](docs/RECOVERY.md)

## Reproduction prerequisites

This is **not currently a complete reproducible training package**. Obtain the original `ESM_filtered.csv` under its applicable terms, establish its provenance/units, and recover the matching `hmt_v3_5inp.pt` and `hmt_v3_6inp.pt` if comparing against historical results. Do not substitute unrelated weights.

The notebook imports PyTorch, NumPy, pandas, scikit-learn, matplotlib, SciPy, SHAP and IPython/Jupyter; it also contains a pip-install cell and optional Colab Drive mounting. No dependency lockfile was recovered. Historical saved output reports PyTorch 2.5.1+cu121 and NumPy 2.2.6; this is not a complete environment specification.

Before running, inspect cell 5 paths and the defects in the implementation document. Cell 15 can initiate training, cell 28 trains a separate sensitivity sweep, and existing checkpoint loading uses `torch.load`. Use only trusted checkpoints and an isolated environment. No cells were executed as part of recovery.

No dataset redistribution permission, project license, publication, or independent benchmark validation is implied. Do not use this notebook for structural design or safety-critical decisions.


## Archive safety

Raw assets and original notebooks remain local under `private/`. Credentials are not copied. See [archive safety](docs/ARCHIVE-SAFETY.md) for verification limits and the existing-history warning.
