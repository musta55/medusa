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

## 6. How to use the tool

Medusa provides the building blocks of an e-commerce platform: product catalog, carts, orders,
pricing, promotions, inventory, payments, and an admin dashboard — exposed through Store and Admin
REST APIs and customizable via modules and workflows.

- Scaffold a store (step 3), then manage products/orders in the **admin** at `http://localhost:9000/app`.
- Explore the **Store & Admin APIs**: <https://docs.medusajs.com/api>.
- Learn the framework concepts (modules, workflows, API routes): <https://docs.medusajs.com/learn>.
- Architecture and code patterns used in this repo: see `CLAUDE.md` in the repo root.

---

## 7. How to review the code

Code review is part of the work, not an afterthought:

1. Read the linked issue first, then the diff (GitHub → **Files changed**).
2. Check out the branch locally, build, and run the relevant tests
   (`yarn workspace @medusajs/<pkg> test`).
3. Leave **line comments** for specific problems and finish with a summary review —
   **Approve** or **Request changes**.
4. Look for: correctness, tests covering the new behavior, naming/clarity, and unintended
   changes (lockfiles, build artifacts, formatting noise).

Every PR needs at least **one teammate approval** before merge — no self-merges. Review the code,
not the person; be specific and constructive.

---

## 8. Pull requests (PRs)

1. Branch off `develop`: `git checkout -b feature/<short-name>` (or `fix/<issue-number>-<slug>`).
2. Keep commits small with meaningful messages.
3. Run the tests (section 4) before pushing.
4. Push to the course fork and open the PR against the course fork's `develop` — **never upstream**.
5. In the description: what changed, why, how you tested it, and the linked issue (`Closes #12`).
6. Address review comments with follow-up commits (avoid force-pushes during review), then
   re-request review.

---

## 9. Issue resolution process

1. All work is tracked as **GitHub Issues** on the course fork — bug, feature, or task.
2. Before coding: pick or create an issue, get it **assigned** to you, and outline your approach
   in a comment if it's non-trivial.
3. One issue → one branch → one PR, linked with `Closes #<n>` so the issue closes automatically
   on merge.
4. Blocked for more than a day? Say so on the issue (what you tried, where you're stuck) instead
   of going quiet.
5. An issue is **done** when its PR is merged and the behavior is verified.

---

## 10. AI policies

AI assistants (Claude, ChatGPT, Copilot, …) are allowed as a learning and productivity aid,
under these rules:

- **You are the author.** Understand and be able to explain every line you submit — "the AI
  wrote it" is never an explanation.
- **Test before you commit.** Never push AI-generated code you haven't run and tested locally.
- **Disclose it.** Note meaningful AI assistance in the PR description
  (e.g. *"AI-assisted: first draft of the workflow + its tests"*).
- **Protect data.** Never paste secrets, tokens, or private data into AI tools.
