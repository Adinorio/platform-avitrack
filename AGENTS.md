Of course. I will modify the provided `AGENTS.md` file to perfectly suit your new Python/Django repository for the **Avitrack System**.

This version preserves the excellent structure, format, and intent of the original while translating all concepts to a Python, Django, DRF, and React ecosystem. It removes Docker/Supabase specifics in favor of a standard virtual environment and PostgreSQL setup, and is tailored for an AI-assisted development workflow like the one in Cursor.

Here is the complete, modified `AGENTS.md` file for your new repository.

-----

## AGENTS.md – Operating Manual for Autonomous AI Assistants

### 1\. Purpose & Audience

This document is a concise but complete operating manual for autonomous or semi-autonomous AI coding, documentation, and maintenance agents working inside the `avitrack/system` repository. It defines:

  - **Repository mental model**: A decoupled system with a **Python/Django REST Framework** backend, a **React** web frontend, and Python-based utilities.
  - **Canonical workflows**: Implementing features, adding packages, database migrations, creating AI-powered endpoints, updating documentation, and dependency maintenance.
  - **Guardrails**: Best practices for security, secrets management, data boundaries, idempotency, reproducibility, performance, and coding style.
  - **Collaboration protocol**: How multiple concurrent agents and humans should interact effectively.

**Primary audiences**:

1.  **Execution Agents**: Generate or modify code, tests, migrations, and documentation.
2.  **Review / Validation Agents**: Perform static analysis and verify linting, type-checking, tests, and builds.
3.  **Architectural / Refactor Agents**: Implement cross-cutting improvements (performance, modularity) under defined constraints.
4.  **Knowledge / Docs Agents**: Keep documentation and schemas synchronized with the codebase.

All agents **MUST** treat this file as the source of truth when policy conflicts arise. If a required rule is missing, propose an addition rather than inventing ad-hoc behavior.

-----

### 2\. Canonical Capabilities & Hard Boundaries

Approved capability surface (default-allowed unless explicitly restricted):

| Domain                | Allowed Actions                                                                            | Must Also Do                                                                 | Never Do                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **Code (Python/Django)** | Create/modify Django models, views, serializers, templates, and services in Django apps. | Add/update minimal tests & type hints; update relevant docs.               | Mix unrelated refactoring with a feature Pull Request.                   |
| **Code (React)** | Create/modify React components, hooks, and pages in the `frontend/` directory.             | Add/update minimal tests & types; update relevant docs.                      | Introduce breaking public API changes without a `BREAKING` note.         |
| **Database (PostgreSQL)** | Create Django migration files via `manage.py makemigrations`.                              | Update application code that uses the new schema.                            | Directly edit generated migration files manually, except for minor fixes. |
| **AI Endpoints** | Add DRF routes under `api/` that use YOLO models via Celery tasks for image processing.     | Enforce authentication & permissions; validate all incoming data.            | Expose raw model paths or skip input validation.                         |
| **Tooling** | Update configs (`pyproject.toml`, `.ruff.toml`).                                           | Document the rationale for the change in the PR description.                 | Remove caching or security settings silently.                            |
| **Docs** | Update `.md` / `.mdx` files for accuracy and clarity.                                      | Cross-link to related guides and API endpoints.                              | Invent undocumented behavior.                                            |
| **Dependencies** | Add/remove Python dependencies via `pip` and update `requirements.txt`.                    | Keep requirements pinned using a tool like `pip-tools`.                      | Add a duplicate or conflicting dependency version.                       |

**Mandatory guardrails**:

1.  **Least Privilege**: Touch **ONLY** the files required for the requested change.
2.  **Idempotency**: Re-running a workflow yields the same state (migrations are additive, scripts are safe to repeat).
3.  **Determinism**: Generated artifacts (migrations) must result from scripted commands (`makemigrations`), not manual edits.
4.  **Reproducibility**: Provide succinct setup and run steps (`pip install`, `python manage.py runserver`) if a novel workflow is introduced.
5.  **Security**: Never output secret values; reference environment variable names only.
6.  **Observability**: Add logs or comments only where they materially aid in debugging—avoid noisy `print()` statements.
7.  **Explicit User Intent**: Do **NOT** run `python manage.py runserver` or other long-running commands unless the user explicitly requests it.
8.  **User-Only Database Apply**: **NEVER** run `python manage.py migrate`. Prepare migrations and instructions; the user applies them to the database.
9.  **User-Only Linters**: **NEVER** run `black . --check` or `ruff check --fix`. Surface the needed changes and ask the user to run the formatting/linting commands.

