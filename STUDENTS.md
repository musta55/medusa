# Medusa — Student Setup Guide

This is a **course fork of [medusajs/medusa](https://github.com/medusajs/medusa)** — the open-source
commerce **framework monorepo** (30+ TypeScript packages). It is pinned to a verified baseline:

- **Version:** `@medusajs/medusa` **2.17.2**
- **Baseline commit:** `8f97e3f371`
- Everyone starts from this exact commit. Do **not** pull from upstream `develop`/`main` — it moves and can break.

📚 **Official documentation:** <https://docs.medusajs.com>

---

## 1. Prerequisites

| Tool | Version | Notes |
| --- | --- | --- |
| **Node.js** | **20 LTS or 22 LTS** | Do not use Node 24 — a few test suites fail on it. Use [nvm](https://github.com/nvm-sh/nvm). |
| **Corepack** | ships with Node | Run `corepack enable` — it activates the repo-pinned **Yarn 3.2.1** automatically. Do not install Yarn globally. |
| **Docker** | any recent | Only needed for **integration tests** (and for running a real store DB). |
| **Git** | any | |

**OS:** Linux, macOS, or **Windows via WSL2** recommended.

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

## 3. Start the project

```bash
# Admin dashboard dev server (Vite):
yarn workspace @medusajs/dashboard dev

# Design-system component explorer (Storybook, http://localhost:6006):
yarn workspace @medusajs/ui storybook

# Full store + admin with seed data (needs the Postgres from step 5):
npx create-medusa-app@latest my-store \
  --db-url "postgres://postgres:postgres@localhost:5433/my_store"
cd my-store && yarn dev        # admin at http://localhost:9000/app
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

## 6. Project documentation & policies (required reading)

📚 **Official documentation:** <https://docs.medusajs.com> — framework concepts at
[docs.medusajs.com/learn](https://docs.medusajs.com/learn), API reference at
[docs.medusajs.com/api](https://docs.medusajs.com/api).

Medusa has its own established contribution processes. They are **not restated here** — you are
responsible for finding, reading, and following them from the sources below:

| You must take care of | Where to find it |
| --- | --- |
| How to use the tool | <https://docs.medusajs.com> |
| Code review process | [CONTRIBUTING.md — Pull Requests](CONTRIBUTING.md#pull-requests) |
| Bug / issue resolution process | [CONTRIBUTING.md — Issues before PRs](CONTRIBUTING.md#issues-before-prs) |
| Pull request conventions & PR policies | [CONTRIBUTING.md — Workflow](CONTRIBUTING.md#workflow) (branches, commits, PRs) |
| AI policies | [CLAUDE.md](CLAUDE.md) (this repo's AI conventions); check [CONTRIBUTING.md](CONTRIBUTING.md) for the current policy |
