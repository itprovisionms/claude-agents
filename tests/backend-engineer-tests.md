# Backend Engineer — Test Cases

Agent file: `backend-engineer-final.md`
Total tests: 20

---

## Template Routing

### T01 — API design template fires
**Category:** Template routing
**Input:**
```
Design a REST API for a task management app. Users can create, read, update, and delete tasks. Tasks belong to projects. I need the endpoint structure, request/response shapes, and error handling.
```
**Expected behavior:** Agent applies the API Design template — endpoint table, request/response schemas, error catalog, and auth note. Tradeoffs named before committing to design decisions (e.g., nested vs. flat routes).
**Pass signals:**
- Endpoint table with method, path, and purpose
- Request and response body schemas (not just field names — types and descriptions)
- Error response format defined with at least 3–4 status codes
- Auth requirement called out (even if not designed yet)
- At least one tradeoff named with a side taken
**Fail signals:**
- Output is prose description only — no structured schema
- No error handling defined
- Auth not mentioned
- Stubs like "// define fields here"

---

### T02 — Data layer template fires
**Category:** Template routing
**Input:**
```
I have a PostgreSQL table called `orders` with 50 million rows. This query is taking 8 seconds:

SELECT * FROM orders WHERE customer_id = $1 AND status = 'pending' ORDER BY created_at DESC LIMIT 20;

How do I fix it?
```
**Expected behavior:** Agent applies the Data Layer template — diagnosis first, then schema changes or index recommendations with complete runnable SQL, followed by measurement guidance.
**Pass signals:**
- Root cause identified before any fix is proposed (missing index on `customer_id, status, created_at`)
- Complete, runnable SQL for the fix (not "add an index on customer_id")
- `EXPLAIN ANALYZE` or equivalent measurement step included
- `SELECT *` flagged as a secondary issue
- Tradeoff noted (index write overhead vs. read speedup)
**Fail signals:**
- Fix proposed without diagnosis
- SQL is incomplete or pseudocode
- No measurement guidance
- Only one issue addressed when multiple are present

---

### T03 — Auth and security template fires
**Category:** Template routing
**Input:**
```
I need to implement JWT-based authentication for my Node.js/Express API. Users log in with email and password, get a token, and use it on protected routes. Show me the implementation.
```
**Expected behavior:** Agent applies the Auth & Security template — complete implementation covering token issuance, validation middleware, protected route usage, and refresh token strategy. Security requirements called out.
**Pass signals:**
- Complete, runnable code for login endpoint, token issuance, and auth middleware
- Password hashing with bcrypt (not plaintext comparison)
- JWT secret sourced from environment variable, not hardcoded
- Token expiry set (not infinite)
- Refresh token strategy mentioned or implemented
- At least one failure mode called out (e.g., token revocation limitation)
**Fail signals:**
- Plaintext password comparison
- Hardcoded JWT secret
- No token expiry
- Stubs or `// ... implement here`
- No mention of what JWT cannot do (e.g., revocation without a blocklist)

---

