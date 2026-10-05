# First week — the work

Five days, two pull requests, one demo. Every task has a **definition of done**; you are not done until every line in it is true.

**Author: Faizan Hameed (https://faizcasm.me) — Team Lead of Ryuksaidso** · Repo: <https://github.com/Ryuksaidso/ai-engineering-team> · License: MIT

Before starting: finish [onboarding.md](onboarding.md) and [codebase-tour.md](codebase-tour.md), and read [CONTRIBUTING.md](../../CONTRIBUTING.md) end to end.

## Standing rules for every day

```bash
npm run typecheck   # must be silent
npm test            # must be green — build + node --test dist/test
git status          # never commit dist/, node_modules/, .env, or anything in $AET_HOME
```

- Branch per piece of work: `git checkout -b docs/intern-<name>-<topic>` or `feat/…` / `fix/…` / `test/…`.
- Commit subjects: imperative, lowercase, ≤ 72 chars, with the suggested prefixes from CONTRIBUTING (`docs:`, `test:`, `tools:`, `cli:`, `perm:`, `chore:`).
- Open the PR as a **draft** the first time; your mentor marks it ready after a first read.
- If a task turns out to be bigger than a day, stop, write what you learned in the PR description, and pick a smaller one. Scope discipline is the skill being taught.

---

## Day 1 — Environment and product literacy

**Work:** [onboarding.md](onboarding.md), in order.

Tasks:

1. Clone, `npm install`, build, typecheck, test.
2. Export `AET_HOME=/tmp/aet-home`; work in `/tmp/aet-demo`.
3. Run `doctor`, `agents`, `tools`, `config`, `help team` and read the output.
4. Produce two offline runs and save the output:
   ```bash
   node dist/src/index.js --provider mock "offline demo run"
   node dist/src/index.js -p --output-format json --provider mock "create hello.txt"
   ```
5. Read `README.md` sections *Commands & flags*, *Permission modes*, *Team pipeline*.
6. Skim the six files in `docs/` — you do not need to retain them, only know where each answer lives.

**Definition of done**

- [ ] `npm run typecheck` silent and `npm test` green on your machine.
- [ ] `doctor` output explained to your mentor line by line (2 minutes, verbal).
- [ ] You can state, without looking, what `aet -p`, `--plan` and `--provider mock` each do.
- [ ] Your shell has `AET_HOME` exported and no real API key anywhere.

---

## Day 2 — Read the code, trace a prompt

**Goal:** be able to explain how one user sentence turns into a `Write` tool call and back into an answer.

Tasks:

1. Read `src/index.ts` → `src/cli/runtime.ts` → `src/core/agent.ts` (top to bottom, once, without trying to understand every line).
2. Pick one tool you find interesting — `Read`, `Bash` or `Task` — and read its implementation plus its entry in [docs/tools.md](../tools.md).
3. Trace permissions: read `src/core/permissions.ts` and map the four modes in [docs/permissions.md](../permissions.md) to the code that enforces them.
4. Run one verbose offline session and match the log to the loop steps:
   ```bash
   node dist/src/index.js -v --provider mock "create hello.txt"
   ```
   Then inspect the session file under `$AET_HOME/sessions/`.
5. Write your trace as a short PR: a new file `docs/interns/notes/<your-name>-trace.md` — **or** give the walkthrough verbally if your mentor prefers. Ask on day 1 which one they want.

**Self-check — answer these from memory before you ask:**

1. Which file turns `argv` into a subcommand, a print run, or a REPL?
2. Where does the deny-rule-then-allow-rule-then-mode ordering live?
3. Which class loads `.aet/mcp.json` and what are its tools called?
4. What makes `--provider mock` deterministic?
5. What happens to a tool call that needs approval in `-p` mode?
6. Where is a turn persisted, and in what format?

**Definition of done**

- [ ] You can answer all six questions with a file path.
- [ ] Your trace/notes PR is opened (or the verbal walkthrough is scheduled).
- [ ] Still no changes to `src/`.

---

## Day 3 — First PR: tests

**Goal:** your first merged PR. Test-only, zero behaviour change — the safest possible first contribution.

### Find your target

Pick one exported helper that is **weakly covered or uncovered**. Candidates to check first:

| Helper | File | Why it is a good target |
|---|---|---|
| `indent`, `countLines`, `plural` | `src/util/text.ts` | pure, obvious edge cases |
| `globToRegExp`, `matchGlob` | `src/util/text.ts` | `**`, `?`, `[a-z]`, case sensitivity |
| `timeAgo`, `shortTimestamp` | `src/util/time.ts` | boundaries (0s, 59s, 60s, 1d) |
| `looksBinary`, `writeTextFileAtomic` | `src/util/fsx.ts` | needs `node:fs` + `node:os` temp dirs |
| a tool's failure path | `src/tools/*.ts` | error message must be actionable |

Confirm coverage by reading `test/util.test.ts` (and friends) — the helper may already be tested. If everything you picked is covered, pick the next one.

### Do the work

```bash
git checkout -b test/intern-<name>-<helper>
```

- Add cases to the suite that owns the helper (`test/util.test.ts` for `src/util/*`), or a new `test/<area>.test.ts` if there is no home for it.
- Aim for **6–10 assertions**: happy path, boundary, and at least one failure/empty input.
- Test the *behaviour contract*, not the implementation. `truncateEnd("abcdefghij", 5) === "abcd…"` is a contract; a snapshot of internal state is not.

```bash
npm run typecheck && npm test
git diff --stat          # only test files should appear
```

Open the PR with the CONTRIBUTING template:

```markdown
## What
Add coverage for <helper> in src/util/<file>.ts (test-only).

## Why
<the edge case that could silently regress>

## How tested
npm run typecheck && npm test  — green; no src/ changes (git diff --stat)

## Checklist (see CONTRIBUTING.md)
- [x] typecheck + tests pass
- [x] no runtime dependency added
- [x] no src/ behaviour change
```

**Definition of done**

- [ ] Only files under `test/` changed.
- [ ] `npm run typecheck` silent, `npm test` green.
- [ ] At least one boundary case and one failure/empty-input case included.
- [ ] PR opened as draft, then marked ready after your mentor's first read.
- [ ] Your mentor's comments addressed in new commits (no force-push after review starts).

---

## Day 4 — Second PR: a small change

**Goal:** ship a real behaviour or documentation change end to end: code → test → docs → CHANGELOG.

**Pick exactly one**, agreed with your mentor before you start (they may have an issue waiting for you):

| Option | Scope | Files |
|---|---|---|
| **A. Improve a command's help text** | Add a missing example or clarify a flag description | `src/cli/help.ts`, `test/cli.test.ts`, README table if the flag list changes |
| **B. Fix a stale doc claim** | Find where `docs/*.md` disagrees with the code, correct the doc | `docs/*.md` (+ `CHANGELOG.md`) |
| **C. Add a failure-path test + fix the message** | Trigger a tool error, make the message actionable (*what failed, where, what next*) | `src/tools/*.ts`, `test/*.test.ts`, [docs/tools.md](../tools.md) |
| **D. Small config/CLI gap** | e.g. a config key that `aet config` does not list | `src/core/config.ts`, `src/cli/commands.ts`, [docs/configuration.md](../configuration.md), tests |

Process:

1. **Write the failing test first** (unless the change is docs-only). Run it, watch it fail for the right reason.
2. Make the smallest change that makes it pass.
3. Update the docs that describe the behaviour — `aet help <command>` text, `docs/*.md`, README where relevant.
4. Add an entry under `[Unreleased]` in `CHANGELOG.md` (create the heading if it is missing) using Keep a Changelog wording: `### Added` / `### Changed` / `### Fixed`.
5. Verify the offline path still works:
   ```bash
   npm run typecheck && npm test
   node dist/src/index.js -p --output-format json --provider mock "smoke test"
   ```

**Definition of done**

- [ ] A test covers the new/changed behaviour (both happy and failure path).
- [ ] `npm run typecheck` silent and `npm test` green.
- [ ] `dependencies` in `package.json` is still `{}`.
- [ ] Docs and `CHANGELOG.md` updated; no unrelated reformatting.
- [ ] Works offline with `--provider mock`.
- [ ] Full CONTRIBUTING PR checklist ticked in the description.

---

## Day 5 — Docs, review, merge, retro

Tasks:

1. **Finish your PRs.** Respond to every review comment; push fix commits with the same prefix style (`test:`, `docs:`, `fix:` style subjects).
2. **Verify the checklist yourself before you ask your mentor to.** Run each item from CONTRIBUTING's *Pull request checklist* and paste the results into the PR.
3. **Confirm the published docs still match reality** — your day-4 change may have touched `docs/*.md` or the README table.
4. **Demo (10 minutes):** your trace of the agent loop (day 2) plus your two PRs. Live, offline, `--provider mock`.
5. **Retro notes** in your PR or to your mentor:
   - What surprised you about the codebase?
   - Which rule in CONTRIBUTING cost you the most time, and why do you think it exists?
   - One thing the docs got wrong or explained too late.

**Definition of done**

- [ ] Both PRs merged (or approved with a scheduled merge date).
- [ ] `main` builds and tests green after your merge — pull and re-run before you celebrate.
- [ ] Demo delivered.
- [ ] You know where next week's work will come from (ask your mentor for the backlog).

## What not to do in week one

- Do not add a runtime dependency, a build tool, a linter config or a formatter — the project is deliberately minimal.
- Do not refactor files you are not assigned. Drive-by reformatting blocks review.
- Do not commit `dist/`, `node_modules/`, `.env`, or session files from `$AET_HOME`.
- Do not open a PR that mixes a feature, a refactor and docs.
- Do not use `--dangerously-skip-permissions` outside a throwaway directory.

## After week one

- Claim an issue labelled for new contributors and follow the same loop: agree scope → draft PR → review → merge.
- Read [docs/agents.md](../agents.md) and write your own custom agent in `.aet/agents/*.md` — it is the fastest way to understand how roles drive behaviour.
- Try `aet team "<small goal>" --plan-only --provider mock` and read the plan the `tech-lead` produces.

---

**Author: Faizan Hameed (https://faizcasm.me) — Team Lead of Ryuksaidso** · Copyright (c) 2026 Faizan Hameed · [MIT License](../../LICENSE)
