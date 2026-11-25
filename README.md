# 15-City Air Quality Index (AQI) Predictor & Dashboard

A full-stack atmospheric intelligence system featuring a **FastAPI backend** running a trained scikit-learn machine learning pipeline and an **Angular 22 frontend** dashboard with real-time pollutant telemetry and EPA AQI classification.

---

## 🎨 UI Color Palette

The interface was designed to match the specified custom palette:
- **`#87CEEB` (Sky Blue)**: Accents, borders, badges, atmospheric highlights.
- **`#FFFFFF` (White)**: Clean card backgrounds and high-contrast typography.
- **`#FFD700` (Gold / Yellow)**: Moderate category badge and warning highlights.
- **`#8B0000` (Dark Red)**: Hazardous air quality status and error action buttons.
- **`#4682B4` (Steel Blue)**: Primary headers, navigation, structure, and icons.

Along with standard 6-tier EPA AQI category indicator colors:
- `Good` (0–50): `#22c55e` (Green)
- `Moderate` (51–100): `#FFD700` (Gold / Yellow)
- `Unhealthy for Sensitive Groups` (101–150): `#f97316` (Orange)
- `Unhealthy` (151–200): `#ef4444` (Red)
- `Very Unhealthy` (201–300): `#a855f7` (Purple)
- `Hazardous` (301+): `#8B0000` (Dark Red)

---

## 🏗️ Architecture

```
Angular 22 App (Port 4200)
    │
    │  GET /api/cities
    │  GET /api/predict?city=...
    │  GET /api/compare
    ▼
FastAPI Service (Port 8000)
    │
    ├── 1. Validate City (15 US Metros)
    ├── 2. Fetch Pollutants + Weather (OpenWeatherMap / Calibrated Demo Telemetry)
    ├── 3. Unit Conversion (CO ug/m³ -> mg/m³ via UNIT_FACTORS)
    ├── 4. Feature Assembly (['City', 'PM2.5', 'PM10', 'Ozone', 'NO2', 'CO', 'SO2', 'Temperature', 'Humidity', 'Year', 'Month', 'Day', 'DayOfWeek'])
    ├── 5. ML Model Inference (`best_aqi_model.joblib` ExtraTreesRegressor Pipeline)
    ├── 6. EPA PM2.5 Reference Calculation & Gap Analysis
    └── 7. Category & Public Health Advisory Message Generation
```

---

## 🏙️ Supported Cities

1. Austin
2. Charlotte
3. Chicago
4. Columbus
5. Dallas
6. Fort Worth
7. Houston
8. Jacksonville
9. Los Angeles
10. New York
11. Philadelphia
12. Phoenix
13. San Antonio
14. San Diego
15. San Jose

---

## ⚡ Quick Start

### 1. Launch Everything with One Click
Double-click **`start_all.bat`** in the project root. It opens separate terminal windows for the FastAPI backend and Angular frontend. The frontend launcher installs npm dependencies the first time if needed.

### 2. Or Run Separately:

Install the backend dependencies once, then start the backend in a terminal:
```powershell
cd backend
python -m pip install -r requirements.txt
py -m uvicorn main:app --host 127.0.0.1 --port 8000 --reload
```
- Interactive Swagger documentation: **http://localhost:8000/docs**
- Health check: **http://localhost:8000/**

In a second terminal, start the frontend:
```powershell
cd frontend
npm install
npm start
```
- Open browser at: **http://localhost:4200**

Install Python 3 and Node.js (including npm) before running the application. If a command is not recognized in the terminal, add its installation directory to `PATH` and reopen the terminal.

---

## 📡 API Contract

| Endpoint | Method | Description |
|---|---|---|
| `/api/cities` | `GET` | Returns array of the 15 supported city names |
| `/api/predict?city=Philadelphia` | `GET` | Returns predicted AQI, category, colour, advice, full message, input breakdown, reference PM2.5 AQI, and difference |
| `/api/compare` | `GET` | Returns all 15 cities ranked from worst AQI to best AQI |

### Status Codes:
- `200 OK`: Successful response.
- `404 Not Found`: Unknown or unsupported city.
- `502 Bad Gateway`: External weather/telemetry service unreachable.

---

## 🧪 Model Details & Unit Conversion

- **Model File**: `backend/best_aqi_model.joblib`
- **Algorithm**: `sklearn.pipeline.Pipeline` containing a `ColumnTransformer` (median imputation + standard scaler for numeric features, and most frequent imputation + one-hot encoding for `City`), followed by an `ExtraTreesRegressor`.
- **Unit Factors**:
  - OpenWeatherMap returns CO in $\mu\text{g/m}^3$ (~ 1500–3000). The model's scaler mean is 2.17 (in $\text{mg/m}^3$ / ppm). The backend automatically applies `UNIT_FACTORS['CO'] = 0.001` to maintain feature consistency.
