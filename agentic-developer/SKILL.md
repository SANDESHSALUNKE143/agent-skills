---
name: agentic-developer
description: >-
  Builds a new agentic application on the organization reference, one specialist
  at a time: agent.yaml, a registered plugin, a workflow, approval, and an
  evaluation case, including security, rate limits, denial of service,
  traceability, and observability. Use when creating an agent, a plugin, a
  workflow, or any problem domain, or when the user mentions agentic
  architecture, specialists, agent.yaml, the gateway, LangChain, or LangGraph.
---

# Agentic developer

The reference is the `agentic-architecture` repository. This skill is the standard for any problem. Specialist names, tool names, and fixtures belong to that problem. The folders, the contract, the gateway, and the approval rule stay. Python in the numbered stages follows the `senior-python` skill. Console visuals and shared controls follow the `react-typescript` skill. Cloud resources follow the Azure and Terraform skills.

`docs/assets/architecture.svg` is the request picture. A change to the request path, the console flow, the agent sequence, memory, the gateway, approval, an outside system, or the trace worker updates that SVG in the same change. Labels name real folders and real behavior.

## Build a new application

A new application uses this layout for whatever problem it solves. Build one specialist at a time. Do not start a second runtime, a second gateway, a per-agent loop, or an automation graph as the sequencer. Do not add a stage beside `01-entry` through `08-response`. `agentic/bootstrap.py` already binds those folders to `entry`, `guardrails`, `workflows`, `loops`, `tools`, `jobs`, `observability`, and `response`. A path with a hyphen is loaded with `agentic.platform.loader.load_module`, the same way `registry.py` loads a domain tool module. Do not add a second import mechanism.

## Folder structure

The console is `ui`. The request path is `agentic`, numbered in the order a request moves. `agentic/platform` is configuration and composition. It is not a request step. Place new code in the folder that already owns that concern. Do not add a top-level folder, a second `agentic` tree, or a Python package inside a specialist folder.

```text
agentic-architecture/
├── ui/
│   ├── admin-dashboard/
│   │   └── src/
│   │       ├── views
│   │       ├── agent
│   │       ├── plugins
│   │       └── config
│   └── packages/
│       └── controls/
│           └── src
├── agentic/
│   ├── platform
│   ├── 01-entry/
│   │   └── routes
│   ├── 02-guardrails/
│   │   ├── auth
│   │   ├── limits
│   │   ├── policy
│   │   └── connections
│   ├── 03-workflows/
│   │   ├── router
│   │   ├── sequences
│   │   ├── engine
│   │   └── state
│   ├── 04-loops/
│   │   ├── runtime
│   │   ├── llm
│   │   ├── memory
│   │   └── specialists
│   ├── 05-tools/
│   │   └── mcp/
│   │       ├── tool-gateway
│   │       └── <domain>-tools
│   ├── 06-jobs/
│   │   ├── queue
│   │   └── workers
│   ├── 07-observability/
│   │   ├── trace
│   │   ├── evaluation
│   │   └── sinks
│   └── 08-response/
│       ├── contracts
│       └── project
├── mocks/
│   └── <system>
├── infrastructure
├── docs
└── tests
    ├── unit
    ├── integration
    ├── security
    └── agent-evaluation
```

| New work | Folder |
| --- | --- |
| One specialist | `agentic/04-loops/specialists/<name>-agent/agent.yaml` only. No loop module beside the YAML |
| A domain plugin | `agentic/05-tools/mcp/<domain>-tools/tools.py`, then one entry in `tool-gateway/registry.py` |
| A shared port | `agentic/05-tools/mcp/tool-gateway/ports.py` |
| A sequence | `agentic/03-workflows/sequences/<name>/workflow.py` |
| Who may hand work to whom | `agentic/02-guardrails/policy/delegation.py` |
| Phrase that selects the sequence | `agentic/03-workflows/router/router.py` |
| A route | The matching module under `agentic/01-entry/routes/` |
| A regression | `tests/unit`, `tests/integration`, `tests/security`, or `tests/agent-evaluation/cases.json` |
| An outside system | `mocks/<system>`, its own process. The API image does not copy it |
| Operator explanation | `docs/` |

`01-entry` receives the task. `03-workflows/router` chooses the agents. `02-guardrails/policy` stops a plan that is too long or that hands work to an agent that is not allowed. `03-workflows/engine` runs that plan. `03-workflows/state` keeps the run so a different actor can approve it. `04-loops/runtime` loads each `agent.yaml`. `05-tools/mcp/tool-gateway` is the only plugin caller. `06-jobs/workers` is the only writer of traces. `07-observability` holds the trace, the score, and the sinks. `08-response` is the contract and the projected result. `ui/admin-dashboard` is the console. `ui/packages/controls` is the only place a text box, checkbox, radio, or dropdown is drawn.