-----

### 3\. Repository Structure & Semantics

Top-level layout (partial):

```
avitrack-system/
├── backend/                  # Django project root
│   ├── avitrack_project/      # Core Django project settings, urls.py
│   ├── apps/                 # All Django applications
│   │   ├── users/            # User management, roles, permissions
│   │   ├── species/          # Species data, IUCN status
│   │   ├── sites/            # Site management, census data
│   │   ├── processing/       # Image processing, Celery tasks for YOLO
│   │   └── reports/          # Report generation, analytics
│   ├── manage.py
│   └── requirements.txt
├── frontend/                 # React frontend application
│   ├── public/
│   ├── src/
│   └── package.json
├── scripts/                  # Maintenance & automation scripts (e.g., deployment)
└── AGENTS.md
```

**Semantics & norms**:
| Area | Rule |
|------|------|
| **Code Sharing** | Prefer extracting reusable logic into a new Django app under `apps/` before duplicating it. |
| **Database Models** | Models are the single source of truth for the schema. All changes must go through the ORM and `makemigrations`. |
| **App Isolation** | Avoid importing directly from another app's internal modules; use established service layers or signals if needed. |
| **Environment Config** | The Django backend consumes a single `.env` file at the `backend/` root. |
| **Naming** | Django app names should be plural (e.g., `users`, `sites`). Commit scopes should mirror these names. |
| **Server vs. Client Components** | For the React frontend, default to components without state. Add state management (`useState`, `useReducer`) only when interactivity is required. |

-----

### 4\. Canonical Workflows

Each workflow must be: **minimal**, **idempotent**, and **documented** in the PR description.

#### 4.1 Add / Update a Python Dependency

1.  Activate the virtual environment: `source venv/bin/activate`.
2.  Install the package: `pip install <package-name>`.
3.  Update the requirements file: `pip freeze > requirements.txt`. (Or use `pip-compile` if using `pip-tools`).
4.  Run tests to validate that the new dependency does not break existing code.
5.  Update relevant documentation if the package exposes a new public surface.

#### 4.2 Create a New Django App

1.  Navigate to the `backend/` directory.
2.  Run `python manage.py startapp <app_name> apps/<app_name>`.
3.  Add the new app to `INSTALLED_APPS` in `avitrack_project/settings.py`.
4.  Define models in `apps/<app_name>/models.py`.
5.  Create a minimal `README.md` within the app directory explaining its purpose.
6.  Add at least one test case.

#### 4.3 Add a DRF API Route

1.  Choose the appropriate Django app (e.g., `apps/species/`).
2.  Create or update the `serializers.py` file with a `ModelSerializer`.
3.  Create or update the `views.py` file with a `ViewSet` or `APIView`.
4.  Use DRF's permission classes for authentication and authorization.
5.  Validate input using the serializer; reject invalid data early with a 4xx error.
6.  Add the new route to the app's `urls.py` and include it in the main `avitrack_project/urls.py`.
7.  Add a test for the new endpoint.

#### 4.4 Implement an Image Processing Task

1.  Define a Celery task in `apps/processing/tasks.py`. This task will load the YOLO model and process an image.
2.  [cite\_start]Create a DRF API endpoint that accepts an image upload[cite: 93].
3.  [cite\_start]The endpoint should save the image and trigger the Celery task asynchronously, passing the image's path or ID[cite: 94].
4.  The task saves the results (species, count) to the `Census` or a related model.
5.  [cite\_start]The frontend can poll a status endpoint to check for completion and display the results for review[cite: 98, 99].
6.  [cite\_start]Enforce permissions to ensure only authorized users (`admin`) can initiate processing[cite: 10, 63].

