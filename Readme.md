# SynthLab — Automated Test Data Generator

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![React](https://img.shields.io/badge/React-Vite-61DAFB?logo=react&logoColor=black)
![Firebase](https://img.shields.io/badge/Firebase-Auth%20%7C%20Firestore-FFCA28?logo=firebase&logoColor=black)
![Google Cloud](https://img.shields.io/badge/Cloud%20Run-GCP-4285F4?logo=googlecloud&logoColor=white)

SynthLab creates realistic, privacy-safe synthetic datasets for development and QA. Sign in to a private workspace, define a table, preview generated rows, and export a CSV. The app never reads production data.

---

## 📦 Download the Full Project

The full project is large, so the complete source package is hosted on Google Drive instead of this repository.

**👉 [Download SynthLab from Google Drive](https://drive.google.com/drive/folders/1laKsmrKBmaf64jp976Fxcq-GKA6nyQol?usp=sharing)**

How to use it:

1. Open the link above and download the `.zip` file.
2. Extract it to a folder (for example `SynthLab/`).
3. Follow [Start locally on Windows](#start-locally-on-windows) below.

> Make sure the Drive link is set to **"Anyone with the link → Viewer"** so everyone can download it.

---

## ✨ Features

- Firebase email-link signup and login, with signed-out access blocked from workspace data. Password login is also available.
- Account settings for profile details, password changes, activity counts, and sign out.
- Firebase ID-token verification on the API, followed by an HTTP-only app session.
- SQLite and in-memory storage for local development; selectable Firestore and Cloud Storage adapters for Cloud Run.
- Firestore accounts, hashed passwords, revocable sessions, account-owned schemas, and shared rate limits in production mode.
- Dataset generation with 12 field types, previews, and CSV downloads.
- Per-account access checks for schemas, generated dataset previews, and downloads.
- Responsive dashboard with a subtle pointer glow on devices with a mouse.
- Input validation, allowed-origin checks, and shared request limits when Firestore is enabled.

## 🛠 Technology

| Layer | Stack |
| --- | --- |
| Frontend | React, Vite |
| Backend | Python, FastAPI, Pydantic, Pandas, NumPy, Faker |
| Accounts & schemas | SQLite (local), Cloud Firestore (production) |
| Temporary datasets | Process memory (local), private Google Cloud Storage (production) |
| Export | CSV, with 15-minute API expiry |

## 📁 Project Structure

```text
SynthLab/
├── backend/            # FastAPI app, tests, Cloud Run config
├── frontend/           # React + Vite app
├── firebase.json       # Firebase Hosting config
├── docker-compose.yml  # Docker setup
└── .env.example
```

---

## Start locally on Windows

Use Python 3.10 or newer and Node.js 20.19 or newer. Open two PowerShell terminals in the project directory.

### Backend

```powershell
cd backend
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
Copy-Item .env.example .env
uvicorn app.main:app --reload --port 8000
```

The backend creates `backend/data/synthlab.sqlite3` automatically. The API is at `http://localhost:8000` and interactive docs are at `http://localhost:8000/docs`.

### Frontend

In a second terminal:

```powershell
cd frontend
Copy-Item .env.example .env
npm install
npm run dev
```

Open the local Vite URL, usually `http://localhost:5173`. Create an account from the sign-in screen to enter the workspace. The account menu in the workspace header opens account settings.

---

## Accounts and data

Existing password accounts require at least 10 characters, including a letter and a number. Passwords are stored as Argon2 hashes. Firebase email-link users do not set a SynthLab password. The backend issues a random, HTTP-only, SameSite=Lax cookie with a seven-day lifetime; only a hash of the session token is stored, in SQLite locally or Firestore in production. Logging out revokes that session. Changing a password revokes other active sessions and keeps the current device signed in.

With the local defaults, SQLite stores account details, password hashes, session records, generation counts, and saved schemas. Generated CSVs stay in server memory and expire after 15 minutes. In production mode, Firestore stores account/session/schema data and shared rate limits, while private Cloud Storage holds generated CSVs and preview metadata under the `synthlab/datasets/` prefix. The API checks ownership and expiry on every dataset read and download. Each dataset and schema is scoped to its owner.

The default database path is `backend/data/synthlab.sqlite3` when the backend is started from `backend/`. Set `DATABASE_PATH` to choose another location. Do not place the database or secret credentials in a public web directory.

## Environment settings

Copy the examples to `.env` files before changing settings, and **never commit credentials**.

| Variable | Purpose |
| --- | --- |
| `CORS_ORIGINS` | Comma-separated frontend origins allowed to call the API. |
| `PERSISTENCE_BACKEND` | `sqlite` locally or `firestore` for durable accounts, sessions, schemas, and shared rate limits. |
| `DATASET_STORAGE_BACKEND` | `memory` locally or `gcs` for Cloud Storage-backed temporary CSVs. |
| `DATABASE_PATH` | SQLite database path. |
| `DATASET_GCS_BUCKET` | Private Google Cloud Storage bucket used when `DATASET_STORAGE_BACKEND=gcs`. |
| `DATASET_GCS_PREFIX` | Object prefix for generated data. Defaults to `synthlab/datasets`. |
| `COOKIE_SECURE` | Set to `true` when served over HTTPS. Keep `false` for local HTTP development. |
| `FIREBASE_PROJECT_ID` | Firebase project ID used to verify email-link tokens and connect to Firestore. |
| `FIREBASE_CREDENTIALS` | Optional path to a service-account JSON file; alternatively use `GOOGLE_APPLICATION_CREDENTIALS`. |
| `VITE_API_BASE_URL` | Optional API origin used by the frontend. Derived from the local hostname when empty. |
| `VITE_FIREBASE_*` | Firebase Web app values for email-link authentication. These client values are public; never put service-account keys in frontend variables. |

For local development, copy `backend/.env.example` to `backend/.env` and `frontend/.env.example` to `frontend/.env`. For Docker Compose, copy the root `.env.example` to `.env`.

## Firebase email-link sign-in

The frontend uses Firebase's modular JavaScript SDK. The API verifies Firebase ID tokens with Firebase Admin and then creates SynthLab's HTTP-only session cookie; a Firebase client token is never used as the API session.

1. In Firebase Console, open **Authentication → Sign-in method** and enable **Email/Password** and **Email link (passwordless sign-in)**.
2. `localhost` and `127.0.0.1` are authorized by default. Add your custom domain under **Authentication → Settings → Authorized domains** before using it.
3. Copy the Firebase Web app's API key, auth domain, project ID, storage bucket, messaging sender ID, and app ID into `frontend/.env` as `VITE_FIREBASE_*` values.
4. Set `FIREBASE_PROJECT_ID` to the same project in `backend/.env`. Local token verification uses Firebase's public signing keys. In production, use the hosting platform's Application Default Credentials. If another Firebase Admin feature needs a local credential file, keep it outside the repository and set `FIREBASE_CREDENTIALS`.
5. Restart the frontend and backend after changing environment files. Email links return to the current app origin. The app stores the email temporarily on the device that requested the link and asks for it again if the link opens elsewhere.

## Firebase Hosting deployment

Firebase Hosting can serve the built frontend. The root `firebase.json` builds `frontend/dist` and routes `/api/**`, `/docs`, and `/openapi.json` to a Cloud Run service named `synthlab-api` in `asia-south1`. `firebase deploy` does **not** deploy the FastAPI backend automatically. Deploy that service first and configure its environment variables before running:

```powershell
firebase deploy --only hosting
```

## Production deployment: Firestore, Cloud Storage, and Cloud Run

This deployment creates Google Cloud resources and may incur charges. The Firebase project must have billing enabled, and you need permission to enable APIs, create a Firestore database and bucket, grant service-account roles, and deploy Cloud Run.

1. Install the Google Cloud CLI, authenticate, and enable the required APIs:

   ```powershell
   gcloud auth login
   gcloud config set project YOUR_PROJECT_ID
   gcloud services enable run.googleapis.com cloudbuild.googleapis.com artifactregistry.googleapis.com firestore.googleapis.com storage.googleapis.com --project YOUR_PROJECT_ID
   ```

2. Create the default Firestore database if the project does not already have one:

   ```powershell
   gcloud firestore databases create --database="(default)" --location=asia-south1 --type=firestore-native --project=YOUR_PROJECT_ID
   ```

   The Firestore rules in this repository deny direct browser access. Firebase Admin calls from Cloud Run use IAM and bypass those rules. Review the rules before deployment if another app in the same Firebase project uses client-side Firestore access.

3. Create a private regional bucket dedicated to SynthLab datasets. Uniform bucket-level access and public access prevention keep generated files private; soft delete is disabled and the lifecycle file removes objects older than one day.

   ```powershell
   $datasetBucket = "REPLACE_WITH_A_GLOBALLY_UNIQUE_BUCKET_NAME"
   gcloud storage buckets create "gs://$datasetBucket" --project=YOUR_PROJECT_ID --location=asia-south1 --uniform-bucket-level-access --public-access-prevention --soft-delete-duration=0
   gcloud storage buckets update "gs://$datasetBucket" --lifecycle-file=backend/storage-lifecycle.json
   ```

4. Create the Cloud Run identity and grant access only to Firestore and the dataset bucket:

   ```powershell
   gcloud iam service-accounts create synthlab-api --project=YOUR_PROJECT_ID
   $serviceAccount = "serviceAccount:synthlab-api@YOUR_PROJECT_ID.iam.gserviceaccount.com"
   gcloud projects add-iam-policy-binding YOUR_PROJECT_ID --member=$serviceAccount --role=roles/datastore.user
   gcloud storage buckets add-iam-policy-binding "gs://$datasetBucket" --member=$serviceAccount --role=roles/storage.objectAdmin
   ```

5. Copy `backend/cloud-run.env.yaml.example` to `backend/.env.production.yaml`, set `DATASET_GCS_BUCKET`, and add your custom origin to `CORS_ORIGINS` if needed. Deploy the API:

   ```powershell
   gcloud run deploy synthlab-api --project=YOUR_PROJECT_ID --source=backend --region=asia-south1 --service-account=synthlab-api@YOUR_PROJECT_ID.iam.gserviceaccount.com --allow-unauthenticated --env-vars-file=backend/.env.production.yaml
   ```

   Cloud Run must accept public HTTP requests so Firebase Hosting can proxy to it; account and data routes still require the SynthLab session cookie.

6. Deploy the Firestore rules/TTL settings, then Hosting (backend first):

   ```powershell
   firebase deploy --only firestore
   firebase deploy --only hosting
   ```

`backend/.env.production.yaml` is local deployment configuration and must not be committed. `COOKIE_SECURE=true` is required behind HTTPS. Add any custom domain to both Firebase Authentication's authorized domains and `CORS_ORIGINS`.

The API reports `account_storage`, `template_storage`, and `dataset_storage` at `GET /api/health`.

## Dataset limits and synthetic values

Supported types: integer, float, string, name, email, phone, address, city, country, date, boolean, and UUID. Requests can contain 1–100,000 rows and 1–50 columns, up to 1,000,000 generated cells. The dashboard shows a preview; the API preview contains up to 50 rows. Generated emails use the reserved `.test` domain. Phone values use fictional North American 555-0100 through 555-0199 test numbers.

## API overview

Account and data routes use the session cookie unless otherwise noted.

| Method | Route | Description |
| --- | --- | --- |
| `GET` | `/api/health` | Liveness and storage modes. |
| `POST` | `/api/auth/register` | Create an account and sign in. |
| `POST` | `/api/auth/login` | Sign in. |
| `POST` | `/api/auth/firebase/session` | Verify a Firebase ID token and establish the API session. |
| `POST` | `/api/auth/logout` | Revoke the current session. |
| `GET` | `/api/auth/me` | Get the signed-in account. |
| `GET` | `/api/account` | Get profile and workspace activity. |
| `PATCH` | `/api/account/profile` | Update the display name. |
| `POST` | `/api/account/password` | Change the password. |
| `POST` | `/api/datasets/generate` | Generate a dataset. |
| `GET` | `/api/datasets/{dataset_id}` | Read a dataset preview before expiry. |
| `GET` | `/api/datasets/{dataset_id}/download` | Download its CSV before expiry. |
| `GET` | `/api/templates` | List the current account's schemas. |
| `GET` | `/api/templates/{table_name}` | Load one of the current account's schemas. |
| `POST` | `/api/templates` | Save or replace a schema for the current account. |
| `GET` | `/docs` | Interactive OpenAPI documentation. |

Invalid requests return JSON with an `error` and a user-facing `message`. Sign-in and signup are limited to 10 attempts per minute per client IP; other API routes allow 30 requests per minute.

## Run the backend tests

```powershell
cd backend
python -m pip install -r requirements.txt
python -m pytest
```

The tests cover account lifecycle, session cookies, password changes, dataset generation and download, request validation, and privacy between different accounts.

## Docker Compose

Copy the root `.env.example` to `.env`, then from the project root run:

```powershell
docker compose up --build
```

The app is served at `http://localhost:8080` and the API at `http://localhost:8000`. The named `app_data` volume persists the SQLite database across restarts. Rebuild the frontend image after changing `VITE_API_BASE_URL`, since Vite embeds it into the static bundle.

## Production notes

Use HTTPS and set `COOKIE_SECURE=true` in a hosted environment. Firestore is shared across Cloud Run instances for accounts, sessions, schemas, and rate limits. Cloud Storage is private and shared; dataset URLs are API routes that re-check the signed-in account and expiry rather than public object links. Keep local SQLite mode for development.

Password reset email delivery and account deletion are not included in this version.

## 👤 Author

**Yash Sapavadiya** — [GitHub](https://github.com/yash-sapavadiya-07)
