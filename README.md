# Internship Management Portal

A full-stack internship operations portal for administrators, HR, mentors, and employees. The React client is in `frontend/`; the Express and MongoDB API is in `backend/`.

## Requirements

- Node.js 20.19+ or 22.12+ and npm
- MongoDB 6.0+ locally, or a MongoDB Atlas connection string

## Setup

1. Install root and application dependencies:

   ```powershell
   npm install
   npm --prefix frontend install
   npm --prefix backend install
   ```

2. Create the API environment file:

   ```powershell
   Copy-Item backend/.env.example backend/.env
   ```

3. Set values in `backend/.env`. At minimum, configure `MONGODB_URI`, a unique `JWT_SECRET` with at least 32 characters, administrator credentials, HR credentials, and `SEED_USER_PASSWORD` for the seeded mentor and employee accounts. The example file is for local setup only; replace every example credential before use.

   For a local MongoDB server, the default URI is `mongodb://127.0.0.1:27017/internship_portal`. For Atlas, use the connection string supplied by your cluster and allow the development machine in the cluster network access settings.

4. Seed the database:

   ```powershell
   npm run seed
   ```

5. Start both applications:

   ```powershell
   npm run dev
   ```

   The frontend runs at <http://localhost:5173> and proxies `/api` requests to the API at <http://localhost:5000>. API readiness is reported at <http://localhost:5000/api/health>; it returns HTTP 200 only after MongoDB is connected.

## Environment Variables

| Variable | Purpose |
| --- | --- |
| `PORT` | Express port (default `5000`) |
| `NODE_ENV` | Runtime mode |
| `MONGODB_URI` | MongoDB connection string |
| `JWT_SECRET` | Signing key; use a unique random value of 32+ characters |
| `JWT_EXPIRES_IN` | Token lifetime (default `7d`) |
| `CLIENT_ORIGIN` | Allowed frontend origin(s), comma-separated |
| `ADMIN_NAME` | Name used to seed the initial administrator |
| `ADMIN_EMAIL` | Email used to seed the initial administrator |
| `ADMIN_PASSWORD` | Password used to seed the initial administrator |
| `HR_NAME` | Name used to seed the HR account |
| `HR_EMAIL` | Email used to seed the HR account |
| `HR_PASSWORD` | Password used to seed the HR account |
| `SEED_USER_PASSWORD` | Password shared by the local demo mentor and employee accounts |

## Seed Accounts

The seed script creates one administrator, one HR user, two mentors, five employees, three programs, tasks in active/overdue/submitted/completed states, approved submissions, weighted evaluations, notifications, and an announcement. It is safe to run more than once and does not drop collections. Existing `intern` role values are migrated to `employee`; task and program relationship fields are retained.

- Administrator: the `ADMIN_EMAIL` and `ADMIN_PASSWORD` values in `backend/.env`
- HR: the `HR_EMAIL` and `HR_PASSWORD` values in `backend/.env`
- Mentors: `maya.mentor@example.com`, `jordan.mentor@example.com`
- Employees: `avery.intern@example.com`, `sam.intern@example.com`, `kai.intern@example.com`, `noor.intern@example.com`, `taylor.intern@example.com`
- Mentor and employee password: the `SEED_USER_PASSWORD` value in `backend/.env`

Sign-in accepts any active account; there is no role restriction on login. The account's role determines its workspace after authentication. Public registration can create Employee, HR, Mentor, and Admin accounts. Admin registration requires the server-only `ADMIN_SIGNUP_CODE`; other roles do not. Administrators and HR can manage people and programs.

Separate sign-in entry pages are available at `/sign-in/admin`, `/sign-in/hr`, `/sign-in/mentor`, and `/sign-in/employee`. Each links to the shared `/register` page, where Employee, HR, or Mentor can be selected. These are separate views only; any active account can sign in from any page.

## Run and Build

- `npm run dev` starts the frontend and API together.
- `npm run dev:web` starts only Vite.
- `npm run dev:api` starts the API with Node's built-in watch mode.
- `npm run build` creates the frontend production bundle.
- `npm test` runs the backend domain tests.
- `npm run start:api` starts the API without watch mode.
- `npm run seed` inserts the local sample data.

## Application Structure

```text
frontend/
  src/components/     Shared controls and route protection
  src/context/        Authenticated user session
  src/layouts/        Role-aware portal shell
  src/pages/          Login, dashboards, tasks, programs, people, and reports
  src/services/       Axios client
backend/
  scripts/            Database seed
  src/config/          MongoDB connection
  src/controllers/     API behavior and authorization checks
  src/middleware/      JWT, roles, validation errors
  src/models/          Mongoose schemas
  src/routes/          REST route definitions
```

## Workflow Notes

- Employees can start assigned tasks and submit a description with an optional project URL. They cannot approve their own work or set a task to `Completed`.
- Assigned mentors can request revisions or approve submissions. An evaluation can only be created for an approved submission belonging to one of their assigned employees.
- Evaluation scores are calculated by the API: task quality 40%, timeliness 20%, technical skills 20%, communication and documentation 20%. Each criterion is validated from 0 to 100.
- Overdue tasks are refreshed when dashboards/task lists load and every 15 minutes while the API runs.
- User lists, dashboards, reports, announcements, and notifications are scoped to the authenticated role and account.

## API Overview

- `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me`
- `GET /api/dashboard`
- `/api/users`, `/api/internships`, `/api/tasks`, `/api/evaluations`
- `/api/announcements`, `/api/notifications`
- `GET /api/health`

All routes except login, registration, and health require a JWT bearer token. Role checks and ownership checks are applied in the API; frontend route protection is not treated as an authorization boundary.