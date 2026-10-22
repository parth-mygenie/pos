# MyGenie POS — Deployment PRD

## Source
- **Repo**: https://github.com/Abhi-mygenie/core-pos-front-end-.git
- **Branch**: main
- **Deployed**: 2026-10-22

## Architecture
- **Frontend**: React 19 + CRACO 7.1.0, served via supervisor on port 3000 (`/app/frontend/`)
- **Backend**: Placeholder FastAPI at `/app/backend/` on port 8001 (real backend is `https://preprod.mygenie.online/`)
- **MongoDB**: Local on 27017 (unused by this frontend — it calls the external preprod API)
- **Socket**: `https://presocket.mygenie.online`

## What Was Deployed (2026-10-22)
- Cloned `main` branch from `https://github.com/Abhi-mygenie/core-pos-front-end-.git` into staging
- Synced repo's `frontend/` → `/app/frontend/` (rsync, excluding node_modules + .env)
- Synced repo's `memory/` → `/app/memory/` (1418 files — full remote memory sync)
- Synced `test_reports/`, `training-backend/`, `training-sdk/` from repo
- Wrote `/app/frontend/.env` with all provided env vars
- Ran `yarn install --ignore-engines` (58s, success)
- Restarted supervisor frontend — `webpack compiled with 1 warning` (non-blocking ESLint)
- App live at HTTP 200, title "MyGenie POS"

## Environment Variables (/app/frontend/.env)
| Key | Value |
|-----|-------|
| REACT_APP_BACKEND_URL | https://core-pos-frontend-13.preview.emergentagent.com |
| WDS_SOCKET_PORT | 443 |
| ENABLE_HEALTH_CHECK | false |
| REACT_APP_API_BASE_URL | https://preprod.mygenie.online/ |
| REACT_APP_SOCKET_URL | https://presocket.mygenie.online |
| REACT_APP_FIREBASE_* | Full Firebase config set |
| REACT_APP_CRM_BASE_URL | https://crm.mygenie.online/api |
| REACT_APP_CRM_API_KEYS | JSON with 3 known keys (364, 475, 478) — PARTIAL (27 keys in full set, rest truncated in problem statement) |
| REACT_APP_GOOGLE_MAPS_KEY | AIzaSyCS9rZcttTxbair3abltZ3Fm1vEnmY0mj4 |
| REACT_APP_SHOW_AUDIT_TAB | true |
| REACT_APP_TRAINING_ENABLED | false |
| CORS_ORIGINS | * |

## Known Issues / Backlog
- **REACT_APP_CRM_API_KEYS incomplete**: Problem statement truncated at "509". Full 27-key JSON needed. Only 3 keys written — CRM lookup will fail for restaurants not in keys 364/475/478.
- One non-blocking ESLint warning at `src/pages/LoadingPage.jsx` (react-hooks/exhaustive-deps)

## Test Credentials
See `/app/memory/control/test_credentials.md` and `/app/memory/test_credentials.md`
- Preprod API: `https://preprod.mygenie.online`
- Owner: `owner@18march.com` / `Qplazm@10`
- Manager (Hogwarts): `Manager@hogwarts.com` / `Qplazm@10`