#### 4.5 Database Schema Change

1.  Modify a model in an `apps/<app_name>/models.py` file.
2.  Create a new migration file: `python manage.py makemigrations <app_name>`.
3.  **User-only Application**: The user runs `python manage.py migrate` to apply the changes. The agent **MUST NOT** execute this command.
4.  Update application code (serializers, views) that references the changed model.
5.  If a data backfill is needed (\<30 LOC), create a data migration; otherwise, escalate.
6.  Prepare targeted tests referencing the new schema (the user will execute them).

#### 4.6 Formatting, Linting, Typecheck

Use **Black**, **Ruff**, and **MyPy** (user-run only; the agent must not execute commands directly).

1.  The agent identifies potential lint/format issues and requests the user to run `black .` and `ruff check .`.
2.  For fixes, the agent proposes code edits; the user may optionally run `ruff check --fix`.
3.  The agent re-checks the file content after the user's action to confirm resolution.

-----

### 5\. Coding Standards & Conventions

#### 5.1 Git Hygiene

Follow Conventional Commits.

  - **Format**: `type(scope): description` (e.g., `feat(species): add endpoint for IUCN status`).
  - **Scope**: The name of the affected Django app or frontend area (e.g., `users`, `sites`, `ui`).

#### 5.2 Python

  - Follow **PEP 8**. Use Black for automatic formatting.
  - Use type hints (**PEP 484**) for all function signatures.
  - Follow the "Fat Models, Thin Views" principle in Django.
  - Use Django's ORM efficiently to avoid N+1 query problems.
  - Use Zod (in React) or DRF Serializers (in Django) for runtime validation of all external inputs.

#### 5.3 Error Handling

  - Fail fast: validate inputs in serializers or early in views to return 4xx errors.
  - Wrap external service calls in `try...except` blocks and surface sanitized error messages.
  - Never leak stack traces to the client in production (`DEBUG=False`).

#### 5.4 Accessibility & UI (for React Frontend)

  - Ensure interactive elements are keyboard-accessible.
  - Provide `aria-label` for icon-only buttons.
  - Color choices must respect WCAG AA contrast ratios.

*The following frontend-specific sections from the original document are preserved as they represent best practices that are directly applicable to your React frontend.*

#### 5.5 Data Fetching & React Query

Goal: Minimize client complexity and network chatter while keeping UX responsive.

**Decision Order (Prefer Earlier)**:

1.  **Client-only React Query (`useQuery`/`useMutation`)** for interactive, rapidly changing, or session-local state.
2.  **Realtime subscription** (e.g., via WebSockets) + targeted query invalidation for live updates.

**When TO use React Query**:

  - User-triggered mutations needing optimistic UI or undo.
  - Background refetch for data freshness.
  - Paginated / infinite lists.
  - Dependent queries.
  - Shared client state consumed in multiple sibling components.

**Query Keys**:

  - Always use a stable array form: `['domain', 'subdomain'?, { params }, 'version'?]`.

**Mutations**:

  - Define `useMutation({ mutationFn, onMutate, onError, onSettled })`.
  - Use optimistic updates for a responsive UX, but always include rollback logic in `onError`.
  - Avoid broad invalidations (`invalidateQueries()` with no key); be specific to hurt performance less.

#### 5.6 Toast Notifications

Use a unified toast utility like **Sonner** or **react-hot-toast**.

  - Consolidate all toast notifications to a single, consistent API.
  - Keep toast content concise (≤120 chars).
  - Prefer semantic variants (success, error, info, warning).

#### 5.7 Dialog Components

  - **NEVER** use native browser dialogs (`alert()`, `confirm()`, `prompt()`). They are not accessible and cannot be styled.
  - **ALWAYS** use a proper dialog component from a UI library (e.g., Material-UI, Radix, Shadcn/ui) for all modal interactions.
  - Ensure dialogs are keyboard-accessible and screen-reader friendly with proper ARIA attributes.

-----

