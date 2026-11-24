# Recovery and provenance

## Scope

Recovered source: `recovered/notebooks/HMT_Transformer_v3_6.ipynb` (60 cells). Its original local copy and the pre-recovery repository notebook have identical raw-file SHA256 and identical cell sources. The repository baseline contains only the notebook and a one-line README. Local Git history contains an initial commit and a notebook upload, both dated September 6, 2026; it does not reconstruct the full development process.

The recovery inspected every cell source, saved text/error output, and saved HTML table, and inventoried embedded image MIME payloads by hash. Source syntax parsed successfully for every code cell; this is not execution or training verification. Notebook outputs were not regenerated.

## Missing assets

The source references `ESM_filtered.csv`, `hmt_v3_5inp.pt`, `hmt_v3_6inp.pt`, and a `Report_Figures` output directory. None of those standalone assets was found in the bounded indexed search. Report figure and CSV filenames were searched explicitly; indexed CSV headers and ZIP member names were also checked for compatible assets. Some archives were unreadable, and other drives, cloud-only files, excluded application/dependency directories and external collaborators' machines were outside scope. “Not recovered” is not a claim that an asset never existed.

The original saved notebook contains figures and formatted tables even though standalone exports are missing. Public source copies clear outputs to avoid publishing incidental local runtime paths. The private recovery archive preserves the untouched notebook and detailed output inventory. Saved-output table CSV extractions, if included under `recovered/reports/`, are explicitly historical derivatives from notebook HTML, not newly computed experiments.

## History versus experiment evidence

Relevant prior project-review and resume-editing history was recovered locally. Earlier review text describes an 8,543-row ESM notebook; later resume wording and revision notes repeat 18,500+, 111 periods, GMPE superiority and 76% bias reduction. Such repetition establishes the history of a claim, not its measurement. No original training/development conversation establishing the broader experiment was recovered. Session indexes had broader metadata coverage than full-text search coverage, so absence from search is inconclusive.

Personal notes, resume drafts, private session excerpts and local absolute-path manifests remain local-only. They are not repository source and must not be copied into a public repository. The public README/docs summarize factual limitations without embedding raw chat, private paths or session identifiers.

## Reproducibility gate

1. Recover the actual CSV and its source/filtering/data dictionary; establish records versus events/components and redistribution rights.
2. Recover matching checkpoint weights, preprocessing/scaler parameters and split identities. Do not assume a similarly named model is compatible.
3. Correct preprocessing helpers, EMA/best-checkpoint logic and report lineage in a **new variant**, keeping this historical notebook intact.
4. Ensure soft-physics terms are intentionally activated and valid for the selected model, if used; do not claim guaranteed physics.
5. Reproduce metrics on an untouched event-held-out split; add documented baseline calculations before asserting superiority or percentage reductions.
6. Publish only reviewed code and safe reports; keep credentials, raw histories and personal notes excluded. No license is inferred from a public repository upload.

No notebook execution, checkpoint deserialization, package installation or network retrieval occurred in this audit.
