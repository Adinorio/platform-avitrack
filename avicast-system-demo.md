Of course. Here is the provided text formatted as a Markdown file.

```markdown
# AVITRACK Website Rebuild: Project Plan & MVP

This document outlines a concrete, practical plan to rebuild the AVITRACK website, making it production-ready and acceptable as a Minimum Viable Product (MVP) for CENRO. It includes a step-by-step project plan, a clear MVP scope, and detailed user stories with acceptance criteria based on the provided PDF.

---

## 1. High-level, step-by-step engineering plan (what to do, in order)

Follow these phases in order. Fixed time estimates are not provided, only the logical sequence for implementation.

### Phase A — Project setup & governance
- Create a single repository (e.g., `avitrack-web`) with `main` (protected), `dev`, and feature branches (`feature/xxx`). Enforce Pull Request (PR) reviews.
- Add repository standards: `README.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, `LICENSE`. Add a `docs/` folder for design decisions and API contracts.
- Add templates: issue template, PR template, changelog template. Use Conventional Commits as the required commit style.
- Choose stack & infrastructure (see section 2) and create skeleton applications (frontend + API). Make a minimal “hello-world” end-to-end deployment to validate the infrastructure.

### Phase B — Architecture & design
- Create a short Architecture Decision Record (ADR) list describing decisions: auth, DB, image pipeline (deferred), forecasting (deferred). Store these in `docs/adr/`.
- Design the data model & major endpoints (users, roles, species, sites, counts, reports, image-jobs). A suggested schema is provided later.

### Phase C — Core implementation (MVP first)
- Implement authentication & Role-Based Access Control (RBAC) for Superadmin, Admin, and Field Worker roles, including default superadmin credentials and a first-login flow (force password change or skip). (From PDF: default Superadmin ID `010101` and password `Avitrack123`.)
- Implement User Management UI & APIs (Superadmin-only): create/edit/archive/disable/delete, role assignment, and an activity log card.
- Implement Species Management (CRUD) and Site Management (CRUD), plus manual census counts and an import endpoint for mobile data (field worker requests to admin).
- Implement a Dashboard (tiles linking to features) and a Report Generator that aggregates species, sites, counts, personnel lists, and allows exports (CSV/PDF).

### Phase D — QA, docs, and handover
- Write end-user documentation and an admin manual for CENRO (how to add users, approve counts, generate reports).
- Run a security checklist for a government deliverable: TLS, password policies, role separation, audit logs, data export.
- Conduct acceptance testing with real user scenarios involving CENRO staff.

### Phase E — Post-MVP features (defer until core is stable)
- Integrate the Image Processing (bird identification) pipeline only after manual flows are stable. Implement it as asynchronous jobs with a review/approve/allocate UI as described in the PDF.
- Integrate Forecasting / Weather aggregation (use multiple APIs and present best-day suggestions). Keep forecasting limited to site surroundings as described.

---

## 2. Recommended technical stack & infra (practical & easy to maintain)

- **Frontend**: React (Vite) or Next.js (if server-side pages are needed). Component library: Tailwind CSS or Material UI.
- **Backend**: Node.js + Express or NestJS (good for modularization), or Python Flask/FastAPI if preferred.
- **Auth**: JWT sessions + refresh tokens; initially store credentials in the DB (hashed with bcrypt). Include a forced password change flag for the default superadmin.
- **DB**: PostgreSQL (relational tables make counts & reports easier).
- **Storage**: S3-compatible storage for images and uploads.
- **Jobs**: Redis + Bull/Sidekiq style queue for image-processing jobs & exports.
- **CI / CD**: GitHub Actions to run linting/tests and deploy to staging; use a protected `main` branch.
- **Hosting**: Vercel/Netlify for the frontend (or static), Heroku / Render / DigitalOcean App Platform or a cloud VM for the API + DB, or a cloud-managed DB (Cloud SQL / RDS).
- **Monitoring**: Sentry + basic logs + an audit table for actions.

---