| Keep | Replace with the industry |
| --- | --- |
| One `agent.yaml` per specialist, and `runtime.py` | The names, instructions, and output fields |
| The gateway, the approval wait, and the trace shape | The plugin functions and the local stand-in behind them |
| Secrets only in the environment | `MODEL_API_KEY`, SQL, web, n8n, Qdrant, Neo4j |
| Evaluation of status, tools, and duration | Cases in `tests/agent-evaluation/cases.json` |

A read starts on a local stand-in. A write stays simulated until a real host is connected, and it still waits for approval. Documents start from the local index. A relational read stays a `SELECT`. A URL stays on the allowlist. An automation stays simulated until its body posts after approval.

Prove the path with one task the router selects, then read the trace. The trace names the agents and the tool. It does not include the question.

Start from step 1. Finish every step before calling the specialist done. Say which step you are on. Security, denial of service, traceability, and observability are steps, not a follow-up.

1. **Contract.** Write `agentic/04-loops/specialists/<name>-agent/agent.yaml` before any reasoning code. Copy an existing specialist file and change the name, instructions, tools, permissions, and `output_schema`. The folder name is the id. Instructions say what the agent may do and what it must not invent. List only tools you will register in step 2. Set that tool's permission. A write names the tool in `approval.required_for`. When the wait depends on a number, set `amount_gte` on `approval`. Put `platform_trace` and `evaluation` on `observability`.
2. **Plugin.** Add the function, then the `REGISTRY` entry, then the name on the contract. A contract that names an unregistered tool is denied. The tool states who may call it and what it may touch before the code is written. It returns a dict. A failure sets `error` and a safe `explanation`. It does not receive `Platform`.
3. **Security.** Authenticate first, then authorize. `authorized()` stays on every route except guide assets. A valid bearer is not permission to call every tool. The gateway keeps the allowlist, the permission, and the approval wait. Compare the bearer with `secrets.compare_digest`. Production refuses the sample token. Secrets stay in the environment, never in YAML, the dashboard, `localStorage`, a URL, a log, or a trace. `strip_untrusted` runs before the model, a note, or a citation. `query_sql` accepts only a `SELECT`. `fetch_url` and the MCP client allow only a listed host and do not follow redirects. A plugin is loaded from the repo tree, not from an untrusted URL at runtime. No scrape path and no headless-browser caller.
4. **Denial of service and rate limiting.** Keep the limits that already sit in front of the route. 120 requests in 60 seconds is `429` with `Retry-After`. A body over 256 KB is `413`. A task over 4000 characters never reaches the router. A 33rd queued run is `503`. A plan longer than 6 hops is `hop_limit`. The same tool and arguments twice in one run is `loop_detected`. One API process, two workers, a queue of 32. Do not raise these numbers to absorb a flood, and do not add a second API copy for capacity. A flood from many addresses needs a limiter in front of this private API. Successful reads may be reused for 30 seconds. Writes, failures, and the brief are not cached. Responses stay `Cache-Control: no-store`.
5. **Traceability and observability.** Every run uses one run id from the route, through each agent and tool, to the error. The worker in `agentic/06-jobs/workers` is the only writer. A line has `run_id`, `stage`, `agent_id`, `tool`, `status`, and `duration_ms`. A line without a run id is incomplete. The question, the token, and the password are not fields. Log the auth failure, the denied tool, and the request id. An exception is logged as the type plus the run id, then the client gets `The server could not complete the request.` with no stack trace. `platform_trace` and `evaluation` stay on the contract. `otlp` and `log_store` mark an address configured. This build still keeps the record in the process and still omits the question. Learning does not change model weights.
6. **Approval.** A write stays in `approval.required_for`. The first step calls the tool with `approved=False` and pauses. Resume calls it with `approved=True` only after a different actor approves. The agent id as actor is `403`. The console cannot clear the wait or grant a write permission to a read-only agent. Add a `WRITE_EDGES` pair when a write agent must be reached. Do not put that agent in `READ_AGENTS`.
7. **Workflow.** One agent is a `SPECIALISTS` row and a phrase. Several agents are `AGENTS` in `sequences/<workflow>/workflow.py`, a `WORKFLOWS` entry, a phrase above broader phrases, and a delegation rule for each hop. Teach `tool_arguments` the new tool. A write also joins the mock `writes` set, or the mock skips it.
8. **Tests.** Add `cases.json` with `expect_tools`. A paused write expects `needs_approval`. Add a security case for a missing token, a wrong token, a token that must not appear in the trace, a denied tool, and injection text that must not change the tool. Run unit tests, then integration tests, then the evaluation runner. Coverage of `agentic` stays above 90 percent. `pip-audit` must pass.
9. **Host.** Leave mock mode on until the model host exists. Attach one port at a time: model, SQL, web allowlist, n8n, Qdrant, Neo4j. An address means configured, not healthy. Set `PLATFORM_ENV=production` only with a private `PLATFORM_DEV_TOKEN`, and only on the private API. Update `docs/assets/architecture.svg` when the request path changed.

