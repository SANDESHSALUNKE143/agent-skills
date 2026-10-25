---
name: senior-python
description: >-
  Writes and reviews Python and FastAPI in the numbered agentic stages, including
  authentication, logging, tests, coverage, and dependencies. Use when changing
  Python, APIs, scripts, refactors, or backend bugs.
---

# Senior Python

Follow the `agentic/` tree as it is written. Do not impose one class per file on modules that are already groups of functions or small related types. Match the test runner and linter already in the repository. Do not switch a repository off `unittest` or Ruff when that is what it uses, and do not switch a repository off pytest when that is what it uses.

## Layout

- Numbered stages stay the packages they already are. `agentic/bootstrap.py` binds `01-entry` through `08-response` to `entry`, `guardrails`, `workflows`, `loops`, `tools`, `jobs`, `observability`, and `response`. New code goes in the stage that already owns that concern.
- A path with a hyphen that is not a package is loaded with `agentic.platform.loader.load_module`, the same way routes load `bodies.py` and the gateway loads `registry.py`. Do not add a second import mechanism.
- Put `from __future__ import annotations` on new modules. Public signatures have type hints. Private helpers start with `_`. Ruff stays at line length 100 with `E4`, `E7`, `E9`, `F`, and `I`. Python 3.11 or newer. CI uses 3.12.
- No configuration literals in core modules. Tokens, URLs, limits, timeouts, mode flags, and feature switches are read from environment variables through `agentic/platform/config.py` (`setting`, and a new function beside the ones already there). A route, the runtime, the gateway, or a workflow does not embed its own number or secret. A local default, when the process must boot with the variable unset, lives only in that settings module. A secret has no production default. `PLATFORM_ENV=production` still refuses the sample token.

## What a file is

- A module that holds state is one class: `Runtime`, `ToolGateway`, `MemoryService`, `RunStore`, `TraceLog`, `CredentialStore`. `Platform` in `agentic/platform/composition.py` is the only composition root. It constructs those objects and passes them in. Callers do not construct `Runtime` or `ToolGateway` themselves.
- Stateless behavior stays functions. Routes export `mount(app, platform)`. Tools, guards, planners, and transforms stay functions. Do not wrap them in a class.
- Related types stay in the module that already groups them: request bodies in `agentic/01-entry/bodies.py`, `AgentRequest` and `AgentContract` in `agentic/08-response/contracts/__init__.py`, `VectorPort` and `GraphPort` together, private exceptions beside the client that raises them. A second stateful service does not join an existing service module.
- Production modules top out around 250 lines (`client.py`, `mcp_servers.py`, `catalog.py`). Split before a module takes on a second concern or grows well past that. Do not split a cohesive module only to make more files. `runtime.py` is the one reasoning loop. A new agent does not copy it or copy `composition.py`.

## Documentation

- Every module opens with a docstring of three parts: a one-line name, `Purpose:`, and `Used by:` with the real caller path. That docstring is the class documentation when the module has one class. Do not add a second docstring that repeats it.
- A public method or function whose name and signature already say what it does does not get a comment. Document a non-obvious error or a boundary in the `Purpose` line.

## Design

- No duplicated rule. The second copy becomes a function or a field on the contract. The module docstrings already say when a file must not be copied.
- One reason to change per module. The gateway does not plan. The runtime does not own HTTP. A route module does not call tools itself.
- A new agent is a specialist folder and `agent.yaml`. A new tool is a function plus a registry entry. A new route group is a new `mount` module. Do not add an agent-specific branch inside `Runtime.run`.
- A stand-in honors the same call as the live object (`MockChatModel`, the local corpus behind `VectorPort`).
- Tools and ports stay narrow. A tool does not receive `Platform`. `Runtime` depends on the gateway, memory, traces, and evaluation it was given, so a test can pass substitutes.
- Explicit errors. No bare `except`. Validate at the trust boundary (task size, contract schema, tool permission). No secrets in code, logs, or fixtures. No new dependency unless the task needs it.

## Dependencies

An external library is added only when the task needs it, pinned in `pyproject.toml`. Before it is added or upgraded, audit it. The pipeline runs `pip-audit` and fails when any installed library has an open known vulnerability. Do not keep a vulnerable version, and do not add a package whose current release has an unfixed advisory. A transitive dependency with an open advisory is upgraded or removed. Add `pip-audit` to the same CI job.

## OWASP Top 10

Apply this to every FastAPI and Python change:

