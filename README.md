# Hospital Management System 🏥

A web-based hospital management system covering patient registration, doctor
scheduling, appointment booking, medical records, prescriptions, pharmacy
inventory, equipment tracking, and operational dashboards.

Built across two iterative phases for the **Software Architecture** course.

- **Phase 1 (delivered 4 May 2026):** architecture design + initial implementation
- **Phase 2 (delivered 21 May 2026):** development & deployment views + production deployment
  
The architecture follows **Kruchten's 4+1 view model** and is realized in the
**Model–View–Controller (MVC) + REST API** style: React 18 SPA frontend
communicating with a Django 5 + Django REST Framework backend.

---

## How to Run

A fresh clone to a working login screen with `admin / Pass1234!`. The
setup script for your OS installs deps, runs migrations, seeds all
demo accounts (see [Demo accounts](#demo-accounts)), starts both
servers, and opens the browser.

You need **Python 3.10–3.13** (3.12 recommended; **3.14 may fail on some
dependencies** — see the Troubleshooting section below) and
**Node.js 20+**.

### macOS

Open Terminal in the folder where you want the repo and paste these
four commands one at a time:

```bash
git clone https://github.com/alrawii2/hospital-managemen-systems-PHASE-2.git
```

```bash
cd hospital-management-system-
```

```bash
chmod +x setup.sh
```

```bash
bash setup.sh
```

When the browser opens at <http://localhost:5173>, log in as
`admin` / `Pass1234!`.

If you get `python3: command not found`, install Apple's command-line
tools first and re-run `bash setup.sh`:

```bash
xcode-select --install
```

> **Linux:** the same commands work identically.

### Windows

Open Command Prompt or PowerShell in the folder where you want the
repo and paste these three commands one at a time:

```bat
git clone https://github.com/alrawii2/hospital-managemen-systems-PHASE-2.git
```

```bat
cd hospital-management-system-
```

```bat
.\setup.bat
```

When the browser opens at <http://localhost:5173>, log in as
`admin` / `Pass1234!`.

---

## Demo accounts

`setup.bat` / `setup.sh` runs `python manage.py seed_data` as part of
the install, so every account below exists in the database
immediately after the script finishes.

**All accounts use the same password: `Pass1234!`**

| Role        | Usernames                                                                                                                            |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------|
| Admin       | `admin`                                                                                                                              |
| Pharmacist  | `pharmacist1`, `pharmacist2`                                                                                                          |
| Management  | `manager1`, `manager2`                                                                                                                |
| Doctor      | `doctor_cardio`, `doctor_cardio2`, `doctor_pediatric`, `doctor_pediatric2`, `doctor_neuro`, `doctor_general`, `doctor_general2`, `doctor_ortho`, `doctor_ortho2` |
| Nurse       | `nurse_cardio`, `nurse_pediatric`, `nurse_neuro`, `nurse_general`, `nurse_ortho`, `nurse_oncall`                                       |
| Patient     | `patient1` … `patient15`                                                                                                              |

The exact list is defined in `accounts/management/commands/seed_data.py`
(`USERS`) — that file is the source of truth.

The older minimal seed (`seed.py`) additionally creates `doctor1`–`doctor5`
with the legacy password `test123`. They are kept for backwards
compatibility but are not the recommended demo accounts.

---

## Troubleshooting

**Login is rejected.** Open browser DevTools → Network tab → click the
`/api/login/` request and read the response:

- **401 Unauthorized** → the username or password is wrong. Use one
  from [Demo accounts](#demo-accounts) above with `Pass1234!`.
- **(failed) / red** → the backend isn't running. Check the Django
  terminal window is alive and reachable at <http://localhost:8000/api/health/>.
- **429 Too Many Requests** → DRF's anonymous rate-limit has kicked in
  after too many failed attempts. Wait one minute, then try again.
- **Anything else** → the login UI displays the HTTP status code in
  the error message — share that with whoever is helping you debug.

**`Pass1234!` is still rejected after running setup.** From the repo
root, re-run the seed (idempotent — uses `update_or_create`):

```bash
python manage.py seed_data
```

**`pip install` fails with `pg_config executable not found` or
`Failed to build psycopg2-binary`.** You're on a Python version (often
3.14) that doesn't have a prebuilt `psycopg2-binary` wheel, so pip
tries to build it from source — which needs PostgreSQL development
tools.

The local setup script does NOT install `psycopg2-binary` anymore
(it's in `requirements-prod.txt`, which is only for the Docker /
PostgreSQL path). If you're still hitting this error, you probably
pulled an older clone — re-pull the latest `main`, delete the local
`venv/` folder, and re-run the setup script.

If you have a real reason to install `requirements-prod.txt` locally
and pip can't find a wheel for your Python version, the simplest fix
is to install Python 3.12:

```bash
brew install python@3.12
```

On Windows, download the 3.12 installer from
<https://www.python.org/downloads/release/python-3120/>.

**macOS: `python3: command not found`.** Install Apple's command-line
tools:

```bash
xcode-select --install
```

**macOS / Linux: `setup.sh: bad interpreter`.** A Windows clone may have
rewritten the shell script with CRLF line endings. Re-clone the repo
(the committed `.gitattributes` enforces LF for `*.sh`), or run
`dos2unix setup.sh` if available.

**Windows: PowerShell blocks `Activate.ps1`.** Run once:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

---

## Scope

- **6 roles:** Patient, Doctor, Nurse, Pharmacist, Management, Admin
- **15 domain entities:** User, Patient, Doctor, Nurse, Pharmacist, Manager,
  Department, Appointment, MedicalRecord, Prescription, Drug, DrugStock,
  Equipment, DoctorAvailability, Room
- **18 use cases** covering registration, login, booking, cancellation,
  doctor availability, prescribing, dispensing, inventory, equipment
  status, and operational KPIs
- **Centralized RBAC** matrix in `accounts/permissions.py`
- **Field-level encryption** (Fernet AES-128) for `User.phone` and `Patient.address`
- **React SPA** frontend (Vite) connected to a **Django REST Framework** backend
- **Token-based authentication** (`rest_framework.authtoken`)
- **Docker-based production deployment** (Nginx + Gunicorn + PostgreSQL + Redis) —
  see the "Production deployment (advanced)" section at the bottom.

---

## Architecture at a glance

**MVC + REST API layers:**

| Layer            | Where it lives                                        |
|------------------|-------------------------------------------------------|
| View (Frontend)  | React SPA — `Frontend/hospital-frontend/src/App.jsx`  |
| API Bridge       | DRF — `accounts/api_views.py`, `accounts/serializers.py`|
| Controller       | `accounts/views.py` — request handlers per use case   |
| Model            | `accounts/models.py` — 15 ORM entities                |

**Cross-cutting:**

- `accounts/permissions.py` — RBAC matrix (single source of truth)
- `accounts/forms.py` — input validation
- `accounts/admin.py` — Django admin config
- `hospital_project/urls.py` — URL routing
- `hospital_project/settings.py` — env-driven configuration

**4+1 views — coverage per phase:**

| View         | Section in SAD v2 | Phase 1 | Phase 2  |
|--------------|-------------------|---------|----------|
| Use Case     | §2                | ✅      | ✅       |
| Logical      | §3                | ✅      | ✅       |
| Process      | §4                | ✅      | ✅       |
| Development  | §7 (new)          | —       | ✅       |
| Deployment   | §8 (expanded)     | stub    | ✅       |

---

## Repository layout

```
.
├── README.md
├── SAD.docx
├── SAD_v2.docx
├── setup.sh / setup.bat
├── seed.py
├── requirements.txt
├── manage.py
├── hospital_project/
├── accounts/
│   ├── models.py
│   ├── api_views.py
│   ├── serializers.py
│   ├── permissions.py
│   ├── views.py
│   ├── forms.py
│   ├── admin.py
│   └── management/commands/seed_data.py
├── Frontend/hospital-frontend/
│   └── src/
│       ├── App.jsx
│       └── App.css
├── deployment/
│   ├── README.md
│   ├── Dockerfile.backend
│   ├── Dockerfile.frontend
│   ├── docker-compose.yml
│   ├── nginx.conf
│   ├── entrypoint.sh
│   ├── .env.example
│   └── .dockerignore
├── use_case_diagram.puml
├── class_diagram.puml
├── sequence_diagram.puml
├── sequence_diagram_v2.puml
├── activity_diagram.puml
├── sequence_prescription_workflow.puml
├── component_diagram.puml
├── deployment_diagram.puml
└── .github/workflows/ci.yml
```

The diagram sources live at the project root as `*.puml` and their
rendered `*.png` siblings.

---

## Tech stack

- Python 3.12 + Django 5.2 + Django REST Framework 3.15
- React 18 + Vite 5
- PostgreSQL 15 (prod) · SQLite 3 (dev fallback)
- Redis 7 (cache + DRF throttling)
- Nginx 1.25 (reverse proxy + static)
- Gunicorn 21 (WSGI app server)
- Docker + Docker Compose for deployment
- Token-based authentication (`rest_framework.authtoken`)
- Field-level encryption via `django-encrypted-model-fields` (Fernet AES-128)

---

## Team and Contributions

| Name                                | Student ID  | Role                            |
|-------------------------------------|-------------|---------------------------------|
| Marwan Mohammed Taher Alkhatib      | 220911700   | Backend / Domain Engineer       |
| Anas Ravioglu                       | 2309015858  | Frontend / Integration Engineer |
| Saleem Yahya Ahmad Almadfaie        | 210911095   | Documentation Engineer          |

Per-phase contributions are documented in the SAD (§"Team and Contributions").

---

## Course context

- **Course:** Software Architecture
- **Phase 1 submission:** 4 May 2026
- **Phase 2 submission:** 21 May 2026

---

## Production deployment (advanced)

> **⚠️ Do NOT use this to try the project for the first time.**
> Use the setup script in [How to Run](#how-to-run) above.

Advanced Docker-based deployment (Nginx + Gunicorn + PostgreSQL +
Redis) and bare-metal "manual run" instructions live in a dedicated
guide: [`deployment/README.md`](deployment/README.md).