A new Python module opens with a docstring of three parts: a one-line name, `Purpose:`, and `Used by:` with the real caller path. `from __future__ import annotations` goes on the module. Public signatures have type hints. A tool returns a dict. A failure sets `error` to a short code and `explanation` to a safe sentence. It does not raise the raw upstream error to the client.

```python
"""<Tool name>.

Purpose: one sentence. This file holds no secrets.
Used by: agentic/05-tools/mcp/tool-gateway/registry.py.
"""

def read_record(query: str) -> dict:
    return {"explanation": "Answered from the local stand-in.", "sources": []}
```

Use the problem's own tool name. Load that file at the top of `registry.py` with `load_module`, then add the entry before any contract names it:

```python
"read_record": {
    "group": "<Domain>",
    "summary": "What this tool returns.",
    "permission": "<domain>_read",
    "write": False,
    "fn": _domain.read_record,
    "schema": {"query": "string"},
},
```

A tool that calls an outside system sets `mcp_purpose` to a purpose name this problem owns and calls `client.call_tool(purpose, name, arguments)`. Every tool with that purpose uses the server saved for it. A SQL, web, automation, or generic MCP tool sets `endpoint` instead and lives in `ports.py`. A write adds `"write": True` and, when the operator must already have approved, an `approved: bool` parameter. The gateway passes `approved` only if that parameter exists.

```python
AGENTS = ["<first>-agent", "<second>-agent"]
```

```python
READ_AGENTS = {
    "<read>-agent",
}
WRITE_EDGES = {("<read>-agent", "<write>-agent")}
```

```python
"read_record": {"query": task[:180]},
```

```json
{
  "name": "<what the case proves>",
  "task": "<a task the router selects>",
  "workflow": "<workflow>",
  "agent_id": "<name>-agent",
  "expect_tools": ["read_record"]
}
```

| Change | File |
| --- | --- |
| Instructions, tools, permissions, output schema | `agentic/04-loops/specialists/<name>-agent/agent.yaml` |
| Domain behavior | `agentic/05-tools/mcp/<domain>-tools/tools.py` |
| Register a plugin | `REGISTRY` in `agentic/05-tools/mcp/tool-gateway/registry.py` |
| SQL, web, or n8n stand-in | `agentic/05-tools/mcp/tool-gateway/ports.py` |
| Who may hand work to whom | `agentic/02-guardrails/policy/delegation.py` |
| Words that select a workflow | `RULES` in `agentic/03-workflows/router/router.py` |
| Agent order | `agentic/03-workflows/sequences/<name>/workflow.py` |
| Mock arguments for a tool | `tool_arguments` in `agentic/04-loops/llm/mock.py` |
| A case the evaluation run must pass | `tests/agent-evaluation/cases.json` |
| Operator-facing explanation | `docs/` |

The application is ready when each new agent is only a YAML file plus a registered plugin, a denied plugin never runs, a write stays paused until a different actor approves it, the output matches the schema, and the evaluation case fails if the expected plugin is missing.

## Standard

1. Write `agent.yaml` before any reasoning code. LangChain does not own that file. One runtime serves every agent. A new agent does not copy `runtime.py` or `composition.py`, and it does not add a branch inside `Runtime.run`.
2. The gateway is the only caller. A plugin runs only when the contract lists it and the permission is true. The model cannot widen that list. A route does not call a tool, a database, or the model.
3. Every write stays in `approval.required_for`. The console cannot clear that list, grant a write permission to a read-only agent, add a tool URL, or show a secret.
4. A result that misses `output_schema` is rejected. Citations are known source ids. `long_term_memory: optional` may be read. It is not invented.
5. Instruction-like lines are stripped before the model sees them. Traces, logs, error bodies, and cache keys omit the token, the password, and the raw question.
6. The actor who approves a paused write is different from the agent that asked.
7. `PLATFORM_ENV=production` refuses `local-dev-token-not-for-production`. Secrets live in the API environment. A key is not stored in YAML, the dashboard, or `localStorage`.
8. Mock mode is the default. A live model call happens only with `PLATFORM_LLM_MODE=live` and `MODEL_API_KEY`. An address saved for SQL, n8n, Qdrant, Neo4j, Grafana, or OpenTelemetry means configured. The console says "Address saved" or "Mock". It says "Healthy" only after a driver and a health check exist.
9. Ship a test for the route, the plugin, schema validation, a denied plugin, ignored injection text, a blocked write, and any threshold read from the contract.

`agentic/platform/composition.py` is the only composition root. `Platform` builds the gateway, `MemoryService`, `TraceLog`, `EvaluationLog`, `CredentialStore`, `JobRunner`, `RunQueue`, `RunStore`, and one `Runtime`, then loads every specialist. Callers use that object. They do not construct `Runtime` or `ToolGateway` themselves. `reload` reads `agentic/04-loops/specialists/*/agent.yaml` again.