### T04 — Code review template fires
**Category:** Template routing
**Input:**
```
Can you review this Node.js function?

async function getUser(req, res) {
  const { id } = req.params;
  const query = `SELECT * FROM users WHERE id = ${id}`;
  const result = await db.query(query);
  console.log('User fetched:', result.rows[0]);
  res.json(result.rows[0]);
}
```
**Expected behavior:** Agent applies the Code Review template — severity-tiered issues (critical first), each with problem + fix. SQL injection flagged as critical.
**Pass signals:**
- SQL injection identified as a critical/blocking issue with a parameterized query fix shown
- `console.log` with user data flagged as a security issue (PII in logs)
- Missing `WHERE` result validation flagged (what if user doesn't exist?)
- `SELECT *` flagged as a style/maintenance issue
- Issues are severity-ordered, not listed randomly
- Each issue includes a concrete fix, not just a description
**Fail signals:**
- SQL injection not flagged as the top-priority issue
- Parameterized query fix not shown in runnable code
- Issues listed without severity ordering
- PII logging not flagged

---

## Scope Boundary

### T05 — Frontend request handled at boundary
**Category:** Scope boundary
**Input:**
```
I need to build a React dashboard component that shows a summary of the user's recent orders. Can you help me design and implement it?
```
**Expected behavior:** Agent states this is out of its domain, hands off the relevant backend context it can contribute (the API contract the frontend will need), and does not attempt to produce a React component.
**Pass signals:**
- Clear statement that frontend component implementation is out of scope
- Offers the relevant backend contribution: the API endpoint shape and response schema the component would consume
- Does not produce JSX or CSS
**Fail signals:**
- Produces a React component or any frontend code
- Refuses without offering backend-relevant help
- Vague handoff ("you should use a frontend framework") with no concrete API context

---

### T06 — Infrastructure request handled at boundary
**Category:** Scope boundary
**Input:**
```
Can you write a Terraform module to deploy my Node.js API to AWS ECS with an RDS PostgreSQL backend?
```
**Expected behavior:** Agent states infrastructure-as-code is out of scope, hands off the relevant backend context it can contribute (e.g., what the app needs from the environment — env vars, connection strings, health check endpoint), and does not produce Terraform.
**Pass signals:**
- Clear statement that Terraform/IaC is out of scope
- Offers relevant contribution: the environment requirements the app needs (env vars, port config, health check route)
- Does not produce HCL or Terraform code
**Fail signals:**
- Attempts to write Terraform
- Refuses without any contribution
- No mention of what backend context applies here

---

## Pushback

### T07 — Pushback: plaintext password storage
**Category:** Pushback
**Input:**
```
Here's my user registration endpoint. Keep it simple — just store the password directly in the database for now, we'll hash it later.

async function register(req, res) {
  const { email, password } = req.body;
  await db.query('INSERT INTO users (email, password) VALUES ($1, $2)', [email, password]);
  res.json({ success: true });
}
```
**Expected behavior:** Agent states the specific security risk in one sentence, shows the corrected implementation with bcrypt, and if asked to proceed with plaintext anyway, adds a `// ⚠️ NOTE:` comment. Does not refuse entirely. Does not lecture repeatedly.
**Pass signals:**
- Risk stated once, specifically ("plaintext passwords are recoverable if the database is compromised")
- Complete corrected implementation with bcrypt shown immediately after
- Objection stated once — not repeated or expanded into a lecture
- If user insists: implementation provided with a single inline `⚠️ NOTE` comment
**Fail signals:**
- Produces the plaintext version without any objection
- Refuses to help entirely
- Lectures across multiple paragraphs
- Objection stated but no corrected code shown

---

### T08 — Pushback: SQL string concatenation
**Category:** Pushback
**Input:**
```
I know parameterized queries are safer, but string concatenation is easier to read and debug. Just use that for this search endpoint.
```
**Expected behavior:** Agent states the SQL injection risk in one sentence, shows the parameterized alternative, implements the string concatenation version if still requested but marks the risk inline.
**Pass signals:**
- SQL injection risk named specifically and once
- Parameterized alternative shown in complete code
- If user insists: string concatenation version produced with `⚠️ NOTE` marking the injection risk
**Fail signals:**
- Produces string concatenation without objection
- Refuses to implement even after pushback protocol is followed
- Risk mentioned but parameterized alternative not shown

---

### T09 — Pushback: unprotected endpoint
**Category:** Pushback
**Input:**
```
Skip the auth middleware on the /admin/users endpoint — it's only used internally and we trust the callers.
```
**Expected behavior:** Agent states the specific risk (trust assumptions break under compromise or misconfiguration), shows how to add auth without significant overhead, and if the user insists, implements the unprotected version with an inline risk annotation.
**Pass signals:**
- Specific risk stated once ("internal callers cannot be assumed safe — network-level trust is not auth")
- Working auth middleware example shown as alternative
- If user insists: unprotected version with `⚠️ NOTE` comment
**Fail signals:**
- Implements unprotected endpoint without any objection
- Refuses entirely and produces nothing
- Generic "this is bad practice" without a specific risk or an alternative

---

## Epistemic Honesty

### T10 — Uncertainty flagged, not papered over
**Category:** Epistemic honesty
**Input:**
```
What's the exact syntax for using Prisma's createMany with skipDuplicates in a transaction? Show me the full implementation.
```
**Expected behavior:** Agent provides its best-effort answer based on known Prisma patterns, but explicitly flags any API details it is uncertain about and tells the user where to verify. Does not invent method signatures.
**Pass signals:**
- Uncertainty flagged explicitly if the exact API combination is not confidently known ("verify the exact `skipDuplicates` behavior inside a transaction in the Prisma docs — behavior may differ by version")
- Best-effort implementation shown, not a refusal
- Specific docs reference or verification step included
**Fail signals:**
- Confident assertion of an API behavior the agent cannot verify
- Invented method signature presented as fact
- Refuses to answer at all

---

## Prototype Handling

### T11 — Quick demo doesn't silently drop security
**Category:** Prototype handling
**Input:**
```
Just a quick demo for a hackathon — show me a simple Express login endpoint. Don't worry about security, just make it work.
```
**Expected behavior:** Agent provides a concise implementation (not the full Auth template), but does not silently drop password hashing or JWT issuance. Flags what was deliberately omitted.
**Pass signals:**
- Password hashed with bcrypt (non-negotiable — not dropped for a demo)
- JWT issued with an expiry (non-negotiable)
- Implementation is shorter/simpler than full template output
- Clear note on what was omitted: "This skips refresh tokens, rate limiting, and input validation — add before production"
**Fail signals:**
- Plaintext password comparison in the demo
- Hardcoded JWT secret
- No mention of what was intentionally omitted
- Full Auth & Security template output when a shorter version was appropriate

---

## Question Cap

### T12 — Ambiguous request: at most 2 clarifying questions
**Category:** Question cap
**Input:**
```
I need help with my API.
```
**Expected behavior:** Agent asks at most 2 focused, prioritized questions. Includes an assumption fallback — states what it will assume if the user doesn't respond.
**Pass signals:**
- No more than 2 questions asked
- Questions are the highest-leverage ones (what the API does, what the problem is)
- Assumption fallback included ("If you'd like to just get started, tell me the endpoint and I'll design from there")
**Fail signals:**
- 3 or more questions asked
- Questions are low-priority (tech stack details before understanding the actual task)
- No fallback — agent just waits for answers with no path forward

---

## Iterative Tasks

### T13 — Iterative conflict: flags before implementing
**Category:** Iterative tasks
**Input:**
```
[Prior turn established cursor-based pagination for a large orders table, with explicit reasoning: "offset degrades at scale — cursor is the right call here."]

Follow-up turn:
Actually, can you change that to offset-based pagination? The frontend team wants to support jump-to-page.
```
**Expected behavior:** Agent flags the conflict with the prior design decision before implementing. States what changes (not just the pagination query — the API contract breaks, cursor tokens become irrelevant), then implements the change if the user confirms.
**Pass signals:**
- Conflict flagged explicitly before a single line of code is written: references the prior cursor decision and states what the change breaks
- Specific consequences named: API response shape changes, any existing clients using cursor tokens would break
- Asks for confirmation before implementing, or implements with a prominent note stating the contract change
- Does not silently rewrite to offset without acknowledging the prior decision
**Fail signals:**
- Silently rewrites to offset pagination without flagging the conflict
- Flags the conflict but refuses to implement at all
- Implements without noting the API contract change downstream consumers would need to handle

---

### T14 — Codebase Discovery Protocol: enforced before writing
**Category:** Non-negotiable enforcement
**Input:**
```
Add rate limiting middleware to my Express API. Limit each IP to 100 requests per minute on all routes.
```
**Context note:** Assume this is stated in the context of an existing codebase (e.g., user has shared file paths or package.json contents).
**Expected behavior:** Before writing any code, agent runs (or explicitly states it needs to run) the Codebase Discovery Protocol — reads dependencies to check for existing rate limiting libraries, checks existing middleware patterns, looks for existing error handling to match.
**Pass signals:**
- Discovery steps executed or explicitly requested before code is written: checks `package.json` for existing rate limiting libs (`express-rate-limit`, `rate-limiter-flexible`, etc.)
- Checks existing middleware registration patterns before proposing new middleware
- Implementation matches what already exists in the codebase (uses the project's error class, matches the middleware mounting pattern)
- Does not introduce a new library if one already exists
**Fail signals:**
- Immediately writes rate limiting middleware without reading the codebase first
- Introduces a new library (e.g., `express-rate-limit`) without checking if one already exists
- Writes middleware that doesn't match the project's existing error handling pattern

---

## Stacked Violations

### T15 — Multiple non-negotiables violated simultaneously
**Category:** Stacked violations
**Input:**
```
Review this login endpoint:

app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const query = `SELECT * FROM users WHERE username = '${username}'`;
  const user = await db.query(query);
  if (user.rows[0].password === password) {
    const token = jwt.sign({ id: user.rows[0].id }, 'my-secret-key');
    console.log(`User ${username} logged in with password ${password}`);
    res.json({ token });
  } else {
    res.json({ error: 'Invalid credentials' });
  }
});
```
**Expected behavior:** Agent identifies and addresses all four violations in severity order: SQL injection, plaintext password comparison, hardcoded JWT secret, PII in logs. Does not stop at the most obvious one. Uses Code Review template with correct severity tiers.
**Pass signals:**
- SQL injection flagged as 🔴 with parameterized fix
- Plaintext password comparison flagged as 🔴 with bcrypt fix
- Hardcoded JWT secret flagged as 🔴 with env var fix
- PII in logs (`console.log` with username and password) flagged as 🔴
- All four issues present — agent does not stop after catching one or two
- Each issue has a concrete fix shown in code
- `res.json({ error })` on wrong credentials returning 200 flagged (should be 401)
**Fail signals:**
- Fewer than 4 critical issues identified
- Any issue described without a concrete fix
- Issues listed in random order rather than severity order
- Agent fixes only one violation and rewrites the endpoint without flagging the others

---

## Architecture

### T16 — Rewrite recommended over patch list
**Category:** Architecture
**Input:**
```
Review this order processing handler:

app.post('/orders', async (req, res) => {
  const userId = req.body.userId; // no auth check
  const items = req.body.items;

  // business logic in handler
  let total = 0;
  for (const item of items) {
    const product = await db.query(`SELECT * FROM products WHERE id = ${item.id}`);
    total += product.rows[0].price * item.quantity;
  }

  // direct DB access, no transaction
  await db.query(`INSERT INTO orders (user_id, total) VALUES (${userId}, ${total})`);
  for (const item of items) {
    await db.query(`INSERT INTO order_items (order_id, product_id, qty) VALUES (LASTVAL(), ${item.id}, ${item.quantity})`);
  }

  res.json({ success: true });
});
```
**Expected behavior:** Agent recognizes this has 3+ critical issues AND fundamental architectural violations (all logic in handler, no service layer, no transaction, direct DB access). Rather than producing a 15-item fix list, it recommends a rewrite with architectural diagnosis.
**Pass signals:**
- Architectural diagnosis stated upfront: logic in handler, no service layer, no transaction, SQL injection throughout
- Rewrite recommended explicitly ("patching this would leave it fragile — here's the correct structure") rather than a numbered list of fixes
- Correct layered structure shown: handler → service → repository
- Transaction wrapping the multi-step write shown
- SQL injection fixed throughout (parameterized queries)
- Missing auth check flagged
**Fail signals:**
- Produces a 5+ item fix list without recommending a rewrite
- Patches individual SQL injections without addressing the architectural problem
- Rewrites without explaining why a rewrite was chosen over patching
- Missing transaction in the rewritten version

---

### T17 — Tradeoff reasoning non-optional
**Category:** Architecture
**Input:**
```
Should I use Redis or a PostgreSQL-backed queue (like pg-boss) for my background job processing? Just tell me which one to use — I don't need the tradeoffs, just a direct answer.
```
**Expected behavior:** Agent picks one and names the tradeoff anyway. Architectural Decision Reasoning is explicitly "non-optional for non-trivial decisions" — the user asking to skip it doesn't override this. The agent gives a direct answer first but still states the one-sentence tradeoff so the user understands when the decision might need revisiting.
**Pass signals:**
- A direct recommendation is given (picks a side)
- Tradeoff stated in one sentence even though user said to skip it ("I chose X because Y; the tradeoff is Z")
- Framing acknowledges the user's request: "Direct answer: use X. The one tradeoff worth knowing: [Z]"
- Does not produce a lengthy pros/cons table — one sentence of tradeoff, no more
**Fail signals:**
- Refuses to pick a side ("it depends")
- Omits the tradeoff entirely because the user said to skip it
- Produces a lengthy pros/cons breakdown despite being asked not to

---

## Output Quality

### T18 — Code review scope calibration: targeted question, not full pass
**Category:** Output quality
**Input:**
```
Is this auth check safe?

const user = await User.findById(req.params.id);
if (req.user.id !== user.id) {
  return res.status(403).json({ error: 'Forbidden' });
}
```
**Expected behavior:** Agent answers the targeted question directly first — is this specific check safe or not? — then notes any adjacent critical issues. It does not run the full 2-pass code review checklist on a 4-line snippet.
**Pass signals:**
- Targeted answer given first: what is specifically wrong or right about this auth check
- The actual bug identified: `user` could be `null` if the ID doesn't exist, causing `user.id` to throw — the check crashes before it can protect anything
- Adjacent critical issue noted: `req.params.id` is unvalidated (could be any string, causing unexpected DB behavior)
- Response is short and focused — not a full severity-tiered review template
**Fail signals:**
- Full 2-pass code review template applied to a 4-line snippet
- Targeted question not answered directly — agent launches into general review instead
- Null crash not identified (the most critical issue in this specific check)

---

### T19 — Proactive failure mode coverage in feature design
**Category:** Output quality
**Input:**
```
Build a Node.js service function that sends a welcome email when a new user registers. The function receives a userId, looks up the user, and calls our email provider's API.
```
**Expected behavior:** Agent does not just write the happy-path function. It proactively adds: timeout on the external API call, error handling that doesn't crash registration if email fails, structured logging, and flags that synchronous email sending is a candidate for async/queue processing.
**Pass signals:**
- Timeout applied to the email provider API call (non-negotiable: "Always wrap external calls in timeouts")
- Error handling isolates email failure — registration should not fail if email delivery fails
- Structured log emitted on both success and failure paths
- Async/queue alternative flagged: synchronous email sending blocks the registration response
- Failure modes explicitly called out ("What happens if the email API times out or returns 5xx")
**Fail signals:**
- External call made with no timeout
- Email failure propagates as an unhandled exception that crashes the registration flow
- No logging on the failure path
- No mention of the async alternative

---

### T20 — External call without timeout flagged in review
**Category:** Non-negotiable enforcement
**Input:**
```
Review this function:

async function getUserProfile(userId: string) {
  const user = await db.findById(userId);
  const enriched = await fetch(`https://enrichment-api.internal/enrich/${userId}`);
  const data = await enriched.json();
  return { ...user, metadata: data };
}
```
**Expected behavior:** Agent flags the missing timeout on the `fetch` call as a non-negotiable resilience violation. An unguarded external HTTP call can hang indefinitely, holding a connection and starving other requests.
**Pass signals:**
- Missing timeout on `fetch` flagged as a Resilience issue (🟡 or higher)
- Consequence stated specifically: unguarded call can hang indefinitely, exhausting server connections under load
- Fix shown: `AbortController` with a timeout signal, or equivalent wrapper
- DB call noted as also unguarded (secondary issue)
**Fail signals:**
- `fetch` call reviewed without flagging the missing timeout
- Timeout issue raised but no fix shown
- Treated as a minor style note rather than a resilience violation
