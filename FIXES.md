# FIXES.md — Bug Report for hng14-stage2-devops

## Bug 1 — .env file committed to repository
- **File:** `.env`, `api/.env`
- **Problem:** Files containing secrets (`REDIS_PASSWORD`) were tracked by git, exposing credentials in the repository history.
- **Fix:** Removed from git tracking with `git rm --cached`, added to `.gitignore`.

## Bug 2 — Redis host hardcoded to localhost
- **File:** `api/main.py` line 9, `worker/worker.py` line 6
- **Problem:** `redis.Redis(host="localhost")` works locally but fails inside Docker containers where services communicate by service name, not localhost.
- **Fix:** Changed to read from `REDIS_HOST` environment variable, defaulting to `"redis"` (the Docker Compose service name).

## Bug 3 — Redis password never used
- **File:** `api/main.py` line 9, `worker/worker.py` line 6
- **Problem:** `.env` defines `REDIS_PASSWORD` but neither service passed it to the Redis connection, causing authentication failures when Redis requires a password.
- **Fix:** Added `password=os.getenv("REDIS_PASSWORD", "")` to all Redis connections.

## Bug 4 — Redis connection created at module load
- **File:** `api/main.py` line 9
- **Problem:** Redis client was instantiated at import time, before environment variables were guaranteed to be set. Any startup failure crashed the entire app.
- **Fix:** Moved Redis connection into a `get_redis()` function called per request.

## Bug 5 — Signal handlers imported but never implemented
- **File:** `worker/worker.py`
- **Problem:** `signal` was imported but never used. Docker sends SIGTERM on container stop — without handling it the worker could be killed mid-job.
- **Fix:** Implemented `handle_signal()` for SIGTERM and SIGINT with a clean shutdown loop.

## Bug 6 — API_URL hardcoded to localhost in frontend
- **File:** `frontend/app.js` line 6
- **Problem:** `const API_URL = "http://localhost:8000"` fails inside Docker since localhost refers to the frontend container itself, not the API container.
- **Fix:** Changed to `process.env.API_URL || "http://api:8000"` to read from environment variable.

## Bug 7 — Missing job returns HTTP 200 instead of 404
- **File:** `api/main.py` lines 20-23
- **Problem:** When a job ID is not found, the API returned `{"error": "not found"}` with HTTP status 200, which is incorrect and breaks integration tests.
- **Fix:** Replaced with `raise HTTPException(status_code=404, detail="Job not found")`.

## Bug 8 — Unused import `os` 
- **File:** `api/main.py` line 3
- **Problem:** `os` was imported but never used, causing flake8 lint failure.
- **Fix:** `os` is now used via `os.getenv()` in the `get_redis()` function.

## Bug 9 — No package-lock.json for frontend
- **File:** `frontend/`
- **Problem:** Without a lockfile, `npm install` in Docker could pull different dependency versions on each build, making builds non-reproducible.
- **Fix:** Generated `package-lock.json` by running `npm install` locally and committed it.

## Bug 10 — Unpinned dependency versions
- **File:** `api/requirements.txt`, `worker/requirements.txt`
- **Problem:** Dependencies had no pinned versions, allowing different versions to be installed across environments.
- **Fix:** Pinned all versions explicitly (e.g. `fastapi==0.104.1`, `redis==5.0.1`).