| Step | Folder | Owns |
| --- | --- | --- |
| Perception | `agentic/01-entry` | HTTP body to `AgentRequest` |
| Planning | `agentic/03-workflows` and `agentic/02-guardrails/policy` | Agent order, hop limit, delegation |
| Foundation model | `agentic/04-loops/llm` and `runtime` | The next step inside one agent |
| Memory | `agentic/04-loops/memory` | Session turns, notes, retrieval |
| Action | `agentic/05-tools/mcp/tool-gateway` | The only plugin caller |
| Learning | `agentic/06-jobs` and `agentic/07-observability` | A stage or a score. The worker stores it |
| Communication | `agentic/08-response` | `AgentResponse` for the API and the console |

LangChain `create_agent` is the loop inside one agent. LangGraph, in `agentic/03-workflows/engine` only, runs the `AGENTS` list and pauses for approval. n8n is the write tool `invoke_n8n`. It is not the sequencer. A failed step does not replan. Learning does not change model weights.

Leave on the roadmap until backend enforcement exists: agent versioning, draft and publish, token budgets, provider routing, role lists, tenant isolation, and a durable secret manager. Add another warehouse or model vendor only when a driver and a health check exist. Working model choices are the platform model, an OpenAI-compatible host, and AWS.

## Contracts

A specialist is `agentic/04-loops/specialists/<name>-agent/agent.yaml`. Copy an existing specialist file. The folder name is the id. `load_contract` refuses a file missing `name`, `description`, `instructions`, `tools`, `permissions`, or `output_schema`.

```yaml
name: <name>
description: >
  One sentence a business reader can use.
instructions:
  - What this agent may do.
  - What it must not invent.
tools:
  - <registered-tool>
memory:
  conversation_state: true
  long_term_memory: optional
permissions:
  <permission>: true
output_schema:
  type: object
  properties:
    explanation:
      type: string
  required:
    - explanation
approval:
  required_for: []
rag:
  retrieve: false
  capture: false
  vector: local
  embedding: local
model:
  provider: platform
observability:
  - platform_trace
  - evaluation
evaluation:
  expect_tools:
    - <registered-tool>
```

List only names in `registry.py`. Each tool's `permission` is true on this contract. Name the permission for the problem, such as `<domain>_read` or `<domain>_write`. A write tool also appears in `approval.required_for`. A write with `amount_gte` waits only when the arguments include `amount` and the amount meets that number. A write without that field waits on every call. `output_schema` is JSON Schema. A miss is `schema_invalid`. Required fields are fields the tools return. A citation field is `sources`.

`agentic/platform/contract_edit.py` is the only writer the console uses. A save may change `instructions`, `tools`, `memory`, `approval`, `observability`, `rag`, and `model`. Permissions and `output_schema` stay in the file, so a save cannot grant a write permission or rewrite the schema. It refuses an unknown tool, a tool whose permission is false, a tool whose host is not ready, clearing approval on a write, an observability id outside `platform_trace`, `evaluation`, `otlp`, `log_store`, `grafana`, a vector outside `local`, `qdrant`, `aws`, an embedding outside `local`, `openai`, `aws`, a model provider outside `platform`, `openai`, `aws`, and a cadence outside `off`, `each_run`, `hourly`, `daily`. Capture false forces cadence `off`. Contract notes are at most 40 strings of 2000 characters. After the write, `platform.reload()` loads the file. Add one `expect_tools` case before calling the agent done.

## Runtime

`Runtime.run` is the only loop.

1. `POST /agent-runs` does not send permissions. The gateway uses the contract flags. Runtime copies those flags, then applies `request.permissions` only for keys the contract already declares.
2. Trace `load_instructions` with the agent id and tool names.
3. Load notes when `long_term_memory` is `optional`. Retrieve when `rag.retrieve` is set. Capture a redacted note when `rag.capture` is set. Remember the stripped user turn.
4. Build one `StructuredTool` per contract tool that exists in the gateway. The closure calls `gateway.call`. The first pass passes `approved=False`.
5. Use `MockChatModel` unless `PLATFORM_LLM_MODE=live` and `MODEL_API_KEY` are both set. Live mode uses `ChatOpenAI` with `MODEL_NAME` (default `gpt-4.1-mini`).
6. Invoke with the stripped task and the contract instructions as the system prompt.
7. Project onto `output_schema`. Keep citations `accept_citations` accepts. A schema miss adds `schema_invalid`.
8. Record evaluation when listed: run id, agent, status, tool names, duration in milliseconds. Trace `persist`. Trace `export` when `otlp` or `log_store` is selected.

Status is `needs_approval` when any call returned that, `failed` on an error code or a schema miss, otherwise `completed`. The response carries `result`, `citations`, `tool_calls`, `errors`, `requires_approval`, `pending_tool`, `agent_id`, and `request_id`. Tool calls on the response are name and status. `platform_trace` gates the other stages.

