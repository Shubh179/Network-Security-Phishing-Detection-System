<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white" />
  <img src="https://img.shields.io/badge/MLflow-0194E2?style=for-the-badge&logo=mlflow&logoColor=white" />
  <img src="https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white" />
  <img src="https://img.shields.io/badge/Scikit--Learn-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white" />
  <img src="https://img.shields.io/badge/DagsHub-FF6F00?style=for-the-badge&logoColor=white" />
</p>

# 🛡️ Network Security — Phishing Detection System

An end-to-end **MLOps-powered** machine learning system for detecting phishing websites. The project features a fully automated training pipeline, experiment tracking with **MLflow + DagsHub**, hyperparameter tuning via **GridSearchCV**, and a production-ready **FastAPI** REST API — all backed by **MongoDB Atlas** for data storage and **AWS S3** for artifact syncing.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Dataset](#-dataset)
- [Models Trained & Performance](#-models-trained--performance)
- [MLflow Experiment Tracking](#-mlflow-experiment-tracking)
- [Architecture](#️-architecture)
- [Project Structure](#-project-structure)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [API Endpoints](#-api-endpoints)
- [Docker Deployment](#-docker-deployment)
- [AWS Deployment](#-aws-deployment)
- [Troubleshooting](#-troubleshooting)
- [Tech Stack](#️-tech-stack)
- [Acknowledgments](#-acknowledgments)

---

## 🔍 Overview

Phishing websites are one of the most common cyber threats. This project builds a complete ML pipeline that:

1. **Ingests** data from MongoDB Atlas
2. **Validates** data quality and detects distribution drift (Kolmogorov–Smirnov test)
3. **Transforms** features using KNN Imputation for missing values
4. **Trains** 5 different classifiers with hyperparameter tuning
5. **Tracks** all experiments with MLflow on DagsHub
6. **Deploys** the best model via FastAPI for real-time predictions
7. **Syncs** artifacts and models to AWS S3

---

## 📊 Dataset

| Property | Value |
|---|---|
| **Name** | Phishing Websites Dataset |
| **Samples** | 11,055 |
| **Features** | 30 (all numerical/binary) |
| **Target** | `Result` (1 = Legitimate, -1 = Phishing) |
| **Class Distribution** | 6,157 Legitimate (55.7%) · 4,898 Phishing (44.3%) |
| **Train/Test Split** | 80% / 20% |
| **Storage** | MongoDB Atlas (`KRISHAI.NetworkData`) |

### Feature Categories

| Category | Features |
|---|---|
| **URL-Based** | `having_IP_Address`, `URL_Length`, `Shortining_Service`, `having_At_Symbol`, `double_slash_redirecting`, `Prefix_Suffix`, `having_Sub_Domain` |
| **Domain-Based** | `SSLfinal_State`, `Domain_registeration_length`, `Favicon`, `port`, `HTTPS_token`, `age_of_domain`, `DNSRecord` |
| **Content-Based** | `Request_URL`, `URL_of_Anchor`, `Links_in_tags`, `SFH`, `Submitting_to_email`, `Abnormal_URL`, `Redirect`, `on_mouseover`, `RightClick`, `popUpWidnow`, `Iframe` |
| **Reputation-Based** | `web_traffic`, `Page_Rank`, `Google_Index`, `Links_pointing_to_page`, `Statistical_report` |

---

## 🏆 Models Trained & Performance

The system trains **5 classifiers** using `GridSearchCV` (3-fold cross-validation) for hyperparameter tuning and selects the best model automatically based on R² score.

### Models & Hyperparameter Search Space

| # | Model | Hyperparameters Tuned | Search Space |
|---|---|---|---|
| 1 | **Random Forest** | `n_estimators` | `[8, 16, 32, 128, 256]` |
| 2 | **Decision Tree** | `criterion` | `['gini', 'entropy', 'log_loss']` |
| 3 | **Gradient Boosting** | `learning_rate`, `subsample`, `n_estimators` | `[0.1, 0.01, 0.05, 0.001]` × `[0.6, 0.7, 0.75, 0.85, 0.9]` × `[8, 16, 32, 64, 128, 256]` |
| 4 | **Logistic Regression** | — | Default parameters |
| 5 | **AdaBoost** | `learning_rate`, `n_estimators` | `[0.1, 0.01, 0.001]` × `[8, 16, 32, 64, 128, 256]` |

### Metrics Tracked Per Model

Every training run logs the following metrics to MLflow for both **train** and **test** splits:

| Metric | Description |
|---|---|
| **F1 Score** | Harmonic mean of precision and recall |
| **Precision** | True positives / (True positives + False positives) |
| **Recall** | True positives / (True positives + False negatives) |

### Model Selection Criteria

- All 5 models are evaluated on the **test set** using R² score
- The model with the **highest R² score** is selected as the best model
- A **minimum expected accuracy of 0.6** is required (`MODEL_TRAINER_EXPECTED_SCORE`)
- An **overfitting threshold of 0.05** ensures the train-test gap is acceptable (`MODEL_TRAINER_OVER_FIITING_UNDER_FITTING_THRESHOLD`)

### Data Preprocessing Pipeline

| Step | Method | Parameters |
|---|---|---|
| Missing Value Imputation | `KNNImputer` | `n_neighbors=3`, `weights='uniform'` |
| Target Encoding | Label mapping | `-1 → 0` (Phishing), `1 → 1` (Legitimate) |

---

## 📈 MLflow Experiment Tracking

All experiments are tracked using **MLflow** integrated with **DagsHub** for collaborative experiment management.

### Tracking Configuration

| Setting | Value |
|---|---|
| **Tracking Server** | DagsHub MLflow |
| **Tracking URI** | `https://dagshub.com/Raiyyanpatel/Network-Security-Phishing-Detection-System.mlflow` |
| **Experiment Dashboard** | [View on DagsHub →](https://dagshub.com/Raiyyanpatel/Network-Security-Phishing-Detection-System/experiments) |
| **Model Registry** | DagsHub MLflow Model Registry |

### What Gets Logged

```
Each training run logs to MLflow:
├── Metrics
│   ├── f1_score        (train & test)
│   ├── precision       (train & test)
│   └── recall_score    (train & test)
├── Model Artifact
│   └── sklearn model (serialized via mlflow.sklearn.log_model)
└── Model Registration
    └── Best model registered in MLflow Model Registry
```

### How It Works

1. **Training Phase**: Each of the 5 models is trained with `GridSearchCV`
2. **Evaluation Phase**: The best model is selected based on test R² score
3. **MLflow Logging**: Train and test metrics are logged as **separate runs**
4. **Model Registration**: The best model is registered in the MLflow Model Registry on DagsHub
5. **Artifact Storage**: Model `.pkl` files are saved locally and synced to S3

### Viewing Experiments

Visit your DagsHub experiments page to:
- Compare F1, Precision, and Recall across all runs
- View hyperparameters of each trained model
- Download registered model artifacts
- Analyze train vs. test metric gaps for overfitting detection

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        TRAINING PIPELINE                            │
│                                                                     │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐        │
│  │   MongoDB     │──▶│    Data      │──▶│      Data        │        │
│  │   Atlas       │   │  Ingestion   │   │   Validation     │        │
│  │              │   │              │   │  (KS-test drift) │        │
│  └──────────────┘   └──────────────┘   └──────────────────┘        │
│                                                │                    │
│                                                ▼                    │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐        │
│  │   MLflow /   │◀──│    Model     │◀──│      Data        │        │
│  │   DagsHub    │   │   Trainer    │   │  Transformation  │        │
│  │  (Tracking)  │   │ (5 Models +  │   │  (KNN Imputer)   │        │
│  └──────────────┘   │  GridSearch) │   └──────────────────┘        │
│                     └──────┬───────┘                                │
│                            │                                        │
│                            ▼                                        │
│                     ┌──────────────┐   ┌──────────────────┐        │
│                     │  Best Model  │──▶│     AWS S3       │        │
│                     │  Selection   │   │  (Artifact Sync) │        │
│                     └──────────────┘   └──────────────────┘        │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                       PREDICTION PIPELINE                           │
│                                                                     │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐        │
│  │   CSV Upload │──▶│ Preprocessor │──▶│   Best Model     │──▶ HTML│
│  │  (FastAPI)   │   │   (.pkl)     │   │   Prediction     │  Table │
│  └──────────────┘   └──────────────┘   └──────────────────┘        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
networksecurity/
├── app.py                              # FastAPI application (train + predict endpoints)
├── main.py                             # Training pipeline entry point (CLI)
├── push_data.py                        # MongoDB data ingestion utility
├── test_mongodb.py                     # MongoDB connection testing
├── Dockerfile                          # Container configuration
├── requirements.txt                    # Python dependencies
├── setup.py                            # Package setup configuration
├── .env                                # Environment variables (not committed)
│
├── networksecurity/                    # Main package
│   ├── components/                     # Pipeline components
│   │   ├── data_ingestion.py          #   → MongoDB export + train/test split
│   │   ├── data_validation.py         #   → Schema check + KS-test drift detection
│   │   ├── data_transformation.py     #   → KNN Imputation + target encoding
│   │   └── model_trainer.py           #   → 5 models + GridSearchCV + MLflow logging
│   │
│   ├── pipeline/                       # Orchestration
│   │   ├── training_pipeline.py       #   → End-to-end training orchestrator
│   │   └── batch_prediction.py        #   → Batch prediction pipeline
│   │
│   ├── entity/                         # Data classes
│   │   ├── config_entity.py           #   → Pipeline configuration classes
│   │   └── artifact_entity.py         #   → Artifact dataclasses (metrics, paths)
│   │
│   ├── constant/training_pipeline/     # Constants & hyperparameters
│   ├── exception/                      # Custom NetworkSecurityException
│   ├── logging/                        # Timestamped logging
│   ├── utils/                          # Utility functions
│   │   ├── main_utils/utils.py        #   → save/load objects, GridSearchCV evaluator
│   │   └── ml_utils/
│   │       ├── model/estimator.py     #   → NetworkModel (preprocessor + model wrapper)
│   │       └── metric/classification_metric.py  # → F1, Precision, Recall
│   └── cloud/
│       └── s3_syncer.py               #   → AWS S3 sync (artifacts + models)
│
├── Artifacts/                          # Timestamped training artifacts
├── final_model/                        # Production model
│   ├── model.pkl                      #   → Best trained classifier
│   └── preprocessor.pkl               #   → KNN Imputer pipeline
├── Network_Data/phisingData.csv        # Raw dataset (11,055 samples)
├── data_schema/schema.yaml             # Validation schema
├── prediction_output/output.csv        # Prediction results
├── templates/table.html                # Jinja2 prediction results template
└── logs/                               # Application logs
```

---

## 🚀 Installation

### Prerequisites

- Python 3.10+
- MongoDB Atlas account
- DagsHub account (for MLflow tracking)
- AWS CLI (optional, for S3 sync)

### Setup

```bash
# 1. Clone the repository
git clone https://github.com/Raiyyanpatel/Network-Security-Phishing-Detection-System.git
cd Network-Security-Phishing-Detection-System/networksecurity

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate          # Linux/Mac
venv\Scripts\activate             # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Install package in development mode
pip install -e .

# 5. Configure environment variables (see below)
cp .env.example .env
# Edit .env with your credentials
```

---

## ⚙️ Configuration

### Environment Variables (`.env`)

Create a `.env` file in the project root:

```env
# ── MongoDB Atlas ──────────────────────────────────────────
# Used by: push_data.py, data_ingestion.py
MONGO_DB_URL=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/?appName=<app_name>

# Used by: app.py (FastAPI server) — same URL, different key name
MONGODB_URL_KEY=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/?appName=<app_name>

# ── MLflow / DagsHub Tracking ──────────────────────────────
# Used by: model_trainer.py
MLFLOW_TRACKING_URI=https://dagshub.com/<username>/<repo>.mlflow
MLFLOW_TRACKING_USERNAME=<dagshub_username>
MLFLOW_TRACKING_PASSWORD=<dagshub_token>  # Get from dagshub.com/user/settings/tokens

# ── AWS S3 (Optional — for artifact sync) ──────────────────
AWS_ACCESS_KEY_ID=<your_aws_access_key>
AWS_SECRET_ACCESS_KEY=<your_aws_secret_key>
AWS_DEFAULT_REGION=us-east-1
```

### Where to Get Credentials

| Credential | Source |
|---|---|
| **MongoDB URL** | [MongoDB Atlas](https://cloud.mongodb.com) → Database → Connect → Drivers |
| **DagsHub Token** | [DagsHub](https://dagshub.com/user/settings/tokens) → Settings → Tokens |
| **AWS Keys** | [AWS Console](https://console.aws.amazon.com/iam) → IAM → Security Credentials |

### GitHub Secrets (for CI/CD)

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION=us-east-1
AWS_ECR_LOGIN_URI=<your-ecr-uri>
ECR_REPOSITORY_NAME=networkssecurity
```

---

## 💻 Usage

### 1. Push Data to MongoDB (First Time Setup)

```bash
python push_data.py
```

Reads `Network_Data/phisingData.csv` → Converts to JSON → Inserts into MongoDB Atlas (`KRISHAI.NetworkData`)

### 2. Train Models (CLI)

```bash
python main.py
```

Runs the full pipeline: Ingest → Validate → Transform → Train 5 models → Log to MLflow → Save best model

### 3. Start the FastAPI Server

```bash
python app.py
```

Server starts at **http://localhost:8000** → auto-redirects to Swagger docs.

---

## 🌐 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Redirects to Swagger UI (`/docs`) |
| `GET` | `/train` | Triggers full training pipeline |
| `POST` | `/predict` | Upload CSV → Get phishing predictions |

### Train Model

```bash
curl http://localhost:8000/train
```

### Predict

```bash
curl -X POST "http://localhost:8000/predict" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@test_data.csv"
```

Returns an **HTML table** with all features + `predicted_column` appended. Results also saved to `prediction_output/output.csv`.

---

## 🐳 Docker Deployment

### Build & Run Locally

```bash
# Build
docker build -t networksecurity:latest .

# Run
docker run -p 8000:8000 --env-file .env networksecurity:latest
```

---

## ☁️ AWS Deployment

### EC2 Setup

```bash
# Install Docker on EC2
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
newgrp docker
```

### Deploy from ECR

```bash
# Login to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ecr-uri>

# Pull & Run
docker pull <ecr-uri>/networkssecurity:latest
docker run -d -p 8000:8000 --env-file .env <ecr-uri>/networkssecurity:latest
```

### S3 Artifact Sync

The training pipeline automatically syncs:
- **Artifacts** → `s3://netwworksecurity/artifact/<timestamp>/`
- **Final Model** → `s3://netwworksecurity/final_model/<timestamp>/`

---

## 🔧 Troubleshooting

| Issue | Solution |
|---|---|
| `MongoDB connection failed` | Check `.env` credentials · Whitelist your IP in Atlas |
| `ModuleNotFoundError` | Run `pip install -e .` to install the package |
| `Model files not found` | Ensure `final_model/model.pkl` and `preprocessor.pkl` exist |
| `MLflow tracking error` | Verify DagsHub token is valid and not expired |
| `Port 8000 already in use` | Kill the existing process or use `--port 8001` |
| `Docker build fails` | Ensure Docker daemon is running |

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **ML Models** | Random Forest, Decision Tree, Gradient Boosting, Logistic Regression, AdaBoost |
| **Hyperparameter Tuning** | GridSearchCV (3-fold CV) |
| **Feature Engineering** | KNNImputer (k=3, uniform weights) |
| **Experiment Tracking** | MLflow + DagsHub |
| **Data Validation** | Kolmogorov–Smirnov test (drift detection) |
| **API Framework** | FastAPI + Uvicorn |
| **Database** | MongoDB Atlas |
| **Serialization** | Pickle (model + preprocessor) |
| **Cloud** | AWS S3, ECR, EC2 |
| **Containerization** | Docker |
| **Metrics** | F1 Score, Precision, Recall, R² Score |

---

## 🙏 Acknowledgments

- **Dataset**: UCI Phishing Websites Dataset
- **Framework**: FastAPI, Scikit-learn, MLflow
- **Database**: MongoDB Atlas
- **MLOps Platform**: DagsHub
- **Cloud**: AWS (S3, ECR, EC2)
- **Original Author**: Krish Naik

---

## 📄 License

This project is for educational and research purposes.

---

<p align="center">
  Built with ❤️ for Network Security
</p>
