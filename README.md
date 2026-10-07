# SynthLab — Automated Test Data Generator

SynthLab creates realistic, privacy‑safe synthetic datasets for development and QA.  
Sign in to a private workspace, define a table, preview generated rows, and export a CSV.  
The app never reads production data.

---

## 📂 Project Download

Due to the large project size, the full source code is hosted on Google Drive.  
You can download it here:

👉 [Download SynthLab Project (Google Drive)](https://drive.google.com/your-link-here)

---

## 🚀 Features

- Firebase email‑link signup/login (password login also available)
- Secure account/session handling with Argon2 password hashing
- Local dev storage (SQLite, memory) and production storage (Firestore, Cloud Storage)
- Dataset generation with 12 field types, previews, and CSV downloads
- Per‑account access checks for schemas, previews, and downloads
- Responsive dashboard with pointer glow
- Input validation, allowed‑origin checks, and request limits

---

## 🛠️ Technology Stack

- **Frontend:** React + Vite  
- **Backend:** Python, FastAPI, Pydantic, Pandas, NumPy, Faker  
- **Storage:** SQLite (local), Firestore + Cloud Storage (production)  
- **Deployment:** Firebase Hosting + Cloud Run  

---

## ⚡ Local Development Setup (Windows)

### Backend
```powershell
cd backend
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
Copy-Item .env.example .env
uvicorn app.main:app --reload --port 8000