`agentic/04-loops/llm/mock.py` calls each allowlisted tool once. A write runs only when the task matches a phrase that tool is registered under. Arguments come from the task text. A new tool gets a branch in `tool_arguments`. A new write joins the `writes` set and a phrase, or the mock skips it.

Live mode copies `model`, `total_tokens`, and `cost` when the provider sends them. The key stays in the environment. `MODEL_BASE_URL` points at the organization model host when that host exists. Leave mock mode until then. A contract provider of `openai` or `aws` records the choice. This runtime still uses the platform model until that path is implemented. Do not add a vendor client the process does not call.

`agentic/08-response/project/output.py` fills schema properties from tool results, then from arguments. Notes fill `explanation` only when no tool did. `public_result` returns only declared keys. Argument models use `string`, `number`, `array`, `object`, and `boolean`. The API result is that projection, not the model transcript.

## Gateway

`gateway.py` is the only plugin caller. Add the function in `agentic/05-tools/mcp/<domain>-tools/tools.py`. A shared port, such as a read-only query, an allowlisted fetch, an automation call, a document search, or a dataset summary, lives in `ports.py`. Register it in `REGISTRY` before a contract names it.

- `permission` is a flag this problem defines. The contract sets that same flag true. Reuse a flag when the capability is the same.
- `write` is true only when the tool changes state outside a read. Every such tool waits for approval.
- `fn` and `schema` (`string`, `number`, `array`). An outside-system tool sets `mcp_purpose` to the purpose this problem owns. A host tool sets `endpoint`.
- `group`, `summary`, and `source` are what `GET /tools` shows.

A tool does not receive `Platform`. It accepts `approved` only when it must know an operator already approved. The gateway passes that flag only if the parameter exists.

`ToolGateway.call` checks, in order: allowlist and registry (`tool_denied`), permission (`tool_denied`), host ready (`connection_required`; a tool with neither endpoint nor purpose runs in process), repeated name plus argument digest for this run and agent (`loop_detected`), then approval. A write in `required_for` returns `needs_approval` and `submitted: false` until `approved` is true. `amount_gte` applies only when the arguments include `amount` and the amount meets it. A successful read may be reused. A write, a failure, and the brief are not stored. A throw is `upstream_unavailable`. A non-dict result is `upstream_rejected`. The client receives no stack trace.

The read cache keeps a successful read for `PLATFORM_READ_CACHE_TTL` seconds (default 30). The key is the tool name, a hash of the address, and a hash of the arguments. `PLATFORM_READ_CACHE_SIZE` defaults to 128. Set it to 0 to turn the cache off. Stored payloads omit `task`, `question`, and `text`.

A relational tool accepts a statement that starts with `SELECT` and rejects every other statement. A configured connection still returns the local fixture until that body is replaced. Keep the `SELECT` limit. A URL tool allows a host on the allowlist and does not follow redirects. A dataset tool reads only the fixture path the problem registered. A code string containing `exec`, `import `, `open(`, or `__` is `rejected`. A document tool answers from the local index and says so when nothing matches. Snippets pass through `strip_untrusted`.

A write returns a simulated result and `submitted: false` until its body is deliberately replaced. Replace that body only after approval. Leave `submitted` false until the post succeeds. Turn on one port at a time: model, relational store, web allowlist, automation, vector, graph. In this platform only the model port calls out until that body is replaced. Add a row to `docs/tool-contracts.md` when a tool is new.

An outside system is a separate process. The API image does not copy it. Records live on that process. Its URL is an environment variable or a saved connector, and it covers a purpose until a saved server replaces that purpose. The token is a bearer header, never placed in the URL, and never returned. The client keeps one session and does not follow redirects. A silent server fails the run. A list-tools call does not execute those remote tools inside the loop. A new remote capability is a registered plugin, then a line on a contract.

## Workflows

A workflow file is only `AGENTS = ["first-agent", "second-agent"]` in `agentic/03-workflows/sequences/<name>/workflow.py`. Names are specialist folder names. `router.py` decides in this order:

1. An `agents` list wins. The workflow name is ignored. The run is called `agent-chain`.
2. A `workflow` name selects `WORKFLOWS` or `SPECIALISTS`. An unknown name yields an empty list.
3. Otherwise the lowercased task matches `RULES`. The first row wins. Put narrower phrases above broader ones.
4. No match is `ambiguous`, then `ambiguous_intent`.

Register the name in `WORKFLOWS` with `_agents("<name>")`. Phrases stay in `RULES`. Single specialists have phrases and no workflow file. `GET /platform` and the Workflows tab read the live list.

`planner.py` then refuses an empty list (`ambiguous_intent`), more than `MAX_HOPS` (6) (`hop_limit`), an unloaded name (`unknown_agent`), and a refused handoff (`delegation_denied`). `can_delegate` allows a hop when both agents are in `READ_AGENTS`. A hop that includes a write agent must be a pair in `WRITE_EDGES`. An agent cannot hand off to itself. Add the new read-only agent to `READ_AGENTS`. Add a pair when a write agent must be reached. Do not put a write agent in `READ_AGENTS`.

