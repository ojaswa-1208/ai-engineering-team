# Intern guide — AI Engineering Team (`aet`)

**Start here.** This folder is the first-week onboarding track for interns at Ryuksaidso. It assumes you can write TypeScript but have never seen this codebase.

**Author: Faizan Hameed (https://faizcasm.me) — Team Lead of Ryuksaidso** · Repo: <https://github.com/Ryuksaidso/ai-engineering-team> · License: MIT

## What you will ship in week one

By Friday you should have **two merged pull requests**: one test-only PR and one small feature/fix PR, plus a docs/CHANGELOG update. Everything is built on the rules in [CONTRIBUTING.md](../../CONTRIBUTING.md).

| Day | Focus | Guide |
|---|---|---|
| **Day 1** | Clone, build, verify, run the CLI offline | [onboarding.md](onboarding.md) |
| **Day 2** | Understand how a prompt becomes a tool call | [codebase-tour.md](codebase-tour.md) |
| **Day 3** | First PR: tests for an untested helper | [first-week.md](first-week.md#day-3--first-pr-tests) |
| **Day 4** | Second PR: a small, scoped change | [first-week.md](first-week.md#day-4--second-pr-small-change) |
| **Day 5** | Docs, CHANGELOG, PR review and merge | [first-week.md](first-week.md#day-5--docs-review-and-merge) |

## Read in this order

1. [**onboarding.md**](onboarding.md) — environment setup, the four commands you run every day, safety rules (no API keys, no network, no real `~/.aet`).
2. [**codebase-tour.md**](codebase-tour.md) — the directory map, the request flow, and "where do I change X".
3. [**first-week.md**](first-week.md) — the day-by-day task list with a definition of done for each task.
4. [../../CONTRIBUTING.md](../../CONTRIBUTING.md) — code style, commit messages, PR checklist. **This is binding**; a PR that breaks the checklist gets sent back.
5. [../../README.md](../../README.md) — what the product actually does; skim Commands, Permission modes and Team pipeline.

Reference docs while you work: [../configuration.md](../configuration.md) · [../agents.md](../agents.md) · [../tools.md](../tools.md) · [../permissions.md](../permissions.md) · [../providers.md](../providers.md) · [../mcp.md](../mcp.md).

## Ground rules for week one

- **`dependencies` stays `{}`.** The zero-dependency promise is the project's headline feature. Node built-ins only — if you think you need a package, ask first.
- **Everything you run must work offline** with `--provider mock`. No API keys, no network, no secrets in code or tests.
- **Keep your real `~/.aet` untouched** by exporting `AET_HOME=/tmp/aet-home` while you experiment.
- **Read before you write.** The permission system auto-approves reads; the smallest change that satisfies the task is the right change.
- **Public behaviour needs a test.** Bug fixes get a regression test; new flags/tools get a happy-path and a failure-path test.
- **Never weaken a test to make it pass.** Fix the code, or explain in the PR why the test was wrong.
- **Two PRs minimum, opened as drafts first.** Your mentor reviews before you mark them ready.

## The four commands you will type most

```bash
npm run typecheck   # tsc --noEmit — must be clean before every commit
npm test            # build + node --test dist/test — must be green before every commit
npm run dev -- --help   # build, then run the CLI from dist/
node dist/src/index.js doctor   # installation/config/credential/workspace diagnosis
```

## Getting unblocked

1. Search this folder and `docs/`.
2. Read the error end to end — messages here follow *"what failed, where, and what to do next"*; the fix is usually named in the message.
3. Ask your mentor with the command you ran, the full output, and what you expected.

---

**Author: Faizan Hameed (https://faizcasm.me) — Team Lead of Ryuksaidso** · Copyright (c) 2026 Faizan Hameed · [MIT License](../../LICENSE)