### 6\. Environment & Tooling Usage

#### 6.1 Package Manager & Scripts

  - **Backend**: Use **pip** and a virtual environment (`venv`). Dependencies are pinned in `requirements.txt`.
  - **Frontend**: Use **Bun** or **npm/yarn** as defined in `frontend/package.json`.
  - **Key Scripts**:
      - `python manage.py runserver`: Starts the Django development server.
      - `python manage.py test`: Runs the backend test suite.
      - `python manage.py makemigrations`: Creates new database migrations.
      - `celery -A avitrack_project worker -l info`: Starts the Celery worker for background tasks.

#### 6.2 Environment Variables

  - Reference variables by name only (e.g., `os.getenv('DATABASE_URL')`).
  - The backend uses a `.env` file in the `backend/` directory.
  - Public-prefixed variables (`REACT_APP_` or `VITE_`) may appear client-side; all others are server-only.

#### 6.3 Local Development

1.  Set up the Python virtual environment: `python -m venv venv` and `source venv/bin/activate`.
2.  Install backend dependencies: `pip install -r backend/requirements.txt`.
3.  Set up the frontend: `cd frontend && bun install`.
4.  Run the backend: `python backend/manage.py runserver`.
5.  Run the frontend: `cd frontend && bun dev`.

-----

### 7\. Agent Collaboration Protocol

(This section is largely stack-agnostic and is preserved for its value in coordinating AI agents.)

#### 7.1 Roles & Handoffs

  - **Execution Agent**: Produces code + tests + doc updates.
  - **Review Agent**: Validates (lint, typecheck, tests) and annotates discrepancies.
  - **Handoff**: From Execution to Review, provide a summary of changes, affected apps, and commands run.

#### 7.2 Concurrency Rules

  - Only one agent mutates a given file path at a time.
  - Use incremental commits grouped by logical concern (e.g., model, serializer, view, test, docs).

#### 7.3 Conflict Resolution

  - Prefer rebasing feature branches over merging to maintain a linear history.
  - If a migration conflict occurs, delete the local migration file and run `makemigrations` again.

-----

### 8\. Guardrails & Pre-PR Verification

#### 8.1 Mandatory Checklist (Execution Agent)

Tick ALL before requesting review:

1.  Scope is limited to the intended change set. ✅
2.  Backend server runs without errors (user ran `python manage.py runserver`). ✅
3.  Linting is clean (user ran `black .` & `ruff check .`). ✅
4.  Tests were added/updated and are passing (user ran `python manage.py test`). ✅
5.  For DB changes: migration file added; user confirms `python manage.py migrate` runs cleanly. ✅
6.  Docs updated for any public API, schema, or environment variable changes. ✅
7.  No secrets, tokens, or API keys are committed. ✅
8.  All new external inputs are validated (DRF Serializers or equivalent). ✅

-----

### 9\. CI / CD Workflows Overview

(A conceptual outline for your future CI pipelines.)

| Category | Representative Workflow(s) | Purpose | Agent Prep |
|----------|----------------------------|---------|-----------|
| **Code Quality** | `linter-check.yaml` | Run Black, Ruff, MyPy | Run formatters/linters locally |
| **Unit Tests** | `django-tests.yaml` | Run Pytest across Django apps | Run failing tests locally before pushing |
| **DB Integrity** | `check-migrations.yaml` | Ensure migrations are not conflicting | Keep migration history linear |
| **Deployment** | `deploy-backend.yaml` | Deploy Django app to production/staging | Ensure `requirements.txt` is up-to-date |

-----

### 10\. Quick Reference (Cheat Sheet)