## 3. Policy & repo standards (Conventional Commits + branching)

**Conventional commit examples:**
```

feat(auth): add default superadmin and forced-password-change flag
fix(species): validate scientificName uniqueness
chore(ci): add lint and test workflows
docs: add README and deployment notes

```

**Branch strategy**: Use branches like `feature/*`, `hotfix/*`, `release/*`. Merge to `dev` first, then create a PR to `main` with a review and passing CI checks.

---

## 4. Acceptance checklist to make it viable for a government beneficiary (CENRO)

When delivering the MVP to CENRO, ensure:
- **Role separation** & admin-only user management UI (superadmin role with default credential behavior).
- **Data export** (CSV/PDF) and printable reports for audits.
- **Audit logs**: who did what (login, add/edit/delete species, approve counts).
- **Secure hosting** (HTTPS/TLS) and strong password rules.
- A **simple admin manual**, a training session, and a contact for bug fixes.
- An option to run an **offline/mobile CSV import flow** for field data (so field workers can upload later and admins can verify).

---

## 5. MVP definition (what MUST be done now)

Make these items the MVP. Everything else — especially image processing and weather forecasting — is explicitly post-MVP.

### Core MVP features
- **Authentication & RBAC**:
  - Default superadmin (`010101` / `Avicast123`) on first run with a UI to change the password (option to “Not now,” which shows the prompt only once unless the DB is reset).
  - Roles: Superadmin, Admin, Field Worker with permissions as described in the PDF.
- **User Management (Superadmin-only)**:
  - Create / Edit / Disable / Archive / Delete users; search by ID or name; view daily active users and latest created users; authorization activity card.
- **Species Management**:
  - CRUD for bird species (common + scientific name); can attach an image; admin-only archive/delete.
- **Site Management & Bird Census (manual)**:
  - CRUD for sites (ID, name, location, optional image).
  - Manual counts per site/year/month. Field workers can request additions/edits (not directly change the DB) — requests are queued for admin approval.
- **Report Generator**:
  - Aggregate species/site counts, personnel summary, and exports (CSV/PDF).
- **Import Endpoint**:
  - Allow mobile CSV import (field worker data) with a “request for admin acceptance” workflow.
- **UI: Dashboard**:
  - Dashboard tiles linking to features; recent authorization events; counts overview.

### Not in MVP (explicitly deferred)
- **Image Processing / Bird Identification** (limited to three species per spec) — build later as an asynchronous job pipeline with a review/allocate UI.
- **Forecasting / Weather Aggregation** — defer until site/species workflows and reporting are stable.

---

## 6. Detailed user stories (MVP) — grouped by role

Each story includes acceptance criteria (AC) that reflect the PDF descriptions.

### Role: Superadmin
- **US-SA-01: First login**
  - *As Superadmin, I can log in with default credentials `010101` / `Avitrack123`. After logging in, the system shows a popup allowing me to change my password; "Not now" cancels and the popup never reappears unless the DB is reset.*
  - **AC**: Login succeeds; popup is shown exactly once on first login; the password change option updates the database.
- **US-SA-02: Manage users**
  - *As Superadmin, I can add/edit/archive/disable/delete users and set their roles.*
  - **AC**: A new user gets an incremented user ID; search by ID/name works; archived users are flagged; the UI shows success/error messages (e.g., duplicate entry).
- **US-SA-03: Role/assignment dashboard**
  - *As Superadmin, I can view counts of Admins and Field Workers, and see recent authentication activities.*

### Role: Admin
- **US-A-01: Dashboard access**
  - *As an Admin, I land on the main dashboard and can access Species, Site, Census, and Reports.*
- **US-A-02: Species management**
  - *As an Admin, I can add species with a name, scientific name, and an optional photo; I can edit counts manually and archive species.*
  - **AC**: The species list shows the new species; counts are stored and shown in reports.
- **US-A-03: Site management & approve imports**
  - *As an Admin, I can add/edit sites; I can accept/reject import requests from field workers or mobile app uploads.*
