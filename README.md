# MSc Dissertation: Uncertainty in EHR-grounded Clinical QA

This repository contains the analysis pipeline used for my MSc dissertation on uncertainty in EHR-grounded clinical question answering. The study treats evidence selection and answer generation as two separate stages and measures variability at each stage before examining how the uncertainty measures relate to answer correctness.

## Notebooks

The notebooks are intended to be read in order:

- `01_data_setup.ipynb` — prepares ArchEHR-QA and BioASQ in a common case format.
- `02_evidence_selection_uncertainty.ipynb` — repeats evidence selection and calculates evidence-selection uncertainty (EU).
- `03_answer_generation_uncertainty.ipynb` — generates repeated answers under fixed representative evidence and calculates answer-generation uncertainty (AU).
- `04_combined_uncertainty_analysis.ipynb` — compares EU and AU and runs the reliability analyses.

The notebooks are left in `SAMPLE_MODE = True` by default so that the code can be checked with a small local run. The files in `github_results/development/` and `github_results/heldout/` contain the full development and held-out results used for the dissertation.

## Pipeline

For each case, evidence selection is repeated 10 times. EU is calculated as the mean pairwise Jaccard distance between the selected sentence sets.

A representative evidence set is then formed using a majority rule: a sentence is included if it was selected in at least 50% of the evidence-selection runs.

The representative evidence is kept fixed while 10 answers are generated. The answers are embedded with `sentence-transformers/all-MiniLM-L6-v2` and grouped using agglomerative clustering with cosine distance. AU is calculated from the normalised entropy of the resulting semantic clusters. Mean pairwise cosine distance is also reported as a secondary measure of answer variability.

A representative answer is selected from the largest semantic cluster and compared with the reference answer using a reference-informed LLM judge.

The final analysis includes:

- the relationship between EU and AU;
- relationships between uncertainty and answer error;
- AUROC and risk-coverage/AURC;
- the equal-weight combined score `(EU + AU) / 2`;
- four joint variability profiles;
- bootstrap confidence intervals and paired signal comparisons.

## Main settings

- Evidence-selection runs: 10
- Answer-generation runs: 10
- Generation model: Gemma 3 12B
- Temperature: 0.7
- Top-p: 0.9
- Generation seeds: 1000–1009
- Primary semantic clustering threshold: 0.10 cosine distance
- Correctness judge for the full analysis: Llama 3.3 70B Instruct
- Bootstrap resamples: 5,000
- Bootstrap seed: 42

## Data

ArchEHR-QA is the main EHR-grounded clinical QA dataset. BioASQ is used as a larger biomedical development dataset.

The source datasets are not included in this repository.

## Included results

Only the outputs needed to follow the dissertation analyses are included. Raw repeated generations, full case text, reference answers, judge explanations and processed source data are not committed.

### Development results

`github_results/development/`

- `evidence_summary.csv`
- `answer_summary.csv`
- `threshold_sensitivity.csv`
- `correlations.csv`
- `prediction_metrics.csv`
- `bootstrap_metrics.csv`
- `bootstrap_comparisons.csv`
- `joint_profile_summary.csv`
- `risk_coverage.csv`
- `selective_review.csv`
- `judge_run_metadata.json`

### Held-out results

`github_results/heldout/`

- `evidence_summary.csv`
- `answer_summary.csv`
- `threshold_sensitivity.csv`
- `correlations.csv`
- `prediction_metrics.csv`
- `bootstrap_metrics.csv`
- `bootstrap_comparisons.csv`
- `joint_profile_summary.csv`
- `risk_coverage.csv`
- `selective_review.csv`

These files contain the corresponding results for the 100 held-out ArchEHR-QA cases used for the final evaluation.

## Outputs not included

The following are deliberately excluded from the public repository:

- raw repeated-generation JSONL files;
- case-level answer clustering files;
- judge outputs and explanations;
- `case_analysis.csv`;
- `manual_judge_review.csv`;
- processed dataset files;
- PubMed cache files.
