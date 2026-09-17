import io
from pathlib import Path
import joblib
import pandas as pd
from fastapi import FastAPI, File, HTTPException, UploadFile, status
from fastapi.responses import StreamingResponse
from pydantic import BaseModel, Field

BASE_DIR = Path(__file__).resolve().parent

# Features expected by the trained model (exact order matters)
FEATURES = [
    "MedInc",
    "HouseAge",
    "AveRooms",
    "AveBedrms",
    "Population",
    "AveOccup",
    "Latitude",
    "Longitude",
]

# Model loading with robust path resolution
MODEL_PATH = BASE_DIR / "house_model.joblib"
try:
    model = joblib.load(MODEL_PATH)
except Exception as e:
    model = None

app = FastAPI(
    title="California Housing Price Prediction API",
    version="1.0.0",
    description="Predict median California housing prices using single payloads or batch CSV uploads.",
)


class HouseFeatures(BaseModel):
    MedInc: float = Field(gt=0, description="Median income in block group")
    HouseAge: float = Field(gt=0, description="Average house age in block group")
    AveRooms: float = Field(
        gt=0, description="Average number of rooms per household"
    )
    AveBedrms: float = Field(
        gt=0, description="Average number of bedrooms per household"
    )
    Population: float = Field(gt=0, description="Total block group population")
    AveOccup: float = Field(
        gt=0, description="Average number of household members"
    )
    Latitude: float = Field(ge=32, le=42, description="Latitude coordinate")
    Longitude: float = Field(
        ge=-125, le=-114, description="Longitude coordinate"
    )


@app.get("/")
def home():
    return {
        "status": "running",
        "message": "California Housing Prediction API",
        "endpoints": {
            "predict_single": "POST /predict",
            "predict_batch_csv": "POST /predict-file",
            "docs": "/docs",
        },
    }


@app.get("/health")
def health():
    return {
        "status": "healthy" if model is not None else "model_missing",
        "model_type": "Random Forest Regressor",
        "expected_features": FEATURES,
        "mae_usd": 39000,
    }


@app.post("/predict")
def predict(house: HouseFeatures):
    if model is None:
        raise HTTPException(
            status_code=status.HTTP_503_SERVICE_UNAVAILABLE,
            detail="Model file not loaded.",
        )

    try:
        # Dump schema to dataframe and reindex to guarantee feature order
        input_data = pd.DataFrame([house.model_dump()])[FEATURES]
        raw_pred = float(model.predict(input_data)[0])
        price_usd = raw_pred * 100000

        return {
            "predicted_price": f"${price_usd:,.0f}",
            "raw_prediction": round(raw_pred, 4),
            "confidence_range": f"${max(0, price_usd - 39000):,.0f} - ${price_usd + 39000:,.0f}",
        }
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Prediction failed: {str(e)}",
        )


@app.post("/predict-file")
async def predict_file(file: UploadFile = File(...)):
    if model is None:
        raise HTTPException(
            status_code=status.HTTP_503_SERVICE_UNAVAILABLE,
            detail="Model file not loaded.",
        )

    if not file.filename.endswith(".csv"):
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Invalid file format. Please upload a .csv file.",
        )

    try:
        contents = await file.read()
        df = pd.read_csv(io.BytesIO(contents))
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"Failed to read CSV: {str(e)}",
        )

    if df.empty:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Uploaded CSV file contains no data.",
        )

    missing_cols = [col for col in FEATURES if col not in df.columns]
    if missing_cols:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"Missing required columns: {missing_cols}",
        )

    try:
        input_data = df[FEATURES]
        raw_preds = model.predict(input_data)

        df["predicted_price_usd"] = (raw_preds * 100000).round(2)
        df["predicted_price_formatted"] = df["predicted_price_usd"].apply(
            lambda x: f"${x:,.0f}"
        )

        csv_output = df.to_csv(index=False)
        return StreamingResponse(
            io.BytesIO(csv_output.encode("utf-8")),
            media_type="text/csv",
            headers={
                "Content-Disposition": "attachment; filename=predictions.csv"
            },
        )
    except Exception as e:
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"Batch processing error: {str(e)}",
        )