- **US-A-04: View & generate reports**
  - *As an Admin, I can generate and export reports (per site, date range) with species counts and personnel listings.*

### Role: Field Worker
- **US-FW-01: Read-only dashboard**
  - *As a Field Worker, I can view species and site info and see counts but cannot directly modify them; I can submit requests to add/update counts which go to an admin for verification.*
- **US-FW-02: Upload data (mobile/import)**
  - *As a Field Worker, I can upload a CSV or mobile-recorded data which becomes an import request for an Admin to accept/reject.*

---

## 7. Image processing & forecasting — detailed deferred plan (so you can add later)

The PDF describes a 3-step image processing workflow: Upload → Process → Review → Allocate. Implement this after the MVP.

- **Architecture**: A separate microservice or worker for image jobs. Jobs are stored in an `image_jobs` table with statuses: `uploaded`, `processed`, `reviewed`, `allocated`.
- **Frontend flow**:
  1.  **Upload view**: Select file/folder → show upload progress → store assets. Users must trigger processing; do not auto-process on upload.
  2.  **Process view**: Generate results but wait for user action.
  3.  **Review view**: Approve / reject / override results (with filters for approved/rejected/overridden).
  4.  **Allocate view**: Drag & drop processed results into a target site/year/month (left results column, right allocation panel).
- **Note**: Implement this as asynchronous batch jobs with a preview and manual approval before counts are committed to the census.
- **Forecasting**: Aggregate multiple weather APIs and compute a “best day” score per site (based on tides, rainfall, wind). Keep it read-only in the MVP until validated.

---

## 8. DB / API suggestions (concise)

### Suggested core tables:
- `users` (id, user_id_str, first_name, last_name, email, role, password_hash, disabled, archived, must_change_password)
- `roles` (name, permissions)
- `species` (id, common_name, scientific_name, image_url, archived)
- `sites` (id, site_code, name, location_geo, image_url)
- `counts` (id, site_id, species_id, year, month, count, source (manual/import/ai), status)
- `import_requests` (id, uploaded_by, file_url, parsed_counts, status)
- `image_jobs` (id, uploader, file_url, status, results)
- `audit_logs` (id, user_id, action, target, timestamp)

### Suggested API endpoints:
- `POST /auth/login`, `POST /auth/change-password`
- `GET/POST/PUT/DELETE /users` (restricted)
- `GET/POST/PUT/DELETE /species`
- `GET/POST/PUT/DELETE /sites`
- `POST /imports`, `GET /imports/:id/approve`
- `GET /reports?site=&from=&to=` → CSV/PDF

---

## 9. QA & acceptance criteria (general)

- **Automated tests**: Unit tests for business logic (auth, role checks), integration tests for APIs.
- **Manual acceptance**: Run test cases covering role-based access, import/approve flows, and report exports.
- **Security**: Verify password storage, TLS, audit log integrity, and the principle of least privilege.
- **Documentation**: A short admin manual, a quickstart `README` for developers, and a changelog.

---

## 10. Deliverables to give CENRO (final handover bundle)

- Deployed staging and production URLs (with TLS).
- An admin manual (how to create users, accept imports, generate reports).
- An exported data sample (CSV/PDF).
- The source repository with passing CI and deployment instructions.
- A basic training session & support window (including a list of known limitations — e.g., image processing not yet integrated).

---

## 11. Quick checklist you can copy into issues / sprint board

- [ ] Repo + docs + CI skeleton (`README`, `CONTRIBUTING`, commitlint).
- [ ] Auth + default superadmin + forced-change popup.
- [ ] User management (create/edit/archive/disable).
- [ ] Species management CRUD.
- [ ] Site management CRUD + import request flow.
- [ ] Manual census counts & admin approval.
- [ ] Report generator (CSV/PDF).
- [ ] Post-MVP: Image pipeline (Upload → Process → Review → Allocate).
- [ ] Post-MVP: Forecasting/weather aggregation.
```