The graph has one `step` node. Cancel is read before each step. A timeout (`PLATFORM_TIMEOUT_SECONDS`, default 8) uses one thread and ends the graph, as does a failed agent. `needs_approval` pauses. Resume calls the pending tool with `approved=True`. The first step does not approve itself. The remaining agents then run.

`store.py` keeps runs, approvals, and an audit list in this process. `open_approval` stores the tool, the arguments, and their hash. `decide` accepts `approve` or `reject` while `pending`. The actor must differ from `agent_id` (`self_approval`, `403`). A missing, decided, or stale approval is `409`. Cancel applies only while `running` is true and marks the run failed. A second API process would not see this queue.

## Guardrails

`auth.py` compares `Authorization` to `Bearer` plus `PLATFORM_DEV_TOKEN` with `secrets.compare_digest`, after a length check. Every route except `GET /guide/assets/{name}` calls `authorized()`. A missing or wrong token is `401` with `Unauthorized`. The token is a header, not a query parameter, path segment, log field, or trace field. `assert_runtime_config` refuses production while the sample token is set. A local default lives only in `config.py`. A valid bearer is not permission to call every tool.

Limits run before the body is trusted. `PLATFORM_RATE_LIMIT` requests in `PLATFORM_RATE_WINDOW` seconds is `429` with `Retry-After` (120 and 60). The key is the caller address and a hash of the bearer. A body over `PLATFORM_MAX_BODY` bytes is `413` (262144). A task over 4000 characters never reaches the router. Responses send `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: no-referrer`, and `Cache-Control: no-store`. A flood from many addresses needs a limiter in front of this private API.

`strip_untrusted` runs before the model, a note, a citation snippet, or document search. Lines matching `ignore previous instructions`, `you are now`, or `system:` are removed. A value after `api_key`, `token`, `password`, or `secret` becomes `[redacted]`. `summarize` keeps 240 characters. Do not add a second stripper.

Connectors save URL, connection string, username, and password in `data/connections.json`, which is not committed. The API returns `password_set` and omits the password. A blank password keeps the stored one. The model key is reported only as set or not set. `CredentialStore` is an in-memory slot, not that file, and it does not hold the process token. A durable secret manager is on the roadmap. A new secret uses this file, is omitted from every response, and is never logged. MCP rows are title, purpose, URL, username, and password. `POST /settings/mcp-servers/{id}/check` sends initialize and records whether the server answered. Redirects are not followed. The purpose name belongs to the problem. Tools that share a purpose share that server.

## Memory

Conversation state, notes, and retrieval stay three stores. Tool results stay on the run. `MemoryService` is constructed once by `Platform`.

`long_term_memory: optional` loads notes for this `user_id` whose text shares a token longer than 3 characters with the task. `rag.retrieve` reads the index. `rag.capture` stores a redacted note of at most 240 characters. Both default false. Turn retrieval on only for a specialist whose job is to read the index. An agent that reads does not also capture unless the contract says so. The user turn is stripped, trimmed to 240 on the way in and 500 on the session list. `POST /memory/notes` stores at most 500 characters. Notes disappear when the process stops. `rag.retrieve` false still says "This agent does not read from RAG". Do not turn retrieval on to hide that line.

| Field | Allowed | Default |
| --- | --- | --- |
| `retrieve`, `capture` | true or false | false |
| `vector` | `local`, `qdrant`, `aws` | `local` |
| `embedding` | `local`, `openai`, `aws` | `local` |
| `cadence` | `off`, `each_run`, `hourly`, `daily` | `off`, or `each_run` when capture is true |
| `model.provider` | `platform`, `openai`, `aws` | `platform` |

An unknown value fails the load and names the file. `QDRANT_URL`, `AWS_VECTOR_INDEX`, `EMBEDDING_MODEL`, and `AWS_EMBEDDING_MODEL` mark a host configured. This build still answers from the local index and still hashes text locally. It does not call AWS. Replace `VectorPort.query` or `GraphPort.query` when a real host should answer, and leave the local corpus until that query returns.

`accept_citations` keeps a source id in `KNOWN_SOURCES` or a prefix the corpus allows. A new problem adds its prefix there. The runtime attaches `sources` only when the schema has that property. Add a document in `corpus.py` with `source_id`, location, and text. A dataset fixture is a file the problem registers. A new corpus keeps the same citation check.

## Observability and jobs

`agentic/06-jobs/workers/writer.py` is the only writer of the in-memory trace, the evaluation row, the feedback, and `data/traces.log`. The run response waits until `flush` finishes. A line may contain `run_id`, `stage`, `agent_id`, `tool`, `status`, and `duration_ms`. A line without a run id is incomplete. The question is not a field.

