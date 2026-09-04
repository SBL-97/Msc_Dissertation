# MSc Dissertation: Uncertainty in EHR-grounded Clinical QA

This repository contains the notebooks used for my MSc dissertation on uncertainty in EHR-grounded clinical question answering. The analysis treats evidence selection and answer generation as two separate stages and measures variability at each stage before comparing the uncertainty scores with answer correctness.

## Notebooks

The notebooks are intended to be run in order:

- `01_data_setup.ipynb` - prepares ArchEHR-QA and BioASQ in a common case format.
- `02_evidence_selection_uncertainty.ipynb` - repeats evidence selection and calculates evidence-selection uncertainty (EU).
- `03_answer_generation_uncertainty.ipynb` - generates repeated answers under fixed representative evidence and calculates answer-generation uncertainty (AU).
- `04_combined_uncertainty_analysis.ipynb` - combines the two uncertainty measures and runs the reliability analyses.

## Pipeline

For each case, evidence selection is repeated 10 times. EU is calculated as the mean pairwise Jaccard distance between the selected sentence sets.

A representative evidence set is then formed using a majority rule: a sentence is included if it was selected in at least 50% of the evidence-selection runs.

The representative evidence is kept fixed while 10 answers are generated. The answers are embedded with `sentence-transformers/all-MiniLM-L6-v2` and clustered using cosine distance. AU is calculated from the entropy of the resulting semantic clusters. Mean pairwise cosine distance is also reported as a secondary measure of answer variability.

A representative answer is selected from the largest semantic cluster and compared with the reference answer using a reference-informed LLM judge.

The final analysis includes:

- the relationship between EU and AU;
- relationships between the uncertainty measures and answer error;
- AUROC and risk-coverage/AURC;
- an equal-weight combined score, `(EU + AU) / 2`;
- four EU/AU variability profiles;
- bootstrap confidence intervals and paired comparisons.

## Main settings

- Evidence-selection runs: 10
- Answer-generation runs: 10
- Generation model: Gemma 3 12B
- Temperature: 0.7
- Top-p: 0.9
- Seeds: 1000-1009
- Primary clustering threshold: 0.10 cosine distance
- Correctness judge: Llama 3.3 70B Instruct
- Bootstrap resamples: 5,000
- Bootstrap seed: 42

## Data

ArchEHR-QA is used as the main EHR-grounded clinical QA dataset. BioASQ is used as a larger biomedical development dataset.

The source datasets are not included in this repository. They need to be obtained from their original sources and placed in the local `data/` directory before running `01_data_setup.ipynb`.

## Outputs

Only small summary outputs used to check or reproduce the reported analyses should be shared in the public repository. Raw model generations, processed dataset text and case-level review files are kept local.

The main summary files are:

```text
outputs/
├── evidence_selection/
│   └── full/
│       └── evidence_summary.csv
├── answer_generation/
│   └── full/
│       ├── answer_summary.csv
│       └── threshold_sensitivity.csv
└── combined_analysis/
    └── full/
        ├── correlations.csv
        ├── prediction_metrics.csv
        ├── joint_profile_summary.csv
        ├── risk_coverage.csv
        ├── selective_review.csv
        ├── bootstrap_metrics.csv
        └── bootstrap_comparisons.csv
```

Files containing full questions, EHR text, reference answers, generated answers or manual review content are not included here.

## Held-out evaluation

The main final evaluation uses 100 held-out ArchEHR-QA cases after the analysis choices were fixed during development. The held-out analysis uses the same EU/AU definitions, clustering threshold, profile definitions, correctness assessment and bootstrap procedure.
