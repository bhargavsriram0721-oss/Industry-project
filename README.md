# AIOps Sentinel — Real Local Implementation

This version replaces the simulated dashboard with a local host monitoring implementation.

## Architecture
- **Frontend:** React + Vite dashboard with responsive dark/purple command-center design and animated digital globe.
- **Backend:** FastAPI local agent API.
- **Telemetry:** `psutil` reads actual CPU, memory, disk, network counters and process count from the computer running the backend.
- **Storage:** SQLite persists every telemetry sample and incident in `backend/aiops_sentinel.db`.
- **Anomaly detection:** Isolation Forest is trained on the local host's rolling telemetry after 30 observations. Transparent resource thresholds are available from the first sample.
- **Incident management:** Create, investigate and resolve incidents through persistent API endpoints.
- **CSV analysis:** Upload a telemetry CSV for assessment.
- **Software observability:** The Software Monitor page accepts application log lines, classifies warning/error patterns, stores events in SQLite and creates incidents for detected issues.
- **HTTP service checks:** Check an authorized application/API URL for availability, HTTP status and response latency (5-second timeout); failed checks create incidents.
- **Software event history:** Review recent stored log events and service checks in the dashboard.

## Start on Windows
Open two VS Code terminals.

### Terminal 1 — Backend
```powershell
cd backend
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```
If `py` is not recognized, use `python` instead. If PowerShell blocks `npm.ps1`, that does not affect this backend command.

### Terminal 2 — Frontend
```powershell
cd frontend
npm.cmd install
npm.cmd run dev
```
Open http://127.0.0.1:5173

API documentation: http://127.0.0.1:8000/docs

## Software monitoring quick test
1. Start the backend and frontend as described above.
2. Open **Software Monitor** in the left navigation.
3. Keep the default endpoint `http://127.0.0.1:8000/health` and click **Check service**. A successful local backend should return HTTP 200.
4. Leave the sample log lines in the analyzer and click **Analyze logs**. It should store one ERROR and one WARNING event and create corresponding incidents.
5. Review the event list and the Incidents page.

## Software monitoring API
- `POST /api/software/logs` — ingest up to 200 log lines for classification.
- `GET /api/software/events` — retrieve stored warning/error events.
- `POST /api/software/check` — perform a one-off HTTP GET check for a service URL.
- `GET /api/software/checks` — retrieve recent service checks.

Example log ingestion body:
```json
{
  "service": "Orders API",
  "source": "orders-api.log",
  "lines": ["ERROR database connection refused", "WARNING retrying request"]
}
```

## Current scope and limitations
This build provides manual log ingestion and one-off HTTP endpoint checks. It does not yet automatically instrument every application, continuously poll registered services, collect distributed traces, detect memory leaks from runtime heap profiles, or prove a root cause. The root-cause field is an evidence-based note from the matched log line or failed endpoint check. Only monitor systems you own or are authorized to test. Keep the backend bound to `127.0.0.1`; do not expose this unauthenticated development API to the public internet.

## Important
The API monitors the machine on which the backend is running. Run the backend on the Windows computer you want to monitor. Keep it bound to localhost. This is a functional local monitoring build, not yet a secured multi-user/cloud monitoring service. The Isolation Forest score is an anomaly score, not a calibrated probability of future failure. Root-cause inference across distributed applications requires application logs, traces, dependency topology and labelled incidents; this local build does not pretend to have those sources.
