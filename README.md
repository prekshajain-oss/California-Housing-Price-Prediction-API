                                               California-Housing-Price-Prediction-API
                                             
Production-ready FastAPI service serving a Random Forest regression model to predict California median home values. Supports real-time single-record JSON inference and high-throughput batch CSV processing with direct download.

## Features

Bullet list: single prediction endpoint, batch CSV processing with direct file download, Pydantic input validation, interactive Swagger docs.

## Tech Stack

Python, FastAPI, Uvicorn, Pandas, Scikit-Learn, Joblib, Pydantic.

## Project Structure

ASCII directory tree (matching the structure above).

## Getting Started

### Prerequisites (Python 3.10+)

### Installation (git clone, python -m venv .venv, pip install -r requirements.txt)

### Running Locally (uvicorn app.main:app --reload)

## API Endpoints

GET /health (Health status and model metadata)

POST /predict (Example JSON request payload and response)

POST /predict-file (Details on required CSV columns and return format)

## Expected CSV Format

A table or comma-separated list specifying the 8 mandatory input column headers.
