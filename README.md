# RuralCare Connect — Healthcare Access Without Reliable Internet

**"Healthcare access, even offline."**  
*ఇంటర్నెట్ లేకున్నా ఆరోగ్య సమాచారం మీతోనే.*

RuralCare Connect is a complete, offline-first Progressive Web App (PWA) built specifically for rural communities and primary health workers who operate in low-connectivity, intermittent, or completely offline environments.

---

## 🌟 Key Features

1. **True Offline-First Progressive Web App**
   - Application shell, bilingual translations, and physician-reviewed medical guides cached locally via Service Worker.
   - Operates with zero network connectivity once loaded.
2. **Persistent Appointment Outbox with IndexedDB**
   - Patient appointment requests are stored persistently on the device using **Dexie.js**.
   - Requests survive browser restarts, reboots, and phone battery drains.
   - Non-misleading status guarantees: clearly distinguishes *"Pending locally (clinic has not received it yet)"* from server-confirmed appointments.
3. **Resilient Synchronization Engine**
   - Concurrency mutex lock prevents duplicate sync loops.
   - Automatic sync triggers on device launch, network reconnection (`online` event), and manual *"Sync Now"* button.
   - Exponential backoff with jitter on temporary failures; stops retrying permanent validation errors.
4. **Strict Server-Side Idempotency & Deduplication**
   - Every appointment request carries an original RFC4122 UUID `requestId`.
   - Re-submitting an identical `requestId` returns the existing record (`isDuplicate: true`) without creating duplicate bookings in the database.
5. **Full Telugu & English Bilingual Localization**
   - Natural Telugu translation across navigation, clinical categories, validation errors, emergency guidance, and appointment statuses.
   - Instant offline language toggle persisted locally.
6. **Physician-Reviewed Health Education Library**
   - Categorized health guides covering safe drinking water, maternal care, child immunization, hand hygiene, medical warning signs, and medication safety.
   - One-click *"Save Offline"* action caches full guides into IndexedDB.
7. **Authorized Healthcare Staff Portal**
   - Role-based clinical dashboard for doctors, nurses, and administrators.
   - Triage requests, confirm visits with scheduled times and clinic instructions, reject with documented reasons, and inspect complete audit trail logs.
8. **Emergency Hotlines & Poison Protocol**
   - Offline-accessible directory for 108 (Ambulance), 104 (Health advice), and 102 (Maternal transport), plus a verified Snakebite first-aid protocol.
9. **Interactive Hackathon Demonstration Simulator**
   - Top demonstration bar to simulate Forced Offline Outage, Weak 2G Signal, Server 503 Errors, Quick Offline Requests, and Idempotency tests.

---

## 🏛️ System Architecture

```mermaid
graph TD
    subgraph Browser ["Client Browser / Mobile PWA"]
        UI["React 18 + Tailwind UI\n(Telugu & English)"]
        SW["Service Worker\n(App Shell & Asset Caching)"]
        DexieDB["IndexedDB (Dexie.js)\n- appointmentOutbox\n- healthResources\n- appointmentCache\n- appMetadata"]
        SyncMgr["Sync Engine\n(Backoff, Mutex, Idempotency)"]
        DemoCtrl["Demo Simulator Bar\n(Simulate Offline / Error Injection)"]
    end

    subgraph Backend ["Backend API (Node.js + Express + TypeScript)"]
        API["REST API\n/api/appointments\n/api/resources\n/api/staff/*"]
        ZodVal["Zod Schema Validation"]
        AuthMid["JWT & RBAC Middleware"]
        Idemp["Idempotent Processor\n(Strict request_id constraint)"]
        DB[(Database: SQLite / PostgreSQL)]
        AuditLog[(Audit Trail Logs)]
    end

    UI --> SW
    UI --> DexieDB
    UI --> SyncMgr
    DemoCtrl --> SyncMgr
    SyncMgr -->|Idempotent POST /api/appointments| API
    SyncMgr -->|GET /api/appointments?requestIds=...| API
    API --> ZodVal
    API --> AuthMid
    API --> Idemp
    Idemp --> DB
    Idemp --> AuditLog
```

---

## 🚀 Quick Start & Installation

### Prerequisites
- Node.js (v20+ or v24+ LTS)
- npm (v10+)

The portable runtime is preconfigured in `tools/nodejs` for zero-install Windows execution.

### 1. Start Backend & Frontend

Open two terminal windows:

**Terminal 1 — Backend API Server (Port 4000):**
```bash
cd backend
npm run dev
# Or node dist/server.js
```
*Backend runs on `http://localhost:4000` with pre-seeded clinics, health resources, and staff accounts.*

**Terminal 2 — Frontend PWA Client (Port 3000):**
```bash
cd frontend
npm run dev
```
*Frontend runs on `http://localhost:3000` (proxies `/api` to 4000).*

---

## 🧪 Automated Testing Suite (Tests A through N)

To execute all 15 automated tests covering unit, integration, idempotency, security, and offline outbox persistence:

```bash
cd tests
npm test
```

### Verified Test Results:
- **Test A:** Application startup and health check responds with 200 OK and connected database (PASSED)
- **Test A (cont):** Seeded clinics are reachable and verified (PASSED)
- **Test B:** Bilingual language switching integrity (English & Telugu dictionaries align 1-to-1) (PASSED)
- **Test C:** Caching application assets and service worker precache manifest (PASSED)
- **Test D:** Downloading a health resource and reading it offline from IndexedDB (PASSED)
- **Test E:** Creating an appointment request offline persists in IndexedDB with `PENDING_SYNC` (PASSED)
- **Test F:** Confirming that a pending request survives a page reload / app restart (PASSED)
- **Test G:** Reconnecting and synchronizing resolves local status to `SUBMITTED` (PASSED)
- **Test H:** Prevents duplicate requests with the same request ID (Strict Idempotency) (PASSED)
- **Test I:** Preserving a queued request when backend is unavailable (`RETRY_PENDING`) (PASSED)
- **Test J:** Recovering after an interrupted request on subsequent sync attempt (PASSED)
- **Test K:** Rejects unauthorized staff API requests without valid credentials (PASSED)
- **Test L:** Prevents unauthorized appointment status changes (PASSED)
- **Test M:** Confirming that failed validation does not create a database record (PASSED)
- **Test N:** Preventing accidental deletion of pending requests from outbox (PASSED)

