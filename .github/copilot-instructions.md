# Copilot instructions for this repository

## Project overview

This repository is a small Python/FastAPI app for Mergington High School. The backend and the browser UI are intentionally lightweight and live in the same codebase:

- `src/app.py` contains the FastAPI app, in-memory database, and all route handlers.
- `src/static/` contains the frontend (`index.html`, `app.js`, `styles.css`) that calls the API.
- `src` is the web app root; the app is served with `uvicorn` and the root path redirects to the static HTML.
- The app stores activity data in memory only. Restarting the server clears all signups.

This project is designed as a learning repo and is intentionally simple; when changing behavior, keep the API contract and frontend data expectations aligned.

## Commands

### Setup

```bash
python -m pip install -r requirements.txt
```

### Run the app

```bash
python -m uvicorn src.app:app --reload
```

The repository also includes a VS Code debug config in `.vscode/launch.json` that launches the same app with `uvicorn src.app:app --reload` and watches `src/static/*`.

After startup, open:

- `http://localhost:8000/` for the site
- `http://localhost:8000/docs` for FastAPI Swagger UI

### Testing

There is no committed test suite in the repo right now, but the project uses pytest as the standard runner for Python tests. Examples:

```bash
pytest
pytest tests/test_app.py
pytest tests/test_app.py -k signup
pytest tests/test_app.py::test_signup_for_activity
```

There is no explicit lint or formatting command configured in this repo (`requirements.txt`, `pytest.ini`, and the devcontainer setup do not define a lint target). Treat `pytest` as the validation runner unless new tooling is added.

## Architecture notes

- The `activities` dictionary in `src/app.py` is the single source of truth for data during a process lifetime.
- Activity names are used as dictionary keys and path parameters (`/activities/{activity_name}`), so keep names stable and human-readable.
- The frontend expects `GET /activities` to return an object keyed by activity name, with each value containing `description`, `schedule`, `max_participants`, and `participants`.
- Signup and removal endpoints validate both the activity and the participant list and return HTTP 400/404 errors using the existing FastAPI pattern.
- Because the static UI directly calls the API, backend contract changes must be mirrored in `src/static/app.js` and related templates when necessary.

## Key conventions

- Keep the FastAPI route behavior consistent with the patterns already used in `src/app.py`: validate the input early, raise `HTTPException` with a status code and message, and return a simple JSON payload for success.
- Preserve the in-memory structure of `activities` and the existing email-based participant model.
- If you add new routes or data fields, prefer the same naming and response shape already used across the app instead of introducing a new object schema.
- The repo is intentionally minimal; prefer surgical edits in the existing files instead of introducing a larger framework or new service layer unless the task explicitly requires it.
- The repository targets a devcontainer/Codespace workflow; keep commands compatible with the Python 3 devcontainer configuration in `.devcontainer/devcontainer.json`.

## Repo-specific expectations for Copilot

- Prefer editing the existing FastAPI app and static frontend files rather than creating a second app structure.
- When modifying the API, check whether the browser logic in `src/static/app.js` depends on the response format.
- Keep code simple and consistent with the small, educational scope of the project.
- When adding tests, place them under a `tests/` directory and run them with pytest so they fit the repo's expected tooling.
