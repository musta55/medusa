# Medusa — Student Setup Guide

This is a **course fork of [medusajs/medusa](https://github.com/medusajs/medusa)** — the open-source
commerce **framework monorepo** (30+ TypeScript packages). It is pinned to a verified baseline:

- **Version:** `@medusajs/medusa` **2.17.2**
- **Baseline commit:** `8f97e3f371`
- Everyone starts from this exact commit. Do **not** pull from upstream `develop`/`main` — it moves and can break.

> This repo is the framework itself, **not a ready-to-run store**. See [Running things](#running-things).

---

## 1. Prerequisites

| Tool | Version | Notes |
| --- | --- | --- |
| **Node.js** | **20 LTS or 22 LTS** | **Do not use Node 24** — a few suites fail on it (see [Known failing tests](#known-pre-existing-test-failures)). Use [nvm](https://github.com/nvm-sh/nvm). |
| **Corepack** | ships with Node | Run `corepack enable` — it activates the repo-pinned **Yarn 3.2.1** automatically. Do not install Yarn globally. |
| **Docker** | any recent | Only needed for **integration tests** (and for running a real store DB). |
| **Git** | any | |

**OS:** Linux, macOS, or **Windows via WSL2** recommended. Native Windows works for install/build, but a handful of tests fail on `\` vs `/` path separators (documented below).

---

## 2. First-time setup

```bash
git clone <your-team-fork-url>
cd medusa
corepack enable          # activates Yarn 3.2.1 from the repo
yarn install             # ~1-2 min
yarn build               # ~4 min — builds all 80 packages; REQUIRED before running tests
```

`yarn install` prints ~19 warnings (native-module "must be built" notices + one TS compat-patch
line) — these are **benign**. `yarn build` should end with `80 successful, 80 total`.

---

## 3. Running things

There is **no single "start the app"** in this repo — it's a framework. Your options:

```bash
# Admin dashboard dev server (Vite). Renders the admin UI; API calls need a running
# Medusa backend, so it's limited on its own:
yarn workspace @medusajs/dashboard dev

# Design-system component explorer (Storybook, http://localhost:6006) — fully standalone,
# the easiest way to play with the UI without a backend:
yarn workspace @medusajs/ui storybook
```

**To run an actual store + admin with seed data** (a *separate* project that consumes the framework),
scaffold one with the official CLI — needs a Postgres (see step 5 for a Docker one):

```bash
npx create-medusa-app@latest my-store \
  --db-url "postgres://postgres:postgres@localhost:5433/my_store"
# then: cd my-store && yarn dev   → admin at http://localhost:9000/app
```

---

## 4. Testing — backend (Jest) & frontend (Vitest)

```bash
# FULL unit baseline across all packages. KEEP --continue:
# plain `yarn test` aborts on the first failing package and won't run the rest.
yarn turbo run test --no-daemon --no-cache --force --continue

# A single package:
yarn workspace @medusajs/order test        # backend (Jest)
yarn workspace @medusajs/dashboard test    # frontend (Vitest)
yarn workspace @medusajs/ui test           # frontend (Vitest)

# A single file:
yarn jest <path/to/file.spec.ts>
```

---

## 5. Testing — integration (needs Docker)

Start a Postgres (and Redis, only for the `*-redis` modules) on non-default host ports:

```bash
docker run -d --name medusa-pg -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -p 5433:5432 postgres:16-alpine
docker run -d --name medusa-redis -p 6380:6379 redis:7-alpine
```

Run integration tests with the DB env vars (the runner creates throwaway temp DBs):

```bash
# Git Bash / Linux / macOS:
DB_HOST=localhost DB_PORT=5433 DB_USERNAME=postgres DB_PASSWORD=postgres \
  yarn workspace @medusajs/region test:integration     # one module (verified: 18/18)

# Whole suites (long; `modules` also uses Redis):
DB_HOST=localhost DB_PORT=5433 DB_USERNAME=postgres DB_PASSWORD=postgres yarn test:integration:modules
DB_HOST=localhost DB_PORT=5433 DB_USERNAME=postgres DB_PASSWORD=postgres yarn test:integration:http
```

```powershell
# PowerShell: set vars first, then run
$env:DB_HOST="localhost"; $env:DB_PORT="5433"; $env:DB_USERNAME="postgres"; $env:DB_PASSWORD="postgres"
yarn workspace @medusajs/region test:integration
```

Teardown: `docker rm -f medusa-pg medusa-redis`

---

## 6. Known pre-existing test failures (do NOT chase these)

On the baseline commit the full unit suite is **62/70 packages green**. The 8 failures are
**environmental, not framework bugs** — every commerce module + workflow suite passes. Full analysis
is in `../project-medusa-checklist.md`. Summary:

| Failing packages | Cause | Notes |
| --- | --- | --- |
| `icons`, `ui` | React/jsx-runtime mismatch in the Vitest env | `icons` fails 100% of icon render specs (one systemic issue), `ui` only its 2 icon-dependent specs. |
| `framework`, `utils`, `eslint-plugin`, `http-types-generator` | Windows `\` vs `/` path separators | Pass on Linux/macOS/WSL2. |
| `create-medusa-app`, `medusa` | Node 24 `os.release` / CLI mocks | Pass on Node 20/22. |

On the recommended env (**Node 20/22 + Linux/macOS/WSL2**), most of these clear. If a test fails,
check it isn't one of the above before debugging.

---

## 7. Doing your work

```bash
git checkout -b feature/<your-feature>
# ...make changes, add tests...
yarn workspace @medusajs/<pkg> test
git commit -am "feat: <what you did>"
git push origin feature/<your-feature>     # open a PR against the course fork
```

Good starter areas (per the course backlog):
- New **commerce module** features (pricing rules, inventory logic).
- **Admin UI** improvements (`packages/admin/dashboard`).
- **Workflow / plugin** additions (`packages/core/core-flows`).

See `CLAUDE.md` for architecture, module/service/workflow patterns, and code-style conventions.
