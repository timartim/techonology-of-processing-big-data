# Cat V dogs

This repository contains a simple **Machine Learning Engineering pipeline** for binary image classification (**cats vs dogs**).  
The system includes model training, inference API, experiment tracking, Docker deployment, and CI/CD pipelines.

---

# Project Overview

The model classifies images into two classes:

- `cat`
- `dog`

Architecture:

```
Image
   ↓
ShuffleNet V2 (feature extractor)
   ↓
Embedding vector
   ↓
Logistic Regression
   ↓
Probability of "dog"
```

Key components:

- Feature extractor: **ShuffleNet V2 x0.5 (pretrained)**
- Classifier: **Logistic Regression**
- API: **FastAPI**
- Containerization: **Docker + Docker Compose**
- CI/CD: **GitHub Actions**
- Data versioning: **DVC**

---

# Installation

Clone repository:

```bash
git clone https://github.com/<your_repo>/mle-template.git
cd mle-template
```

Create virtual environment:

```bash
python -m venv venv
source venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

---

# Dataset

The model expects images named in the following format:

```
cat.123.jpg
dog.456.jpg
```

Class is inferred from filename prefix.

Example dataset structure:

```
data/
    cat.1.jpg
    cat.2.jpg
    dog.1.jpg
    dog.2.jpg
```

---

# CLI Usage

The application provides a CLI interface for:

- training models
- running inference
- batch prediction

General command:

```bash
python main.py --mode <mode> --path <path>
```

---

# Training

Train model on a dataset:

```bash
python main.py \
    --mode train \
    --path data/train \
    --device cpu
```

Optional parameters:

```bash
--data_frac 0.25
--test_size 0.2
--seed 42
--experiments_dir experiments
--best_dir experiments
```

Training will:

- generate embeddings with ShuffleNet
- train Logistic Regression
- compute metrics
- save experiment artifacts

Artifacts created:

```
experiments/
    exp_0001_YYYY-MM-DD_HH-MM-SS/
        model.pkl
        report.json
```

Best model is saved as:

```
experiments/model.pkl
experiments/model_metrics.json
```

---

# Predict single image

```bash
python main.py \
    --mode single \
    --path image.jpg
```

Example output:

```json
{
  "mode": "single",
  "path": "image.jpg",
  "prediction": [1]
}
```

---

# Predict directory

Run predictions for all images in a directory:

```bash
python main.py \
    --mode directory \
    --path images/
```

Optional flags:

```bash
--recursive
--return_paths
--skip_errors
```

---

# API

The project provides a REST API built with **FastAPI**.

Run API locally:

```bash
uvicorn src.api:app --host 0.0.0.0 --port 8001
```

Main endpoint:

```
POST /predict
```

Request:

```
multipart/form-data
file=<image>
```

Example request using curl:

```bash
curl -X POST \
  -F "file=@cat.jpg" \
  http://localhost:8001/predict
```

Example response:

```json
{
  "dog_probability": 0.13
}
```

Swagger documentation:

```
http://localhost:8001/docs
```

---

# Docker

Build Docker image:

```bash
docker build -t catdog-api .
```

Run container:

```bash
docker run -p 8001:8001 catdog-api
```

---

# Docker Compose

Start service:

```bash
docker compose up --build
```

API will be available at:

```
http://localhost:8001
```

Swagger:

```
http://localhost:8001/docs
```

---

# CI/CD

CI/CD pipelines are implemented using **GitHub Actions**.

Workflows:

```
.github/workflows/
    ci.yml
    cd.yml
```

### CI pipeline

Runs on every commit:

- builds Docker image
- runs unit tests
- evaluates model metrics

### CD pipeline

Runs after merge into `main` branch:

- pulls Docker image
- launches API
- runs functional API tests

Example functional test:

```bash
curl -X POST -F "file=@dog.jpg" http://localhost:8001/predict
```

Validation rule:

```
dog image  -> probability > 0.5
cat image  -> probability < 0.5
```

---

# DVC

Data versioning is implemented with **DVC** and a remote storage.

Initialize DVC:

```bash
dvc init
```

Pull dataset:

```bash
dvc pull
```

---

# Project Structure

```
.
├── src
│   ├── api.py
│   ├── model.py
│   └── logger.py
│
├── experiments
│
├── .github
│   └── workflows
│
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── config.ini
└── README.md
```

---

# Model Metrics

Example metrics on validation set:

| Metric | Score |
|------|------|
| Precision | 0.96 |
| Recall | 0.97 |
| F1 | 0.96 |
| Accuracy | 0.96 |

---

# Author

Artemiy Korniliev  
ITMO University  
Big Data Infrastructure — Spring 2026