- Broken access control: authenticate first, then authorize. A valid bearer is not access to every route or tool.
- Cryptographic failures: secrets encrypted at rest, compared with `secrets.compare_digest`, never invented cryptography.
- Injection: bound parameters for any relational query. No string-built SQL, shell, or path from request input. Contract output still goes through schema validation.
- Insecure design: a new route or tool states who may call it and what it may touch before the code is written.
- Security misconfiguration: settings come from the environment. Production refuses the sample token. Debug and mock modes are not the production default.
- Vulnerable and outdated components: the `pip-audit` gate above.
- Identification and authentication failures: bearer on every route except guide assets. Failed auth is 401. Tokens are not in traces, logs, URLs, or error bodies.
- Software and data integrity failures: dependencies are pinned. A contract or plugin is loaded from the repo tree, not from an untrusted URL at runtime.
- Security logging and monitoring failures: log the auth failure, the denied tool, and the request id. Do not log the token, the password, or the raw question.
- Server-side request forgery: a user-supplied URL is not fetched unless it matches an allowlisted connector. Redirects are not followed, matching the MCP client.

## FastAPI

New HTTP behavior is a FastAPI route in the matching `mount` module under `agentic/01-entry`. The route checks the body, then calls `platform` or the gateway. It does not call a tool or a database directly.

- Authentication stays `agentic/02-guardrails/auth/auth.py`: `Authorization: Bearer`, compared with `secrets.compare_digest`. Every route except guide assets calls `authorized()`. `PLATFORM_ENV=production` still refuses the sample token in `agentic/platform/config.py`.
- Authorization is a second check after the bearer succeeds. A valid token is not permission to call every tool. The gateway keeps the contract tool list, the permission flags, and the approval wait. A route does not grow its own copy of that decision.
- Credentials and tokens are encrypted at rest. The in-memory `CredentialStore` in `agentic/02-guardrails/connections/credentials.py` is not a place to leave a raw token. A saved connector secret is encrypted before it is written. The bearer is a header, never a query parameter or a URL path.
- Traces, logs, error bodies, and cache keys do not contain a token, a password, or the raw question. `agentic/02-guardrails/policy/redact.py` still strips token-shaped text before the model or a citation sees it. A leaked trace must not be enough to replay a token.
- The API is not open to browserless scraping. A headless client with no valid bearer gets 401, the same as any other caller. Do not add a scrape path, and do not call this API through a headless browser. Callers use HTTP with the `Authorization` header. Rate limiting in `agentic/02-guardrails/limits` stays on the app.

## Logging and traceability

- Every request and every run writes a structured log. The fields are the run id, the stage, the agent, the tool name, the status, and the duration, matching `agentic/07-observability/trace/trace.py`. A log line without a run id is incomplete. The same run id groups the route, the runtime, the tool calls, and the error so one failure can be followed from the first line to the last.
- Exceptions are handled at the boundary. A route or the runtime logs the exception type and the run id, then returns a safe error. A bare `except` is not used. An exception is not swallowed, and the client does not receive a stack trace, a token, or the raw question.
- Logs are shipped to the configured log server, not only kept in the process. The address is an environment variable (`OBSERVABILITY_LOG_URL` or `OTEL_EXPORTER_OTLP_ENDPOINT` in `agentic/07-observability/sinks/sinks.py`). When that address is set, the worker sends the trace there. The record still omits the raw question and secrets. A local `traces.log` is the fallback only while the address is unset.

## Data

- A database read uses a cache. The cache key is the query identity, not a token. A write invalidates the entries it changes. Responses that carry a secret keep `Cache-Control: no-store`. Stream routes keep `Cache-Control: no-cache`.
- This process has no relational database today. Do not add one unless the task needs it. When the store is relational, schema changes are Alembic revisions only. The pipeline runs the migration check, and a model change without a revision fails the build.

## Tests and the pipeline

- Use `unittest`, in `tests/unit`, `tests/integration`, and `tests/security`, with the same `Purpose` / `Used by` module docstring. `tests/unit/test_runtime.py` builds `Platform()` and asserts workflow, tool choice, schema, and denial. New tests follow that. They do not assert private steps. Security tests cover a missing token, a wrong token, a token that must not appear in the trace, and a denied tool.
- The pipeline runs unit tests first. A unit failure fails the pipeline. Integration tests run next, and an integration failure fails the pipeline too. Neither is skipped to make the other look green.
- Overall line coverage of the `agentic` tree must be greater than 90 percent. The pipeline fails under that line. Use `coverage` with the existing `unittest` modules (`coverage run -m unittest ...`, then `coverage report --fail-under=90`). Do not lower the gate and do not exclude a module to pass it.
- Report the commands that ran. CI runs `ruff check .` and `python -m unittest tests.unit.test_runtime tests.integration.test_api tests.security.test_security`. Add `pip-audit` and the coverage fail-under to that job. `pip-audit` fails the pipeline on any open vulnerability.

## Delivery

Delivery follows the shared pull requests: feature into `dev`, `dev` into `uat`, `uat` into `main`. The API workflow deploys only after that merge, and only that environment. Merge into `main` deploys prod. It does not deploy prod from `dev`.
