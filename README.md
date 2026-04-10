# Thesis Cough Screening

Production-style scaffold for the thesis project:

`A mobile-integrated deep learning system for rapid screening of respiratory diseases using cough sound analysis`

## Project Overview

This repository sets up a local end-to-end machine learning pipeline for cough-audio screening of:

- `TB`
- `COVID`
- `HEALTHY_OR_NONTARGET`

The first version is intentionally focused on cough-compatible public datasets and a lightweight baseline model that can later be exported to TensorFlow Lite for mobile integration.

## Thesis Objective

Build a reproducible cough-audio pipeline that:

1. acquires public respiratory audio datasets,
2. audits and cleans their metadata,
3. unifies labels into a controlled schema,
4. preprocesses cough audio,
5. creates train/validation/test splits,
6. trains a baseline TensorFlow CNN on mel spectrograms,
7. evaluates the model with standard classification metrics,
8. prepares for later mobile deployment.

## Revised 3-Class Scope

The current scope uses:

- `TB`
- `COVID`
- `HEALTHY_OR_NONTARGET`

`HEALTHY_OR_NONTARGET` is a combined control class for the first thesis iteration. It may contain:

- explicitly healthy participants,
- COVID-negative participants,
- TB-negative participants,
- other non-target controls that are compatible with the source dataset documentation.

This improves practicality, but it is also a scientific limitation because mixed controls are not equivalent to a strictly healthy population. That limitation is documented in [src/config/label_schema.py](/Users/tm/projects/thesis-cough-screening/src/config/label_schema.py).

## Folder Structure

```text
thesis-cough-screening/
  README.md
  requirements.txt
  .gitignore
  .env.example
  data/
  metadata/
  notebooks/
  src/
  scripts/
  models/
  reports/
  tests/
```

## Setup

Use Python `3.10` or `3.11`.

```bash
cd ~/projects/thesis-cough-screening
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
python -m ipykernel install --user --name thesis-cough-screening
```

Or run:

```bash
bash scripts/setup_env.sh
```

## Dataset Acquisition

### 1. Coswara

- Primary record: https://zenodo.org/records/7188627
- Website: https://coswara.iisc.ac.in/about
- Paper: https://www.nature.com/articles/s41597-023-02266-0

The download script uses the public Zenodo record and extracts the archive to `data/raw/coswara/`.

### 2. COUGHVID

- Dataset page: https://www.epfl.ch/labs/esl/index-html/datasets/coughviddataset/
- Public archive: https://zenodo.org/records/4498364
- Repository: https://github.com/esl-epfl/COUGHVID

The download script uses the public Zenodo archive and extracts it to `data/raw/coughvid/`.

### 3. CODA / TB dataset

- UCSF page: https://tbdata.ucsf.edu/s/rdc-dataset/a0U5w00000fTCKiEAO/ds000731
- Paper: https://www.nature.com/articles/s41597-024-03972-z
- Synapse: https://www.synapse.org/Synapse:syn31472953

This dataset is treated as manual / controlled access. The pipeline will:

- create `data/raw/tb/`,
- create `README_MANUAL_DOWNLOAD.md` there,
- skip fake downloads,
- fail gracefully downstream until the files are available.

## Label Strategy

The target schema is defined in [src/config/label_schema.py](/Users/tm/projects/thesis-cough-screening/src/config/label_schema.py).

High-level rules:

- map only clearly documented source labels,
- exclude ambiguous rows from training,
- keep `label_raw`,
- derive `label_binary` and `label_multiclass`,
- document mixed-control limitations in metadata notes.

## Pipeline Execution Order

1. Create and activate a virtual environment.
2. Install dependencies.
3. Run dataset acquisition.
4. Audit dataset metadata.
5. Build cleaned per-dataset metadata.
6. Create unified metadata and train/validation/test splits.
7. Precompute sample features if desired.
8. Train the baseline model.
9. Evaluate on the test split.
10. Export a TensorFlow Lite model scaffold.

## Assumptions And Limitations

- TB data is not downloaded automatically because access may require registration or approval.
- Dataset-specific metadata schemas differ and are handled with heuristics plus explicit label rules.
- Subject-independent splitting is used where subject identifiers exist; otherwise the pipeline falls back to sample-level stratification and logs the limitation.
- The baseline training code is thesis-friendly and easy to inspect, but not yet optimized for large-scale production training.
- The initial control class mixes healthy and broader non-target controls when necessary.

## Next Steps Toward Mobile Deployment

- stabilize label curation after manual review,
- improve cough event segmentation,
- benchmark feature extraction latency,
- export a well-performing model to TFLite,
- add quantization and mobile benchmarking,
- integrate the inference package into a mobile client once ML quality is acceptable.

## Exact Next Commands

Run these in order:

```bash
cd ~/projects/thesis-cough-screening
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
bash scripts/run_downloads.sh
bash scripts/run_audit.sh
bash scripts/run_preprocessing.sh
bash scripts/run_training.sh
bash scripts/run_evaluation.sh
python -m src.export.export_tflite --checkpoint models/checkpoints/baseline_cnn.keras --output models/exported/baseline_cnn.tflite
```

If the TB dataset is still missing, complete the manual download steps in [README_MANUAL_DOWNLOAD.md](/Users/tm/projects/thesis-cough-screening/data/raw/tb/README_MANUAL_DOWNLOAD.md) before expecting a true 3-class training run.
