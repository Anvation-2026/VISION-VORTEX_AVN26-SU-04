# VoltGrid EV User App

A fresh EV-user-facing FastAPI application.

## Features
- Register / Login / Logout
- Dashboard
- Find nearby charging stations
- Live station lookup through Open Charge Map API when `OCM_API_KEY` is configured
- Local SQLite database for users, vehicles, bookings and charging history
- EV charging constraints: SOC, target SOC, battery capacity, max charging power, connector, deadline and budget
- Charging recommendation engine
- Station details
- Booking
- Profile / vehicle management
- Charging history
- Multiple hyperlink pages
- Responsive professional UI

## Run on Windows

```powershell
cd VoltGrid_EV_User_App
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

If PowerShell blocks activation, use:

```powershell
venv\Scripts\python.exe app.py
```

Then open:
http://127.0.0.1:8000

## Real-world station data

The app is designed to use Open Charge Map as the live station source.

Create an environment variable:

```powershell
$env:OCM_API_KEY="YOUR_OPEN_CHARGE_MAP_API_KEY"
```

Then run the app.

Without an API key, the application still works using a small local demo station dataset so the UI can be tested.

## Important

Do not put API keys directly in frontend JavaScript.


## Sustainability Center

The updated app adds `/sustainability` to the navigation and dashboard. It includes:
- Eco Score
- Renewable energy mix and solar utilization
- Carbon Intelligence
- Peak-load reduction
- Green charging windows
- Cost + carbon optimization
- Battery Health Guard
- Energy Waste Monitor
- Sustainability Opportunity detection
- Optimize for Clean Energy action
- Daily Sustainability Report

The sustainability values are synthetic demo values for the MVP and are labeled/used as operational demo metrics.

## 🔮 Station AI / Predictive Demand

The updated MVP adds **Station Intelligence** for one selected station: **VoltGrid Central Charging Hub, Koramangala, Bengaluru**.

Open:
- `http://127.0.0.1:8000/station-intelligence`

Features:
- Live station status refresh every 30 seconds
- Current vehicles, charger availability, load and solar estimate
- 12-hour EV arrival forecast
- Predicted charging demand in kW
- Peak-load / overload warning against a 100 kW station limit
- Explainable recommendation to shift flexible charging before a predicted peak
- Forecast chart and next-hour operating plan

### Live provider data
If `OCM_API_KEY` is configured, the app uses Open Charge Map station data for the selected station feed. Without an API key, the MVP uses its local station profile and a transparent time/day demand model so the prediction demo still works offline.

The arrival forecast is intentionally transparent rather than falsely claiming a trained ML model. For production, replace `HISTORICAL_ARRIVAL_PROFILE` with the station operator's historical arrival/session data and connect charger/grid/solar telemetry APIs.

## Real-Time Station Telemetry + Arrival Prediction

The Station Intelligence page now persists station telemetry into SQLite (`data/voltgrid.db`) and learns arrival events from positive changes in occupied chargers. The browser refreshes `/api/station/prediction` every 30 seconds, so each refresh can add a new real-time snapshot to the database.

### Connect a real station operator API
Set these environment variables before starting the app:

PowerShell:
```powershell
$env:VOLTGRID_STATION_ID="YOUR_STATION_ID"
$env:VOLTGRID_STATION_TELEMETRY_URL="https://YOUR-OPERATOR-API/station/live"
$env:VOLTGRID_STATION_TELEMETRY_TOKEN="YOUR_TOKEN"
```

The telemetry endpoint should return JSON containing at least:
```json
{
  "vehicles_charging": 4,
  "available_chargers": 2,
  "total_chargers": 6,
  "current_load_kw": 71,
  "solar_kw": 24,
  "grid_capacity_kw": 100,
  "timestamp": "2026-10-09T10:15:00"
}
```

`vehicles_charging`, charger availability and load are persisted on every refresh. A positive occupancy change is recorded as an arrival event, and the forecast progressively switches from the transparent baseline to the observed station history as enough data accumulates.

Important: Open Charge Map is useful for station discovery and metadata, but it should not be represented as a live arrival-count source unless the returned provider data actually contains occupancy/availability. For production, connect the charging-network/operator's telemetry API.

## Real-time database mode

The Station Intelligence module now runs a persistent SQLite database at `data/voltgrid.db` and a background ingestion worker every 30 seconds. When a live source is configured, every station snapshot is stored in `station_snapshots`; positive occupancy changes are stored as arrival events in `station_arrival_events` and are used by the prediction layer.

For a genuinely live station feed, set:

```powershell
$env:VOLTGRID_STATION_TELEMETRY_URL="https://YOUR-OPERATOR-API/station/live"
$env:VOLTGRID_STATION_ID="YOUR_STATION_ID"
$env:VOLTGRID_STATION_TELEMETRY_TOKEN="YOUR_TOKEN"
```

The operator endpoint should return `vehicles_charging`, `available_chargers`, `total_chargers`, and `current_load_kw`, with optional `solar_kw`, `grid_capacity_kw`, `timestamp`, and `status`.

Open Charge Map can be used for station discovery/metadata when `OCM_API_KEY` is configured, but it must not be represented as a dedicated live EV-arrival feed. The dashboard labels the source accordingly. The app never fabricates an external API key.