---

## 🎭 Live Hackathon Demonstration Workflow

Follow these steps for an impressive live presentation:

1. **Launch App in English:**
   - Open `http://localhost:3000/`. Observe the clean healthcare interface with deep blue and teal tokens.
2. **Switch to Telugu:**
   - Click the language button in the navbar (`English` / `తెలుగు`).
   - Notice all navigation, headers, and descriptions instantly switch to natural Telugu.
3. **Explore Health Library & Save Offline:**
   - Navigate to `/health-library`.
   - Click *"డౌన్‌లోడ్ (Save Offline)"* on the Maternal Care guide (*గర్భిణీ స్త్రీల సంరక్షణ*).
   - The badge changes to *"ఆఫ్‌లైన్ సిద్ధం (Offline Ready)"*.
4. **Simulate Network Outage:**
   - In the top **Demo Simulator Bar**, click **"Simulate Offline Outage"**.
   - Notice the top badge turns red: `[Simulation Active: Forced Offline]`.
5. **Read Resource Offline:**
   - Open the downloaded Maternal Care guide. It opens instantly with full text directly from IndexedDB without internet.
6. **Submit an Appointment Offline:**
   - Go to `/appointments/new` (*అపాయింట్‌మెంట్ అభ్యర్థన*).
   - Enter:
     - Name: `Lakshmi Devi`
     - Phone: `9876543210`
     - Clinic: `Rampachodavaram PHC`
     - Date: Tomorrow
     - Reason: `Maternal / Antenatal Care (ANC)`
     - Check consent checkbox.
   - Click **"అభ్యర్థనను సేవ్ చేయండి (Save Appointment Request)"**.
7. **Verify Non-Misleading Local Status:**
   - Notice the amber notification: *"Your request is saved on this device. The clinic has not received it yet."*
   - In My Requests (`/appointments`), status is prominently marked **"పరికరంలో భద్రపరచబడింది (Pending locally)"**.
8. **Reload Page to Demonstrate Persistence:**
   - Press `F5` / Refresh the browser.
   - The appointment request remains safely saved in IndexedDB with zero data loss.
9. **Restore Connectivity & Synchronize:**
   - Click **"Resume Online"** in the Demo Bar.
   - Click **"Sync Now"**.
   - The status transitions in real-time from `PENDING_SYNC` to `SUBMITTED` (*పంపబడింది*).
10. **Triage in Staff Portal:**
    - Open `/staff/login`. Click quick demo login for **Dr. Anita Rao** (`dr_anita`).
    - View incoming request from Lakshmi Devi on the triage board.
    - Click **"Confirm"**, schedule for `Tomorrow at 10:30 AM`, add note: *"Please bring your ANC record booklet."*, and save.
11. **Return to Patient Outbox:**
    - Go back to `/appointments`.
    - Lakshmi Devi's card is now updated with green badge: **"నిర్ధారించబడింది (Confirmed)"**, scheduled time `Tomorrow at 10:30 AM`, and doctor notes!
12. **Test Idempotency Deduplication:**
    - In the Demo Bar, click **"Test Idempotency"**.
    - The request is re-sent to the backend. The backend recognizes the existing UUID and returns HTTP 200 without creating a duplicate record in the database.

---

## 🔐 Staff Demo Credentials

| Role | Username | Password | Assigned Clinic |
|---|---|---|---|
| Medical Officer (Doctor) | `dr_anita` | `ClinicPassword2026!` | Rampachodavaram PHC |
| Staff Nurse | `nurse_ravi` | `ClinicPassword2026!` | Rampachodavaram PHC |
| Health Admin | `admin_rural` | `AdminPassword2026!` | Bhadrachalam CHC |

---

## ⚙️ Environment Variables Template (`.env`)

```ini
PORT=4000
NODE_ENV=development
DATABASE_URL=file:./ruralcare.db
JWT_SECRET=ruralcare_jwt_secure_key_change_in_production_2026
JWT_EXPIRES_IN=24h
VITE_API_BASE_URL=http://localhost:4000/api
```

---

## 🌐 Production Deployment

### Database (PostgreSQL / Supabase)
1. Execute `backend/src/db/schema.sql` in your PostgreSQL or Supabase SQL editor.
2. Set `DATABASE_URL=postgresql://user:password@host:5432/ruralcare`.

### Backend (Render / Railway / Fly.io)
1. Build: `npm run build:backend`
2. Start: `npm run start:backend` (runs `node backend/dist/server.js`)

### Frontend (Vercel / Cloudflare Pages / Netlify)
1. Build: `npm run build:frontend` (runs `vite build`)
2. Output directory: `frontend/dist`
3. Configure rewrite: `/* -> /index.html` for single-page app routing.

---

## 📝 Known Limitations & Future Roadmap
- **SMS / IVR Gateway Integration:** Future versions will integrate with local GSM SMS gateways so patients without smartphones can receive confirmation SMS.
- **Voice Navigation:** Incorporating speech synthesis in Telugu for non-literate community elders.
- **ASHA Worker Sync Role:** Enabling multi-patient tablet synchronization for Accredited Social Health Activists on house visits.
