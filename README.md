# MLOps Lab 4 - Wine Quality Prediction

**Author:** Priyanka Kumari  
**Roll No:** 2022BCD0057

## Project Overview

This project implements a complete MLOps pipeline for predicting wine quality using machine learning. It includes model training, evaluation, and deployment using FastAPI and Docker containerization.
# Wine Quality — MLOps Lab

Small ML project that trains a Lasso regression model on the red wine quality dataset and exposes a prediction API with FastAPI.

**Author:** Priyanka Kumari — Roll No: 2022BCD0057

## Repository layout

- `Dockerfile` — container image definition
- `Jenkinsfile`, `Dockerfile.jenkins` — CI pipeline examples
- `requirements.txt` — Python dependencies
- `app/` — FastAPI app and runtime artifacts
   - `app/app.py` — prediction endpoint (loads model from `app/artifacts/model.pkl`)
   - `app/artifacts/` — model and metrics produced by training
- `data/winequality-red.csv` — dataset (semicolon-delimited CSV)
- `script/train.py` — training + hyperparameter search, saves artifacts to `app/artifacts`
- `output/` — (optional) other generated outputs

## Quickstart (local)

1. Create and activate a virtual environment, then install dependencies:

```powershell
python -m venv venv
venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

2. Train the model (writes `model.pkl` and `metrics.json` to `app/artifacts`):

```powershell
python script/train.py
```

3. Run the API (serves POST `/predict`):

```powershell
uvicorn app.app:app --host 0.0.0.0 --port 8000
```

4. Example request:

```bash
curl -X POST "http://localhost:8000/predict" \
   -H "Content-Type: application/json" \
   -d '{
      "fixed_acidity": 7.4,
      "volatile_acidity": 0.7,
      "citric_acid": 0.0,
      "residual_sugar": 1.9,
      "chlorides": 0.076,
      "free_sulfur_dioxide": 11.0,
      "total_sulfur_dioxide": 34.0,
      "density": 0.9978,
      "pH": 3.51,
      "sulphates": 0.56,
      "alcohol": 9.4
   }'
```

## Notes

- The training script saves artifacts to `app/artifacts` (model and metrics). The API expects `app/artifacts/model.pkl` to exist.
- `script/train.py` uses a `StandardScaler` + `Lasso` pipeline and performs a small grid search over `alpha`.
- Data is read from `data/winequality-red.csv` (semicolon-separated).

## Docker

Build and run the container (optional):

```bash
docker build -t wine-quality-app .
docker run -p 8000:8000 wine-quality-app
```

## CI / Notes

- A `Jenkinsfile` and `Dockerfile.jenkins` are included as examples for CI/CD integration.
- Consider adding input validation, logging, and model versioning for production use.

---

Updated README to reflect current layout and how to run training and the API.
### Prerequisites

