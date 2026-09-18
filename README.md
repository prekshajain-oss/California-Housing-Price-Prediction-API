# California Housing Price Prediction API

A production-ready REST API built with **FastAPI**, **Pandas**, and **Scikit-Learn** that serves a Random Forest regression model to predict California median home values. 

The API supports single-record JSON inference with strict schema validation, as well as high-throughput batch CSV processing with in-memory streaming responses.


## Features

- **Single Prediction Endpoint (`/predict`):** Validates geospatial boundaries (California latitude/longitude) and socioeconomic factors via Pydantic before scoring.
- **Batch CSV Processing (`/predict-file`):** Ingests raw tabular data, verifies required columns, appends predictions, and streams back the populated CSV without intermediate disk writes.
- **Health & Monitoring (`/health`):** Exposes service health, model architecture, expected feature schema, and error benchmarks ($39,000 MAE).
- **Interactive Documentation:** Automatic Swagger UI and ReDoc support out of the box.


## Project Structure

```text
california-housing-api/
├── app/
│   ├── __init__.py
│   ├── main.py               # FastAPI routes and logic
│   └── house_model.joblib    # Serialized scikit-learn model
├── tests/
│   ├── __init__.py
│   └── sample_test.csv       # Sample file for testing batch endpoint
├── .gitignore
├── requirements.txt
└── README.md
```
## Tech Stack

Framework: FastAPI

Data & ML: Scikit-Learn, Pandas, Joblib, NumPy

Validation: Pydantic

Server: Uvicorn

Getting Started
Prerequisites
Python 3.10+

pip package manager

**1. Clone the Repository**
git clone [https://github.com/your-username/california-housing-api.git](https://github.com/your-username/california-housing-api.git)
cd california-housing-api

**2. Create and Activate a Virtual Environment**
# On macOS/Linux
python3 -m venv .venv
source .venv/bin/activate

# On Windows
python -m venv .venv
.venv\Scripts\activate

**3. Install Dependencies**
pip install -r requirements.txt

**4. Run the Application**
uvicorn app.main:app --reload
The API will start locally at http://127.0.0.1:8000.

Interactive Swagger UI: http://127.0.0.1:8000/docs

ReDoc Documentation: http://127.0.0.1:8000/redoc

## API Endpoints & Usage
1. Health Check
Method: GET

Path: /health

Description: Verifies service uptime and confirms model availability.

## EXAMPLE RESPONSE

{
  "status": "healthy",
  "model_type": "Random Forest Regressor",
  "expected_features": [
    "MedInc", "HouseAge", "AveRooms", "AveBedrms", 
    "Population", "AveOccup", "Latitude", "Longitude"
  ],
  "mae_usd": 39000
}

2. Single House Prediction
Method: POST

Path: /predict

Header: Content-Type: application/json

Example Request:
