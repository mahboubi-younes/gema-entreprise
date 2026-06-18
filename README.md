# AssetDesk

AssetDesk is an enterprise-grade service desk and asset inventory platform. It consolidates ticket operations, device health, and field workflows into a unified workspace built for IT organizations.

## Capabilities
- Role-based access with JWT authentication
- Ticket workflows with priority, status, and ownership
- Asset inventory with health states, locations, and assignment
- People directory for service desk and field teams
- Operational dashboard with executive summary metrics

## Technology
- Frontend: React + Vite
- Backend: Node.js + Express
- Database: SQLite (swap-in ready for PostgreSQL)
- Authentication: JWT

## Architecture
```
React UI -> Express API -> SQLite
```

## Screens
![Dashboard](frontend/public/screens/dashboard.svg)
![Tickets](frontend/public/screens/tickets.svg)

## Local Setup

### Backend
```
cd backend
npm install
cp .env.example .env
npm run seed
npm run dev
```

API: `http://localhost:4000`

### Frontend
```
cd frontend
npm install
npm run dev
```

Frontend: `http://localhost:5173`

### Demo Access
- Email: `demo@assetdesk.dev`
- Password: `demo123`

## Notes
- Update `CORS_ORIGIN` in `backend/.env` if the frontend URL changes.
- To use PostgreSQL, replace the SQLite layer in `backend/src/db.js` with a Postgres client.

---

## 🛡️ Security & Architecture Model

In accordance with community security code review, the platform is designed with the following security boundaries:

1.  **SQL Injection Mitigation (100% Parameterized Queries):** 
    All SQLite read/write operations (e.g. users query, tickets insertion, and updates) are constructed using parameterized prepared statements via the SQLite engine (`db.prepare(...)` with placeholder `?`). Raw input string concatenation is never used, completely neutralizing SQL injection vectors.
2.  **API Rate Limiting & Hardening:**
    The login endpoint (`/api/auth/login`) is gated by rate-limiting middleware (`express-rate-limit`) to prevent automated dictionary attacks. The backend uses `helmet` headers for basic security sanitization (CSP, clickjacking prevention, X-Content-Type-Options) and restricts JSON payloads to `200kb`.
3.  **Authentication & JWT Security:**
    In production mode, the server performs a startup validation. If `JWT_SECRET` is missing or set to the default developer string, the process immediately crashes to prevent insecure deployment.

---

## 🌐 Deployment & Persistence Model (SQLite Free-Tier Warning)

By default, this project deploys SQLite on Render's free tier:
*   **Ephemeral Filesystem:** Because free Render instances lack persistent disk volume attachments, the SQLite database (`.db`) is stored in the writable ephemeral container space.
*   **Safety Recycle:** Whenever the dyno goes to sleep due to inactivity or recycles during deployments, the database resets to its default seeded state. For public showcase demos, this acts as a natural security feature, clearing user-submitted spam.
*   **Production Upgrade:** For a production-ready deployment, it is highly recommended to provision PostgreSQL on Render (which is natively supported by swap-in client layers in `db.js`) or use a remote DB provider (like Neon, Turso/LibSQL, or Supabase).

---

## ⚙️ Environment Variables

### Backend Configuration (`/backend/.env`)
| Variable | Description | Default / Example | Required |
|---|---|---|---|
| `PORT` | Port for Express API | `4000` | No |
| `JWT_SECRET` | 256-bit cryptographically secure signature secret | `your_secure_jwt_secret_here` | **Yes (Prod)** |
| `CORS_ORIGIN` | Allowed origin for incoming requests | `https://assetdesk-demo.vercel.app` | Yes |

### Frontend Configuration (`/frontend/.env`)
| Variable | Description | Default / Example | Required |
|---|---|---|---|
| `VITE_API_URL` | Live Render backend endpoint | `https://assetdesk-backend.onrender.com` | No (falls back to local mock storage) |
| `VITE_DEMO_MODE` | Force UI to run in local mock database mode | `true` | No |