| Stage | When | Fields |
| --- | --- | --- |
| `load_instructions` | `platform_trace` | Agent id and contract tool names |
| `tool` | `platform_trace` | Agent id, tool name, status |
| `reason` | `platform_trace` | Agent id |
| `persist` | `platform_trace` | Agent id and response status |
| `export` | `otlp` or `log_store` selected | Whether that address is configured |

Evaluation stores run id, agent id, status, tool names, and duration in milliseconds. Feedback stores helpfulness and a note of at most 240 characters. `TraceLog.export` is the JSON for one run. When the process stops, traces, scores, runs, approvals, and notes are gone. `traces.log` remains on `PLATFORM_DATA_DIR`.

| Sink | Variable | This build |
| --- | --- | --- |
| `platform_trace` | none | In this process. Default on |
| `evaluation` | none | Score in this process. Default on |
| `otlp` | `OTEL_EXPORTER_OTLP_ENDPOINT` | Marks the collector configured. The trace stays here |
| `log_store` | `OBSERVABILITY_LOG_URL` | Marks the store configured. The worker does not POST yet |
| `grafana` | `GRAFANA_URL` | Saves the address. The password is not on the trace. Nothing pushes a line to Grafana or Loki |

Unknown sink ids fail the YAML load. Ship to a collector by extending the worker, and still omit the question and secrets.

`runs.py` accepts work onto 2 workers and a queue of 32. A full queue is `503` with `The work queue is full.` Agents inside one run stay in order. The observability writer is a different thread. `JobRunner` runs a named check in this process. It is not a second service. A new background behavior joins one of these two. It does not start a daemon from a route or an agent.

## API

`agentic/01-entry/app.py` calls `assert_runtime_config`, builds one `Platform`, and mounts the route groups. Interactive docs are `/docs`. The written list is `docs/api.md`. Bodies live in `bodies.py`. Field descriptions are what `/docs` shows.

| Module | Owns |
| --- | --- |
| `routes/runs.py` | Start, stream, get, cancel, feedback |
| `routes/approvals.py` | List and decide |
| `routes/catalog.py` | Agents, tools, plugins, health, platform |
| `routes/settings.py` | Platform settings, contracts, connections |
| `routes/mcp.py` | MCP server create, update, check, delete |
| `routes/guide.py` | Guide pages and assets |

`RunBody` is `task` (required), `user_id` (default `local-development-user`), `session_id`, `workflow`, and `agents`. A public run has `request_id`, `session_id`, `workflow`, `status`, `error`, `error_code`, `agents`, and `approval_id` when a write is waiting. Each agent entry has `agent_id`, `status`, `result`, `tool_calls`, and `requires_approval`. Status values are `completed`, `needs_approval`, `failed`, and `not_implemented`. Stream events are `activity`, `plan`, `step`, `done`, and `error`. The whisper is `Working in the background. This page will update when the run finishes.` Get is `404` when this process lacks the id. There is no run list. Cancel is `409` when the run is not running. `DecisionBody` is `approve` or `reject`. `actor` defaults to `local-operator`.

`ContractBody` is partial. Omitted fields stay. `422` when the change would grant a write the agent lacks, clear approval, name an unknown tool, or attach a tool whose host is not ready. `404` when the agent id is unknown. `PluginBody` is `{ "agents": ["<name>-agent"] }`. `EndpointBody` is url, connection string, username, and password. `NoteBody` text is at most 500 characters. `GET /settings/platform` returns mode, timeout, hop limit, whether the model key is set, the data directory, and the catalogs. It omits the key. An unexpected failure is `500` with `The server could not complete the request.`

To add a route: add a body class, add the route in the matching module, call `authorized()`, call `platform`, name the console caller in the docstring, add the path to `ui/admin-dashboard/vite.config.ts` and `infrastructure/nginx.conf`, add `tests/integration/test_api.py`, and add a row to `docs/api.md`. Nginx already proxies `/health`, `/platform`, `/agent-runs`, `/approvals`, `/plugins`, `/settings`, `/guide`, and `/memory`. A new agent is YAML. A new workflow is a sequence. Neither is a new body class.

## Host

New configuration goes through `agentic/platform/config.py`. A route, the runtime, the gateway, or a workflow does not embed its own number or secret.

| Name | Default | Meaning |
| --- | --- | --- |
| `PLATFORM_ENV` | `development` | `production` refuses the sample token |
| `PLATFORM_DEV_TOKEN` | the sample token | Bearer secret. No production default |
| `PLATFORM_LLM_MODE` | `mock` | `live` plus `MODEL_API_KEY` calls the model |
| `PLATFORM_DATA_DIR` | `data/` | `traces.log` and `connections.json` |
| `PLATFORM_TIMEOUT_SECONDS` | `8` | One agent step |
| `PLATFORM_RATE_LIMIT` / `PLATFORM_RATE_WINDOW` | 120 / 60 | Then `429` |
| `PLATFORM_MAX_BODY` | 262144 | Then `413` |
| `PLATFORM_READ_CACHE_TTL` / `PLATFORM_READ_CACHE_SIZE` | 30 / 128 | Successful reads. Size 0 turns the cache off |

