# Claude Agent Library

A collection of Claude subagent prompt files. Each agent is a specialized Claude instance — scoped, opinionated, and built to produce consistent output for a specific engineering domain.

This is a **prompt library**, not a software project. The files here are instructions that govern Claude's behavior at inference time, not code that executes.

---

## What These Agents Are

Each `.md` file in this folder defines a Claude subagent:

- **YAML frontmatter** declares the agent's name, description, tools, and model
- **The body** is the system prompt — behavioral rules, output templates, hard constraints, and scope boundaries
- **The `description` field** is used for routing — it tells an orchestrating agent when to invoke this subagent and when not to

Agents are designed to be invoked from Claude Code using the `Agent` tool, or referenced directly as subagent definitions in multi-agent workflows.

---

## Agent Index

### `backend-engineer.md`

**Invoke for:** Server-side work — REST/GraphQL API design, database schema and query optimization, auth (JWT, OAuth2, sessions), authorization (RBAC, ABAC), background jobs, caching, distributed transactions, service architecture, security hardening, observability. Also for backend code review and architectural decisions.

**Stack:** Node.js/TypeScript, Python, Go

**Do not invoke for:** Frontend, infrastructure-as-code, ML pipelines, mobile native

---

### `frontend-engineer.md`

**Invoke for:** Frontend work — UI component design and implementation, responsive layouts, design system integration (Tailwind, shadcn/ui, MUI, Chakra, etc.), forms and validation, accessibility (WCAG 2.1 AA), state management, animation, cross-browser compatibility, and code-level debugging of visual or behavioral bugs.

**Stack:** Framework-agnostic — React, Vue, Svelte, vanilla

**Do not invoke for:** Backend logic, API design, database schema, infrastructure, build tooling configuration

---

## How to Use an Agent

### Direct invocation in Claude Code

Reference the agent file when starting a task. The agent's YAML frontmatter and body become its system prompt.

```
Use the frontend-engineer agent to build a data table component with sorting and pagination.
```

### Chaining agents

For full-stack tasks, invoke agents sequentially — backend agent for API contract design, then frontend agent for the component that consumes it. Pass the relevant output from one as context to the next.

```
1. Backend agent → define the API response shape for /users
2. Frontend agent → build the UserList component against that contract
```

### When to use which agent

| Task | Agent |
|---|---|
| API endpoint design | backend-engineer |
| Database schema or query | backend-engineer |
| Auth / session logic | backend-engineer |
| UI component | frontend-engineer |
| Responsive layout | frontend-engineer |
| Accessibility audit | frontend-engineer |
| Form with validation | frontend-engineer |
| Full-stack feature | backend first → frontend second |

---

## Agent Design Principles

Every agent in this library is built against the same standard:

- **Scoped** — each agent has an explicit in-domain and out-of-domain definition, and defined behavior at the boundary
- **Opinionated** — agents have hard rules with pushback behavior, not just suggestions
- **Template-driven** — output structure is deterministic for the top 2–3 task types
- **Honest** — agents flag uncertainty rather than invent API signatures, token names, or config keys
- **State-complete** — output covers failure cases, not just the happy path

---

## Adding a New Agent

New agent files must meet the standards defined in [`CLAUDE.md`](./CLAUDE.md). Before writing, review:

- The **Required Sections Checklist** — every section must be present
- The **Audit Rubric** — run a self-audit before considering the file complete
- The **Five Recurring PE Failure Modes** — check your file against all five

The quality bar: you can predict what the agent will say to 5 different inputs without running it.

---

## Test Suite

Behavioral test specifications for each agent live in the [`tests/`](./tests/) folder. Each test case defines an input, expected behavior, pass signals, and fail signals.

| File | Agent | Tests |
|---|---|---|
| [`tests/backend-engineer-tests.md`](./tests/backend-engineer-tests.md) | backend-engineer | 12 |
| [`tests/frontend-engineer-tests.md`](./tests/frontend-engineer-tests.md) | frontend-engineer | 13 |

See [`tests/README.md`](./tests/README.md) for evaluation methodology.

---

## File Naming

```
{domain}-{role}.md       → backend-engineer.md, frontend-engineer.md
{function}-agent.md      → code-reviewer.md, api-designer.md
{specialization}.md      → auth-specialist.md, sql-optimizer.md
```

The `description` field in YAML frontmatter must start with `INVOKE for` or `USE when` and include explicit anti-triggers where the scope is narrow.
