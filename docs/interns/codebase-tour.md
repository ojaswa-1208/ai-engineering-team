# Codebase tour

Where things live, how a prompt becomes a tool call, and where to change X. Read this with your editor open on the repo.

**Author: Faizan Hameed (https://faizcasm.me) — Team Lead of Ryuksaidso** · Repo: <https://github.com/Ryuksaidso/ai-engineering-team> · License: MIT

## The 30-second map

```
ai-engineering-team/
├── src/            the `aet` CLI (TypeScript ESM, strict, zero runtime deps)
│   ├── index.ts    entry point: parse argv → subcommand | print run | REPL
│   ├── cli/        argument parsing, help text, subcommands, -p print runs, shared Runtime
│   ├── core/       config, auth, paths, the agent loop, permissions, sessions, context
│   ├── tools/      the 13 built-in tools + the tool interface
│   ├── providers/  anthropic / openai-compatible / ollama / mock over one Provider interface
│   ├── team/       the 11 role definitions + the plan → delegate → review orchestrator
│   ├── mcp/        MCP server config (.aet/mcp.json) and client (stdio + HTTP)
│   ├── slash/      custom `/commands` loaded from .aet/commands/*.md
│   ├── ui/         REPL, approval prompts, markdown renderer, spinner
│   └── util/       text, diff, fs, color, time, token helpers
├── test/           node:test suites → compiled to dist/test
├── docs/           feature reference docs (plus this interns/ folder)
├── backend/        separate service (Prisma, workers, graph workflows) — NOT part of the CLI build
├── dist/           build output (gitignored)
├── tsconfig.json   strict, NodeNext ESM; includes src/ + test/, excludes backend/
└── package.json    scripts; dependencies is deliberately {}
```

`backend/` is excluded from `tsconfig.json` and from your first week. Ignore it unless your mentor points you at it.

## How a prompt becomes a tool call

```
argv
 └─ index.ts  main()
     ├─ cli/args.ts        parse flags + split at a subcommand
     ├─ cli/help.ts        command/flag metadata, renderHelp()
     ├─ cli/commands.ts    `aet doctor|mcp|config|team|…`  ─┐
     ├─ cli/print.ts       `-p` / piped stdin, JSON output  ─┼─→ cli/runtime.ts  Runtime
     └─ ui/repl.ts         interactive TTY session          ─┘     (config, provider,
                                                                     tools, MCP, session,
                                                                     permission gate, system prompt)
                                                                     └─→ core/agent.ts  Agent.run()
                                                                         │
   ┌─────────────────────────────────────────────────────────────────────┘
   │  loop (bounded by --max-turns, default 60)
   │   1. core/context.ts        compact history if the window is getting full
   │   2. core/systemPrompt.ts   build the system prompt (+ AET.md instructions)
   │   3. providers/*            provider.complete(messages, tools) → text and/or tool calls
   │   4. core/permissions.ts    checkPermission(): deny rules → allow rules → mode → you
   │   5. tools/*                execute the tool (Read/Write/Bash/Task/…)
   │   6. core/session.ts        append the turn to $AET_HOME/sessions/<id>.jsonl
   └──────────── until the model returns a final answer ────────────────┘
```

Two files are worth reading before anything else:

- **`src/core/agent.ts` (≈450 lines)** — the whole loop, events and token/cost accounting. Read this first on day 2.
- **`src/cli/runtime.ts`** — the wiring shared by the REPL, print runs and subcommands. If you wonder "where does config actually get applied?", it is here.

## Module-by-module

| Directory | Files to know | Responsibility |
|---|---|---|
| `src/cli/` | `args.ts`, `help.ts`, `commands.ts`, `print.ts`, `runtime.ts` | Flag parsing and specs, help/version text, subcommand implementations, `-p` output (`text`/`json`/`stream-json`), shared `Runtime`. |
| `src/core/` | `agent.ts`, `permissions.ts`, `config.ts`, `session.ts`, `context.ts`, `systemPrompt.ts`, `auth.ts`, `paths.ts`, `messages.ts`, `types.ts` | The agentic loop; the 4 permission modes + allow/deny rules; config precedence (flags → env → project `.aet/` → `~/.aet`); JSONL sessions; context compaction; system prompt; credential store; `Provider`/`Message`/`ToolSpec` types. |
| `src/tools/` | `types.ts`, `index.ts`, `files.ts`, `bash.ts`, `search.ts`, `webfetch.ts`, `todo.ts`, `task.ts` | The `Tool` interface and `ToolContext`, `builtinTools()`/`findTool()`/`toSpecs()`, then one file per tool family. `task.ts` spawns a specialist subagent; `bash.ts` owns background shells (`BashOutput`, `KillShell`). |
| `src/providers/` | `index.ts`, `http.ts`, `models.ts`, `anthropic.ts`, `openai.ts`, `ollama.ts`, `mock.ts` | `resolveProvider()` (flag → model prefix → inference → config), the shared `fetch` wrapper and `ProviderError`, model metadata and `costUsd()`, one adapter per backend, and the deterministic offline `mock`. |
| `src/team/` | `agents.ts`, `orchestrator.ts` | The 11 role definitions (and `parseFrontmatter()` shared with custom agents/commands); `aet team`'s plan → delegate → review pipeline with dependency-aware parallelism. |
| `src/mcp/` | `config.ts`, `client.ts` | Reading `.aet/mcp.json` / `~/.aet/mcp.json`; `McpRegistry` connecting stdio and HTTP servers and exposing tools as `mcp__<server>__<tool>`. |
| `src/ui/` | `repl.ts`, `prompt.ts`, `markdown.ts`, `spinner.ts` | The interactive session, slash commands and `!`/`@` input forms; the approval prompt (`QueuedGate`); terminal markdown and progress. |
| `src/slash/` | `commands.ts` | Custom `/name` commands from `.aet/commands/*.md`, frontmatter parsing, `$ARGUMENTS` and `@file` substitution. |
| `src/util/` | `text.ts`, `diff.ts`, `fsx.ts`, `color.ts`, `time.ts`, `tokens.ts` | Pure helpers: wrapping/truncating/globs/shell quoting, line diffs, atomic file writes and `walk()`, ANSI styling, `estimateTokens`, `TokenCounter`. **Pure, dependency-free — the easiest place for a first PR.** |

## Where do I change X?

| Goal | Start in | Then check |
|---|---|---|
| A new CLI flag | `src/cli/args.ts` (spec) + `src/cli/help.ts` (docs text) | `src/cli/runtime.ts` (consume it), `test/args.test.ts`, `docs/`, `CHANGELOG.md` |
| A new subcommand | `src/cli/commands.ts` + `COMMANDS` in `src/cli/help.ts` | `test/cli.test.ts`, README command table |
| Tool behaviour or schema | the tool's file in `src/tools/` | [docs/tools.md](../tools.md), `src/core/permissions.ts` (approval class), tests |
| A new built-in tool | `src/tools/<new>.ts` + register in `src/tools/index.ts` | permission handling, `src/tools/types.ts`, tests, `aet tools` output |
| Permission rules/modes | `src/core/permissions.ts` | [docs/permissions.md](../permissions.md), `test/permissions.test.ts` — read-only must stay auto-approved |
| Config keys | `src/core/config.ts` (defaults + merge + `envOverrides`) | `docs/configuration.md`, `aet config` listing, tests |
| Provider/model support | `src/providers/` (`models.ts`, adapter) | [docs/providers.md](../providers.md), offline `mock` equivalent, `aet doctor` |
| A role or its prompt | `src/team/agents.ts` | [docs/agents.md](../agents.md), `aet agents` output |
| The team pipeline | `src/team/orchestrator.ts` | plan parsing, parallelism, reviewer step; `aet team --plan-only --provider mock` |
| REPL input or slash commands | `src/ui/repl.ts`, `src/slash/commands.ts` | [docs/agents.md](../agents.md#custom-agents--commands) |
| Text output/colour | `src/util/color.ts`, `src/ui/markdown.ts`, `src/cli/print.ts` | `--output-format json` must stay backwards compatible |

## Tests

Suites live in `test/*.test.ts`, compile into `dist/test`, and run with Node's built-in runner — no framework, no network, no API keys:

| File | Covers |
|---|---|
| `test/args.test.ts` | flag parsing, defaults, `splitAtCommand`, option specs |
| `test/cli.test.ts` | subcommands, help/version output, print/JSON output |
| `test/core.test.ts` | config merge, sessions, agent/core behaviour |
| `test/permissions.test.ts` | modes, allow/deny rules, read-only heuristics |
| `test/util.test.ts` | text, glob, diff and other pure helpers |

Add tests as a new `test/<area>.test.ts` file or extend the suite that already owns the code. Shape:

```ts
import { test } from "node:test";
import assert from "node:assert/strict";

test("matchGlob treats ** as any depth", () => {
  assert.equal(matchGlob("src/**/*.ts", "src/a/b/c.ts"), true);
});
```

## Rules that bite if you ignore them

- **ESM + `NodeNext`:** relative imports must carry the `.js` extension — `import { x } from "./y.js"` — even though the source file is `.ts`.
- **`strict`, no `any` where a real type exists.** Prefer discriminated unions and `interface`.
- **`dependencies: {}`.** Node built-ins only; new *dev* dependencies need justification in the PR.
- **Style:** 2-space indent, double quotes, semicolons, named exports, small single-purpose modules. Copy the style of the file you are in.
- **Comments explain *why*, not *what*.** Keep file-header docblocks accurate when behaviour changes.
- **Errors are actionable:** what failed, where, and what the user should do next.

Next: [first-week.md](first-week.md).

---

**Author: Faizan Hameed (https://faizcasm.me) — Team Lead of Ryuksaidso** · Copyright (c) 2026 Faizan Hameed · [MIT License](../../LICENSE)
