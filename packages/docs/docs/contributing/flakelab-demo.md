---
title: FlakeLab demonstration
---

This demonstration fork contains one intentionally timing-sensitive Playwright test and one
corresponding application defect for evaluating [FlakeLab](https://flakelab.vercel.app/) against
a realistic monorepo. `FlakeLab demo: creates a new empty budget file` keeps the original
user-facing assertions. The browser filesystem bootstrap applies an unsafe 200 ms deadline while
downloading the bundled database, allowing a controlled network delay to expose the race. The
test also treats an aborted bundled-database request as an immediate onboarding failure, so fault
trials fail with a stable signature instead of waiting for the one-minute Playwright deadline.

FlakeLab should discover the causal request delay, ask before sharing the relevant application
source with Qwen through Groq, generate a bounded candidate, and prove that candidate in a
disposable Solari microVM. The generated candidate is never applied to this checkout.

## Prerequisites

- Git
- Node.js 22 or newer
- Corepack
- A Chromium browser available to Playwright
- `GROQ_API_KEY` for investigation and candidate generation
- `SOLARI_API_KEY` for isolated proof

Provider keys can be pasted into FlakeLab's hidden one-run prompts. Do not put keys in this
repository, a command argument, screenshots, or committed evidence.

## Clone and install

The demonstration is based on the open-source
[Actual Budget repository](https://github.com/actualbudget/actual). Clone this intentionally
flawed fork and install its locked dependencies:

```bash
git clone https://github.com/kelvinguchu/flakelab-actual-budget-demo.git
cd flakelab-actual-budget-demo
corepack enable
corepack yarn install --immutable
```

Run the following three processes in separate terminals. Keep the first two running while the
third terminal executes FlakeLab.

## Windows PowerShell

### Terminal 1 - watch the browser worker

```powershell
cd D:\flakelab-open-source\actual\packages\loot-core
corepack yarn vite build --config vite.config.mts --mode development --watch
```

When using a fresh clone elsewhere, replace the absolute path with the clone's
`packages\loot-core` directory.

### Terminal 2 - start Actual Budget

```powershell
cd D:\flakelab-open-source\actual\packages\desktop-client
$env:PORT = "3001"
$env:REACT_APP_BACKEND_WORKER_HASH = "dev"
corepack yarn start --mode=browser
```

Wait until Actual Budget is available at `http://localhost:3001`.

### Terminal 3 - diagnose one Playwright test

```powershell
cd D:\flakelab-open-source\actual\packages\desktop-client
$env:E2E_START_URL = "http://localhost:3001"
npx flakelab@latest diagnose "e2e/onboarding.test.ts:108" --artifacts ".flakelab/runs/presentation"
```

## macOS and Linux

Open three shells at the cloned repository.

### Terminal 1 - watch the browser worker

```bash
cd packages/loot-core
corepack yarn vite build --config vite.config.mts --mode development --watch
```

### Terminal 2 - start Actual Budget

```bash
cd packages/desktop-client
export PORT=3001
export REACT_APP_BACKEND_WORKER_HASH=dev
corepack yarn start --mode=browser
```

Wait until Actual Budget is available at `http://localhost:3001`.

### Terminal 3 - diagnose one Playwright test

```bash
cd packages/desktop-client
export E2E_START_URL=http://localhost:3001
npx flakelab@latest diagnose "e2e/onboarding.test.ts:108" \
  --artifacts ".flakelab/runs/presentation"
```

## Interactive choices

1. Approve Solari proof.
2. Approve Groq investigation and candidate generation.
3. When FlakeLab presents application sources, choose or type:

   ```text
   ../loot-core/src/platform/server/fs/index.ts
   ```

4. Approve a larger bounded runtime if the measured test duration requires it.
5. Paste `GROQ_API_KEY` and `SOLARI_API_KEY` only into the hidden prompts if they are not already
   available in the terminal environment.

## Expected evidence

The initial native scan should normally pass because localhost serves the bundled database inside
the unsafe deadline. During discovery, a delay applied to `**/data/default-db.sqlite*` should abort
the request and reproduce the onboarding failure. FlakeLab then minimizes and confirms the trigger.

A successful complete run should show:

- a Qwen-generated candidate that removes the arbitrary request deadline;
- local syntax, repository checks, and Playwright test-list preflight;
- `solari-microvm` execution;
- failing before-hostile trials;
- passing after-hostile, clean-control, and regression trials;
- equal Solari resource-created and resource-released counts;
- zero remaining live resources;
- an unchanged local application checkout.

Evidence is written beneath:

```text
packages/desktop-client/.flakelab/runs/presentation/
```

Review the candidate diff, proof JSON, and self-contained HTML report before publishing any
artifact. FlakeLab evidence is generated output and may contain local paths or failure context; it
must never contain provider keys.