`health` returns status, mode, and the agent count. Secrets belong on the API only: the bearer, the model key, an outside-system token, and connection strings. The console image holds none.

`./start-local.sh` binds the API to `127.0.0.1:8090` and the console to `127.0.0.1:5173`. The interpreter is `.venv`. `docker compose up -d --build` publishes the console on 5173, or `CONSOLE_PORT`. The API has no host port. `./infrastructure/smoke.sh` checks the agents, the seven steps, security, observability, the edit paths, and one run that calls the tool the case expects. `PLATFORM_DATA_DIR` in compose is the `agentic-data` volume. One API process holds approvals, traces, notes, and evaluation. Add a shared store before a second copy.

The root Compose file does not start Grafana, Langfuse, or n8n. On a machine with about 16 GB of RAM, `infrastructure/dependencies/docker-compose.yml` starts Grafana, Prometheus, a collector, Loki, Langfuse, MLflow, n8n, Qdrant, Neo4j, and Postgres. Kibana and Elasticsearch stay out. `check.sh` proves logins. It does not send a question or post a webhook. Lab passwords stay in `.env`. `PLATFORM_ENV` on that stack stays `development`. The API still keeps its trace in process. Azure, when added, is an internal API, an external console, a volume only when data must survive restart, and secrets in Key Vault references.

## Console

`ui/admin-dashboard` holds no secrets. `api.ts` attaches the bearer. Chat uses `stream.ts`. Screens import `TextField`, `TextArea`, `NumberField`, `Checkbox`, `RadioGroup`, `Select`, and `GearButton` from `@agentic/controls`. A screen does not write its own `input`, `select`, or `textarea`.

| Screen | Operator uses it to |
| --- | --- |
| Dashboard | Loaded agents, pending approvals, runs from this session |
| Agent chat | Brief one agent. The citation stays closed until opened |
| Agent library | Open a contract in its chat |
| Run history | Runs from this browser session |
| Approvals | Approve or reject a paused write |
| Configuration | Prompt, knowledge base, ingestion, SQL access, observability, notes |
| Plugins | Connect a registered tool to agents that already have the permission |
| Docs | `GET /guide` |

The gear opens instructions, memory, observability, retrieval, tools, and workflows. Generate YAML writes `PUT /settings/agents/{id}`. The screen does not offer a control that grants a write permission, clears approval, adds a tool URL, shows a password or model key, or marks a host Healthy. A password field starts empty. The label says the password is set. Settings are not stored in `localStorage`. The approval actor does not default to the agent id. Mock mode is labeled on chat. Loading in the header is a skeleton. An error toasts and leaves the screen in place. Statuses use a shared label. Copy is US English. Nav, tabs, titles, and primary buttons are Title Case. Helpers are sentence case.

Editable fields are instructions, tools, memory, approval amount, sinks, RAG retrieve and capture, vector, embedding, cadence, and model provider. Choices come from `GET /settings/platform`. Databricks, Snowflake, Anthropic, and Gemini are not offered. Connectors edit n8n, Grafana, OpenTelemetry, Qdrant, AWS, the model host, SQL, and Neo4j. MCP servers have their own form and a check action.

## Tests

Tests use `unittest`, build `Platform()` the way `tests/unit/test_runtime.py` does, and assert the public run. They do not assert private steps inside `Runtime.run`. Module docstrings use `Purpose:` and `Used by:`.

| Kind | Path |
| --- | --- |
| Workflow, tool choice, schema, denial | `tests/unit/test_runtime.py` |
| HTTP status, auth, bodies | `tests/integration/test_api.py` |
| Missing token, wrong token, token absent from the trace, denied tool | `tests/security/test_security.py` |
| A task an operator would type | `tests/agent-evaluation/cases.json` |

`tests/agent-evaluation/run.py` starts the outside-system stand-in when a case needs one, builds `Platform()`, and fails when the workflow differs, the named agent's status differs, or an `expect_tools` entry is missing. `expect_status` defaults to `completed`. A paused write sets `needs_approval`. The agent id is the folder name.

```bash
.venv/bin/python -m unittest tests.unit.test_runtime tests.integration.test_api tests.security.test_security
.venv/bin/python tests/agent-evaluation/run.py
.venv/bin/python -m ruff check agentic tests
npm test --prefix ui/admin-dashboard
```

Unit tests run first. An integration failure fails the pipeline too. Line coverage of the `agentic` tree stays above 90 percent. Do not lower the gate or exclude a module. Ruff line length is 100. `pip-audit` fails on an open advisory. Dependencies stay pinned. A change is done when a new agent is YAML plus a registered plugin, a denied plugin never runs, a write stays paused until a different actor approves it, the output matches the schema, and the evaluation case fails if the expected plugin is missing.
