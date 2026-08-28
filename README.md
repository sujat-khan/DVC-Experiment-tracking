# Machine Learning Experiment Tracking with DVC & DVCLive

This repository is a hands-on guide and reference project demonstrating how to track, manage, compare, and reproduce machine learning experiments using **DVC (Data Version Control)** and **DVCLive**.

---

## 📌 Project Overview

When building ML models, you frequently iterate over hyperparameters (e.g., `n_estimators`, `max_depth`), feature sets, and datasets. Manually recording metrics or making Git commits for every minor parameter tweak quickly becomes messy.

This project solves that by using:
1. **DVC Data Versioning**: Tracking dataset files (`student_performance.csv`) without bloating the Git repository.
2. **DVCLive**: Programmatically logging parameters and metrics inside the training script and capturing them as isolated DVC experiments.
3. **DVC Experiment Management**: Viewing, comparing, reverting, and branching experiments directly from the CLI.

---

## 📁 Repository Structure

```text
DVC-Experiment-tracking/
├── .dvc/                           # DVC internal configuration and cache tracking
├── .dvcignore                      # Patterns DVC should ignore
├── .gitignore                      # Git ignore file for virtualenv, caches, IDEs
├── dvc.yaml                        # DVC pipeline definition (auto-configured by DVCLive)
├── dvclive/                        # Logged parameters, metrics, and plot data
│   ├── metrics.json                # Latest evaluation metrics
│   ├── params.yaml                 # Latest hyperparameter values
│   └── plots/                      # Generated performance plots over steps/epochs
├── data/
│   ├── .gitignore                  # Git ignores large data files tracked by DVC
│   ├── data_generate.py            # Script to generate sample student performance data
│   ├── student_performance.csv     # Raw dataset (tracked by DVC)
│   └── student_performance.csv.dvc # DVC pointer/metadata file (tracked by Git)
├── src/
│   └── model_training.py           # Model training and DVCLive logging script
└── README.md                       # This reference and revision guide
```

---

## 🚀 How It Works (Step-by-Step)

### 1. Data Generation & DVC Tracking
The synthetic dataset is generated using `data/data_generate.py`. Because data files can grow large, the CSV is placed under DVC control:
```bash
dvc add data/student_performance.csv
git add data/student_performance.csv.dvc data/.gitignore
git commit -m "Track dataset with DVC"
```

### 2. Model Training with DVCLive
In `src/model_training.py`, we train a `RandomForestClassifier` and wrap metric and hyperparameter logging inside a `Live` context manager:

```python
from dvclive import Live

# ... data loading and model training ...

with Live(save_dvc_exp=True) as live:
    # 1. Log evaluation metrics
    live.log_metric('accuracy', accuracy_score(y_test, y_pred))
    live.log_metric('precision', precision_score(y_test, y_pred))
    live.log_metric('recall', recall_score(y_test, y_pred))
    live.log_metric('f1_score', f1_score(y_test, y_pred))

    # 2. Log hyperparameters
    live.log_param('n_estimator', n_estimators)
    live.log_param('max_depth', max_depth)
```

> **Key Setting**: `Live(save_dvc_exp=True)` automatically registers the execution as an isolated DVC experiment (assigning a unique hash and human-readable experiment name like `quiet-haet` or `folio-prof`).

---

## 🛠️ Essential DVC Command Reference & Cheat Sheet

Keep this section handy for quick revision.

### 📊 Viewing & Listing Experiments

| Goal | Command | Description |
| :--- | :--- | :--- |
| **View experiment table** | `dvc exp show --no-pager` | Displays experiments, metrics, and parameters in a clean table without terminal pager freezes. |
| **View across all commits** | `dvc exp show -A --no-pager` | Shows all experiments ever run across any commit/branch in Git history. |
| **List experiment names** | `dvc exp list` | Lists all experiment hashes and generated names for the current branch. |
| **List all experiments** | `dvc exp list -A` | Lists experiment hashes and names across all Git commits. |
| **Markdown / CSV output** | `dvc exp show --no-pager --markdown` | Formats the experiment summary table as Markdown. (Use `--csv` or `--json` for other formats). |

---

### 🔍 Comparing Experiments

| Goal | Command | Description |
| :--- | :--- | :--- |
| **Compare workspace vs last exp** | `dvc exp diff` | Shows differences in metrics and params between your working tree and the baseline. |
| **Compare two specific exps** | `dvc exp diff <exp1> <exp2>` | Compares metric/param differences between two specific experiment names or hashes. |

---

### ⏪ Applying, Branching & Deleting Experiments

| Goal | Command | Description |
| :--- | :--- | :--- |
| **Apply / Restore an experiment** | `dvc exp apply <exp_name>` | Restores the exact code, parameters, and workspace state from a previous experiment into your current workspace. |
| **Create a Git branch from an exp** | `dvc exp branch <exp_name> <branch_name>` | Promotes a successful experiment to a real Git branch for further development or production pull requests. |
| **Remove an experiment** | `dvc exp remove <exp_name>` | Deletes an unwanted experiment record to keep experiment history clean. |
| **Remove all untracked exps** | `dvc exp remove -A` | Removes all temporary experiments across all commits. |

---

### 💾 Data Tracking Commands

| Goal | Command | Description |
| :--- | :--- | :--- |
| **Track a data file or directory** | `dvc add <path/to/data>` | Creates a `.dvc` pointer file and adds the original file to `.gitignore`. |
| **Check data status** | `dvc status` | Checks if any DVC-tracked data files have changed. |
| **Pull data from remote** | `dvc pull` | Fetches data files matching the `.dvc` tracking files from remote storage. |
| **Push data to remote** | `dvc push` | Uploads cached data files to configured remote storage (S3, GCS, Azure, SSH, local folder). |

---

## 💡 Practical Tips & Troubleshooting

1. **Avoid Windows Terminal Freezing on `dvc exp show`**:
   - On Windows, DVC may route table output through `pydoc`/`less`, prompting `"pydoc.out may be a binary file. See it anyway?"`.
   - **Fix**: Always append `--no-pager` (e.g. `dvc exp show --no-pager`).
2. **Iterative Experiment Workflow**:
   - Modify hyperparameter in [src/model_training.py](file:///c:/Users/sujat/projects/ML-Main/DVC-Experiment-tracking/src/model_training.py).
   - Run `python src/model_training.py`.
   - Run `dvc exp show --no-pager` to inspect how accuracy and F1-score changed.
   - If satisfied with an experiment, use `dvc exp apply <exp_name>` to bring back those exact values anytime.
