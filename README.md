# ITS120L-PROJECT

# YangCorp Tax Platform

This repository contains the production-ready YangCorp Tax AI platform. The project includes:

- Marketing landing page (`index.html`)
- Secure signup / login flows with role selection
- Redesigned dashboards for Admin, Accountant, and Client personas
- Node.js + Express backend with SQLite persistence, migrations, seed data, and Hugging Face AI integration

## Getting Started

### Prerequisites

- Node.js 18+
- npm 8+

### Install dependencies

```bash
cd server
npm install
```

### Configure environment

Create `server/.env` (or export environment variables) with your Hugging Face token:

```bash
   HF_TOKEN=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
     # Optional override:
     # HF_MODEL=meta-llama/Meta-Llama-3-8B-Instruct
# Optional: override the default model
# HF_MODEL=meta-llama/Meta-Llama-3-8B-Instruct
```

> **Security tip:** never commit your real token. The project loads environment variables via `dotenv` and fails gracefully if no token is provided (dashboards will show an “AI unavailable” message).

### Uploading CSV data

Admins and accountants can upload CSV files directly from their dashboards. Expected columns:

- `client_name` (or `client_email`)
- `tax_year`
- `revenue`
- `expenses`
- `deductions`
- `payments`

Additional columns are ignored. If `client_id` is supplied in the upload form, all rows are associated with that client; otherwise, clients are resolved/created by name/email. Each upload creates a historical snapshot, runs AI forecasting/anomaly detection, and refreshes dashboards immediately.

Example CSV:

```csv
client_name,tax_year,revenue,expenses,deductions,payments
Harper Media LLC,2023,850000,420000,75000,60000
Harper Media LLC,2022,780000,380000,72000,55000
Summit Retail Group,2023,640000,310000,58000,45000
Summit Retail Group,2022,605000,295000,54000,42000
```

### Run the server

```bash
npm start
```

The server automatically applies migrations, seeds demo data (if no uploads exist), and exposes the web app at `http://localhost:5175`.

### Demo Accounts

| Role       | Email                     | Password       |
|------------|---------------------------|----------------|
| Admin      | `admin@yangcorp.com`      | `Password123!` |
| Accountant | `accountant@yangcorp.com` | `Password123!` |
| Client     | `client@harpermedia.com`  | `Password123!` |

Use these accounts to explore the dashboards immediately after start-up. You can invite new users from the Admin dashboard.

## Usage Overview

1. **Sign in** as Admin or Accountant.
2. **Upload** CSV exports via the “Upload data” section (Admin/Accountant dashboards).
3. The backend parses the CSV, stores historical financial records, triggers Hugging Face to generate forecasts/anomalies, and refreshes dashboard widgets automatically.
4. **Clients** log in to see their personalized forecasts, pending tasks, filings, and upload history.

Uploads and AI outputs are stored in SQLite (`server/data/app.db`). Files are saved under `server/data/uploads/`.

## Project Structure

- `index.html`, `assets/` – marketing site and shared styling/scripts
- `pages/` – auth pages and dashboards for each role
- `server/index.js` – Express app, routes, upload handling, AI endpoints
- `server/lib/` – helpers for database access, finance parsing, AI generation
- `server/migrations/` – SQLite schema migrations (run automatically on start)

## API Summary

- `POST /auth/signup`, `POST /auth/login`, `POST /auth/logout`
- `POST /api/uploads` – CSV uploads (Admin/Accountant)
- `GET /api/uploads` – upload history (Admin/Accountant)
- `GET /api/dashboard/summary` – role-based dashboard data
- `GET /api/dashboard/ai/{admin|accountant|client}` – Hugging Face insights
- Crud endpoints for clients, filings, tasks, documents, and user invites (see `server/index.js` for full list)

## Development Notes

- Authentication uses cookie-based sessions stored in SQLite.
- Passwords are hashed with bcrypt.
- AI endpoints call Hugging Face Inference API with your `HF_TOKEN`.
- The repo includes seed data so the dashboards aren’t empty on first run; uploading real CSVs replaces demo metrics.

## Deployment

A simple deployment flow:

1. Set environment variables (`HF_TOKEN`, optional `HF_MODEL`, `SESSION_SECRET`).
2. Provision persistent storage for SQLite or swap to PostgreSQL (update `lib/db.js`).
3. Run `npm install` and `npm start` (or use a process manager/containers).
4. Serve the static frontend (`index.html`, `assets/`, `pages/`) via the same Express app or a CDN.

Feel free to extend with Docker, PostgreSQL, or additional AI providers as needed.
