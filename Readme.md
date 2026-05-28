# STP Customer Segmentation (MLOps)

An end-to-end machine learning pipeline for e-commerce customer segmentation. This project operationalizes the clustering of retail customers based on purchasing behavior (RFM metrics) to drive personalized marketing strategies.

It implements a strict MLOps process model, ensuring reproducibility, scalability, and continuous monitoring.

***

## System Architecture

| Component | Technology | Details |
|-----------|------------|---------|
| **Data Versioning** | DVC | Remote: OVGU Nextcloud WebDAV |
| **Experiment Tracking & Model Registry** | MLflow | Hosted on DagsHub |
| **Containerization** | Docker | Single modular image |
| **Application / Utilization** | Streamlit | SPA with Marketing & MLOps views |

***

## Repository Structure

```text
├── .dvc/                   # DVC configuration
├── .github/workflows/      # CI/CD Pipelines
├── data/                   # Tracked by DVC (Do not commit to Git)
│   ├── raw/                # Online Retail dataset (.csv)
│   └── processed/          # Engineered RFM features
├── docs/                   # Scientific documentation and architecture
├── models/                 # Local model artifacts (Logged to MLflow)
├── notebooks/              # EDA and prototyping ONLY (Do not deploy)
├── src/                    # Production Code Base
│   ├── data_prep.py        # Data cleaning and feature engineering
│   ├── train.py            # Model selection and training
│   ├── evaluate.py         # Performance and business evaluation
│   ├── app.py              # Single-page UI (Utilization phase)
│   └── main.py             # Pipeline orchestrator
├── tests/                  # Pytest validation scripts
└── Dockerfile              # Deployment environment
```

***

## Local Setup Instructions

> **Prerequisites:** You need an OVGU **App Password** for Nextcloud and a **Personal Access Token (PAT)** from DagsHub before proceeding.

### 1. Clone & Isolate Environment

> Do not install packages globally. Create and activate a virtual environment first.

```bash
git clone https://github.com/[YOUR-USERNAME]/stp-customer-segmentation.git
cd stp-customer-segmentation

# Create virtual environment
python -m venv .venv

# Activate on Windows:
.venv\Scripts\activate

# Activate on Mac/Linux:
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Configure Data Access (DVC)

Run these commands to link your local machine to the university cloud so you can pull the dataset.

```bash
dvc remote modify ovgu-remote user [YOUR-OVGU-USERNAME]
dvc remote modify ovgu-remote --local password [YOUR-NEXTCLOUD-APP-PASSWORD]
dvc pull
```

### 3. Configure Model Tracking (MLflow)

Create a `.env` file in the project root. **DO NOT commit this file to GitHub.**

```env
MLFLOW_TRACKING_USERNAME=[YOUR-GITHUB-USERNAME]
MLFLOW_TRACKING_PASSWORD=[YOUR-DAGSHUB-TOKEN]
```

### 4. Run the Application (Docker)

To verify the deployment environment locally:

```bash
docker build -t stp-segmentation .
docker run -p 8501:8501 --env-file .env stp-segmentation
```

Then open [http://localhost:8501](http://localhost:8501) in your browser.

***

## Development Roles & Workflow

| Role | Owner | Primary Files | Key Responsibility |
|------|-------|---------------|--------------------|
| **Data Lead** | Ayush | `src/data_prep.py` | Track data changes with `dvc add data/processed/file.csv` and `dvc push` |
| **ML Lead** | Kirtan | `src/train.py`, `src/evaluate.py` | All models **must** be logged via `mlflow.start_run()` |
| **MLOps Lead** | Jash | `src/app.py`, `Dockerfile`, `.github/workflows/` | Infrastructure, Docker deployment, CI/CD, and Streamlit application |

***

## Rules of Engagement (Branching Strategy)

> ⚠️ The `main` branch is **locked**. You cannot push directly to `main`.

- **Feature Branching:** Create small, task-specific branches (e.g., `git checkout -b feature/kmeans-baseline`). Do not create massive milestone branches.
- **Pull Requests:** All code must be merged via a Pull Request. It requires **at least 1 approving review** from another team member to merge.
- **Notebooks are temporary.** All exploratory code in `notebooks/` must be translated into modular functions in `src/` to be considered "Done."

***

*This README is the single source of truth for project setup and team conventions. Keep it updated as the project evolves.*