| Goal | Command / Action | Notes |
|------|------------------|-------|
| **Activate Env** | `source venv/bin/activate` | Must be done first |
| **Install Deps** | `pip install -r backend/requirements.txt` | After activating venv |
| **Run Dev Server** | `python manage.py runserver` | For the backend |
| **Run Tests** | `python manage.py test <app_name?>` | Can be scoped to an app |
| **Lint/Format** | `black .` and `ruff check .` | User runs these, not the agent |
| **New Migration** | `python manage.py makemigrations <app_name>` | Generates the migration file |
| **Apply Migrations** | `python manage.py migrate` | **USER-ONLY**. Agent never runs this. |
| **Add Dependency** | `pip install <pkg>` then `pip freeze > requirements.txt` | Pin dependencies |
| **Create Django App**| `python manage.py startapp <name> apps/<name>` | Then add to `INSTALLED_APPS` |
| **Add API Route** | Edit `views.py`, `serializers.py`, `urls.py` | Use DRF ViewSets |
| **Start Worker** | `celery -A avitrack_project worker -l info` | For background tasks |
| **(DO NOT run migrate)** | User-only command | Agent prepares migrations & instructions |
| **(DO NOT run linters)** | User-only command | Agent suggests fixes; user executes |

-----

### 11\. Django App Extraction Decision Matrix

When deciding whether to extract shared logic into a new Django app under `apps/*`, evaluate the following. Extract when ≥3 HIGH signals or a SINGLE Critical apply.

| Criterion | Keep In-App (LOW) | Consider Extraction (MED) | Extract Now (HIGH) | Critical (Immediate) |
|-----------|-------------------|---------------------------|--------------------|----------------------|
| **Reuse Breadth** | Used in 1 app | Needed in 2 apps soon | Actively duplicated in ≥2 apps | Security / auth logic duplicated |
| **Complexity** | \<50 LOC simple | 50–150 LOC moderate | \>150 LOC, multiple modules | Requires specialized setup (e.g., service clients) |
| **Domain Ownership** | App-specific logic | Mixed concerns | Pure cross-domain utility | Enforces a core business rule |
| **Drift Risk** | Low | Emerging duplication | Frequent copy-paste edits | A security patch needs a single point of fix |

-----

### 12\. Image Processing Model Usage Policy

#### 12.1 Model Selection

  - The primary model is a versioned **YOLO model** for bird species detection. The exact version and path should be configured via environment variables.
  - [cite\_start]The system is initially limited to classifying 3 specific birds[cite: 90]. The model used must be trained for this task.

#### 12.2 Confidence Thresholds

  - A minimum confidence threshold (e.g., 85%) must be configured. Detections below this threshold should be flagged for manual review rather than being automatically counted.
  - This threshold should be stored as an environment variable to allow for tuning without code changes.

#### 12.3 Error Handling Strategy

| Error Type | Detection | Action |
|-----------|--------|--------|
| **Model Load Failure** | Exception during model loading in Celery task. | Log the error critically; return a failure state to the user. Do not retry. |
| **Image Format Invalid** | Error during image preprocessing (e.g., with OpenCV). | Reject the request with a 400 Bad Request error. |
| **Low Confidence Results** | All detections are below the confidence threshold. | Mark the processing result as "needs review" and alert the admin. |
| **No Detections** | The model runs successfully but finds no objects. | Mark the processing as complete with a count of zero. |

-----

### 13\. Glossary

| Term | Definition |
|------|------------|
| **ORM** | Object-Relational Mapper. Django's primary way of interacting with the database via Python classes (Models). |
| **DRF** | Django REST Framework. The library used to build web APIs. |
| **Serializer** | A DRF class that converts complex data types, like querysets and model instances, to native Python datatypes that can then be easily rendered into JSON. |
| **View/ViewSet** | A Django/DRF class that processes a web request and returns a web response. This is where API logic lives. |
| **Migration** | A versioned Python file generated by `makemigrations` that represents changes to the database schema. |
| **Celery** | A task queue for executing asynchronous (background) tasks, such as image processing. |
| **Virtual Env (venv)** | An isolated Python environment that allows packages to be installed for a specific project, rather than system-wide. |
| **Idempotent** | An operation that is safe to run multiple times without changing the final state beyond the initial application. |
| **React Query** | A client-side caching & async state library for queries/mutations in the React frontend. |
| **Dialog Components** | Accessible modal components from a UI library; must be used instead of native browser dialogs (`alert()`, `confirm()`, `prompt()`). |