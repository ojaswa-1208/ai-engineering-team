# Onboarding — day 1

Get from a fresh machine to a working, offline `aet` build in under an hour, then prove it works.

**Author: Faizan Hameed (https://faizcasm.me) — Team Lead of Ryuksaidso** · Repo: <https://github.com/Ryuksaidso/ai-engineering-team> · License: MIT

## 1. Prerequisites

| Requirement | Check | Why |
|---|---|---|
| Node.js **>= 20.10.0** | `node --version` | Native `fetch`, `node:test`, ESM. Nothing older runs it. |
| npm | `npm --version` | Ships with Node. |
| git | `git --version` | Branches and PRs. |
| A code editor with a TS plugin | — | `tsconfig.json` is strict; let the editor show errors inline. |

There is **nothing else to install**. The CLI has zero runtime dependencies — only `typescript` and `@types/node` as devDependencies.

## 2. Clone and build

```bash
git clone https://github.com/Ryuksaidso/ai-engineering-team.git
cd ai-engineering-team
npm install     # devDependencies only; the `prepare` script runs the build
```

`npm install` compiles `src/` and `test/` into `dist/` (`tsc -p tsconfig.json`). If it finished without errors you already have a working build.

```bash
npm run build        # explicit rebuild after any source change
npm run typecheck    # strict types, no emit — your pre-commit gate
npm test             # build + node --test "dist/test"
```

Expected: typecheck silent, tests green, no network calls.

## 3. Isolate yourself before you experiment

Two environment variables keep your experiments away from your real machine state:

```bash
export AET_HOME=/tmp/aet-home     # sessions/config live here, not in ~/.aet
mkdir -p /tmp/aet-demo && cd /tmp/aet-demo   # a throwaway workspace
```

Rules for week one:

- **Never commit a real API key.** Use `export AET_HOME=…` plus the `mock` provider.
- **Never run `--dangerously-skip-permissions`** against a directory that has work you care about. Use `--plan` (read-only) or `--provider mock` instead.
- Everything you demo in week one must run with `--provider mock`.

## 4. Verify the installation

Go back to the repo and run the CLI straight from `dist/`:

```bash
cd ai-engineering-team
node dist/src/index.js --help
node dist/src/index.js doctor
```

`doctor` reports Node runtime, `AET_HOME` writability, merged configuration, credentials, provider resolution, MCP servers, workspace and instruction files. On a machine with no keys you should see checks pass with provider reachability skipped or failing gracefully — **0 issues is not required on day 1**; understanding each line is.

Offline smoke test (deterministic, no key, no network):

```bash
node dist/src/index.js --provider mock "offline demo run"
node dist/src/index.js -p --output-format json --provider mock "create hello.txt"
```

The second command prints one JSON object containing `session_id`, `turns`, `usage`, `cost_usd` and `model`. Save that output — you will use it in [first-week.md](first-week.md).

Optionally expose the binaries globally:

```bash
npm link      # makes `aet`, `ai-team`, `ai-engineering-team` available everywhere
aet --version
```

## 5. Project scaffolding (optional but instructive)

```bash
mkdir -p /tmp/aet-init && cd /tmp/aet-init
node /path/to/ai-engineering-team/dist/src/index.js init
ls -la .aet AET.md
```

`init` writes `.aet/config.json`, `AET.md`, a sample `.aet/agents/*.md` and a sample `.aet/commands/*.md`. Read `AET.md` — it is the workspace instruction file `aet` loads on every run (order: `AET.md` → `AGENTS.md` → `CLAUDE.md` → `.aet/AGENTS.md`).

## 6. Tour the product surface

Run these from the repo and skim the output — this is the vocabulary the team uses:

```bash
node dist/src/index.js agents     # the 11 roles
node dist/src/index.js tools      # the 13 tools + MCP servers
node dist/src/index.js config     # merged configuration
node dist/src/index.js help team  # extended usage for one subcommand
```

## 7. End-of-day-1 checklist

- [ ] `npm run typecheck` is clean.
- [ ] `npm test` is green.
- [ ] `node dist/src/index.js doctor` runs and you can explain each line.
- [ ] `AET_HOME=/tmp/aet-home` is exported in the shell you use for experiments.
- [ ] You produced a JSON result with `--provider mock` and know where sessions are written (`$AET_HOME/sessions/<id>.jsonl`).
- [ ] You have read [CONTRIBUTING.md](../../CONTRIBUTING.md) **Code style** and **Pull request checklist** once.
- [ ] You have skimmed the [README](../../README.md) sections *Commands & flags* and *Permission modes*.

Next: [codebase-tour.md](codebase-tour.md).

---

**Author: Faizan Hameed (https://faizcasm.me) — Team Lead of Ryuksaidso** · Copyright (c) 2026 Faizan Hameed · [MIT License](../../LICENSE)
