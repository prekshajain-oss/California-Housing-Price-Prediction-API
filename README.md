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
Tech StackFramework: FastAPIData & ML: Scikit-Learn, Pandas, Joblib, NumPyValidation: PydanticServer: UvicornGetting StartedPrerequisitesPython 3.10+pip package manager1. Clone the RepositoryBashgit clone [https://github.com/your-username/california-housing-api.git](https://github.com/your-username/california-housing-api.git)
cd california-housing-api
2. Create and Activate a Virtual EnvironmentBash# On macOS/Linux
python3 -m venv .venv
source .venv/bin/activate

# On Windows
python -m venv .venv
.venv\Scripts\activate
3. Install DependenciesBashpip install -r requirements.txt
4. Run the ApplicationBashuvicorn app.main:app --reload
The API will start locally at http://127.0.0.1:8000.Interactive Swagger UI: http://127.0.0.1:8000/docsReDoc Documentation: http://127.0.0.1:8000/redocAPI Endpoints & Usage1. Health CheckMethod: GETPath: /healthDescription: Verifies service uptime and confirms model availability.Example Response:JSON{
  "status": "healthy",
  "model_type": "Random Forest Regressor",
  "expected_features": [
    "MedInc", "HouseAge", "AveRooms", "AveBedrms", 
    "Population", "AveOccup", "Latitude", "Longitude"
  ],
  "mae_usd": 39000
}
2. Single House PredictionMethod: POSTPath: /predictHeader: Content-Type: application/jsonExample Request:JSON{
  "MedInc": 8.3252,
  "HouseAge": 41.0,
  "AveRooms": 6.9841,
  "AveBedrms": 1.0238,
  "Population": 322.0,
  "AveOccup": 2.5555,
  "Latitude": 37.88,
  "Longitude": -122.23
}
Example Response:JSON{
  "predicted_price": "$452,600",
  "raw_prediction": 4.526,
  "confidence_range": "$413,600 - $491,600"
}
3. Batch Prediction via CSVMethod: POSTPath: /predict-fileContent-Type: multipart/form-dataPayload: Upload a .csv file with the required columns.cURL Example:Bashcurl -X POST "[http://127.0.0.1:8000/predict-file](http://127.0.0.1:8000/predict-file)" \
  -H "accept: text/csv" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@tests/sample_test.csv" \
  --output predictions.csv
Input Data SchemaAny single payload or CSV file uploaded must contain these exact 8 features:Column NameTypeValid RangeDescriptionMedIncfloat> 0Median income in block group (tens of thousands USD)HouseAgefloat> 0Median house age in block groupAveRoomsfloat> 0Average number of rooms per householdAveBedrmsfloat> 0Average number of bedrooms per householdPopulationfloat> 0Total block group populationAveOccupfloat> 0Average number of household membersLatitudefloat32.0 to 42.0Latitude coordinate (California bounds)Longitudefloat-125.0 to -114.0Longitude coordinate (California bounds)LicenseDistributed under the MIT License. See LICENSE for more details.

