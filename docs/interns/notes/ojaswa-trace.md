\# Day 2 — Prompt Trace



\## Goal



Trace how one user sentence moves through the AI Engineering CLI and eventually becomes a tool call and a response.



\## Overall Flow



User prompt

→ `src/index.ts`

→ `src/cli/runtime.ts`

→ `src/core/agent.ts`

→ provider/model

→ tool call

→ permission check

→ tool execution

→ tool result

→ model response

→ session persistence



\## Self-Check Answers



\### 1. Which file turns `argv` into subcommand, print run, or REPL?



`src/index.ts`



It reads `process.argv.slice(2)`, parses the arguments, and decides whether to run a subcommand, print mode, or interactive REPL.



\### 2. Where does deny-rule → allow-rule → mode ordering live?



`src/core/permissions.ts`



`checkPermission()` evaluates permissions in this order:



1\. Deny rules

2\. Allow rules

3\. Permission mode

4\. Read-only checks

5\. User approval when required



\### 3. Which class manages MCP servers/tools and what are the tools called?



`McpRegistry` in `src/mcp/client.ts` manages MCP server connections and registered tools.



MCP tools are given names in this format:



`mcp\_\_<serverName>\_\_<toolName>`



For example:



`mcp\_\_github\_\_search`



The `.aet/mcp.json` configuration is used for MCP server configuration.



\### 4. What makes `--provider mock` deterministic?



`src/providers/mock.ts`



`MockProvider` is an offline provider used for tests and demos. When an `AET\_MOCK\_SCRIPT` is supplied, it follows the predefined steps in order, such as:



1\. Return text

2\. Request a tool call

3\. Return another response



This allows the agentic loop to run without an API key using predictable scripted behavior.



\### 5. What happens to a tool call that needs approval in `-p` mode?



In non-interactive/print mode, there is no user available to approve the request.



Therefore, a tool call that requires approval is denied unless it is already allowed by the configured permissions/mode.



\### 6. Where is a turn persisted, and in what format?



`src/core/session.ts`



`addMessage()` adds the message to the session and calls:



`append({ type: "message", message })`



The `append()` method uses `appendJsonLine()`, so session entries are persisted as JSON Lines (JSONL) entries.



\## Read Tool



For the tool implementation, I selected `Read`.



Implementation:



`src/tools/files.ts`



Documentation:



`docs/tools.md`



`Read` is a read-only tool that reads text files and returns numbered lines.



\## Conclusion



The main agent flow is:



User prompt → CLI → Runtime → Agent → Provider → Tool call → Permission check → Tool → Tool result → Agent → Final response.



Session messages are persisted through `src/core/session.ts`.



