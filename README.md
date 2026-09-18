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


