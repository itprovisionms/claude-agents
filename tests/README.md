# Agent Test Suite

Behavioral test specifications for agents in this library. These are not runnable code — they are input/output contracts that verify an agent behaves correctly at inference time.

---

## What These Tests Check

Each test case verifies one behavioral decision the agent must make correctly. Tests are organized into categories:

| Category | What it verifies |
|---|---|
| **Template routing** | The agent selects and applies the correct output template for a given task type |
| **Scope boundary** | The agent correctly identifies out-of-domain requests and hands off cleanly |
| **Pushback** | The agent enforces a hard rule with the correct protocol (object once → show alternative → implement with annotation) |
| **Epistemic honesty** | The agent flags uncertainty rather than inventing API signatures, token names, or config keys |
| **Prototype handling** | The agent reduces ceremony for quick examples but does not silently drop safety or correctness requirements |
| **Question cap** | The agent asks at most 2–3 clarifying questions for ambiguous requests and includes assumption fallback |
| **Iterative changes** | The agent shows diffs rather than full rewrites for incremental modifications (frontend only) |

---

## How to Evaluate

### Manual evaluation

1. Send the **Input** to the agent verbatim (no additional context unless specified)
2. Check the response against **Pass signals** — these must all be present
3. Check for **Fail signals** — any of these present means the test fails
4. Record result: Pass / Fail / Partial

### Evaluator agent

You can invoke a Claude instance as an evaluator. Give it:
- The agent's full system prompt (the `.md` file body)
- The test input
- The pass/fail criteria

Ask it to predict the agent's response and evaluate against the criteria. This is faster than live invocation but is a simulation, not a ground truth test.

---

## Test Case Format

```
### T[ID] — [Short name]
**Category:** [category]
**Input:** [exact user message — send verbatim]
**Expected behavior:** [what the agent should do, not word-for-word output]
**Pass signals:** [specific things that must appear in the response]
**Fail signals:** [specific things that indicate failure]
```

---

## Files

- [`backend-engineer-tests.md`](./backend-engineer-tests.md) — 12 test cases for `backend-engineer.md`
- [`frontend-engineer-tests.md`](./frontend-engineer-tests.md) — 13 test cases for `frontend-engineer.md`
