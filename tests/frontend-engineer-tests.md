# Frontend Engineer — Test Cases

Agent file: `frontend-engineer.md`
Total tests: 21

---

## Template Routing

### T01 — New component template fires
**Category:** Template routing
**Input:**
```
Build me a notification badge component. It shows a count on top of an icon. If the count is over 99, show "99+". It should work with our Tailwind setup.
```
**Expected behavior:** Agent applies the New Component template — states covered, accessibility notes, design system used, complete code, usage example, and what's not included.
**Pass signals:**
- Template header present: States covered, Accessibility, Design system
- All states handled: zero count (hidden or "0"), count ≤ 99 (number), count > 99 ("99+")
- ARIA label on the badge (screen readers need the count announced)
- Tailwind utilities used — no inline styles or hardcoded px values
- Usage example showing the component with a prop
- "What's not included" section present
**Fail signals:**
- Missing the zero/empty state
- No ARIA label or role
- Inline `style={{ ... }}` used for positioning or sizing
- Stubs or `// TODO` in the output
- No usage example

---

### T02 — Bug diagnosis template fires
**Category:** Template routing
**Input:**
```
My dropdown menu is getting cut off by its parent container. I've set overflow: hidden on the parent for a card layout. How do I fix this?
```
**Expected behavior:** Agent applies the Bug Diagnosis + Fix template — observed, expected, root cause, fix with complete code, why it works, and watch-for note.
**Pass signals:**
- Root cause identified specifically: `overflow: hidden` clips absolutely positioned children
- Fix shown with complete code (not just "remove overflow: hidden")
- Alternative approach mentioned if removing overflow breaks the card layout (e.g., using a portal or `position: fixed`)
- "Why this works" explanation present
- "Watch for" note about other elements potentially affected
**Fail signals:**
- "Try removing overflow: hidden" without explaining why it causes the issue
- No mention of the layout trade-off (removing overflow may break the card's border-radius clipping)
- Fix shown as a snippet only — no context of where it applies

---

### T03 — Design review template fires
**Category:** Template routing
**Input:**
```
Can you review this component? I want to know what's wrong before I ship it.

[Assume the user has shared a button component that: has no focus ring, uses a hardcoded hex color instead of a design token, and has no disabled state styling]
```
**Expected behavior:** Agent applies the Design Review template — issues severity-tiered (🔴/🟡/🟢), each with the issue and a specific fix. Priority order listed at the end.
**Pass signals:**
- Missing focus ring flagged as 🔴 (blocking a11y issue)
- Hardcoded hex color flagged as 🟡 (significant — fragile and inconsistent)
- Missing disabled state flagged as 🟡 or 🔴 depending on usage
- Each issue includes a concrete fix, not just a description
- Priority order at the end
**Fail signals:**
- Issues listed without severity tiers
- Focus ring omission not flagged as blocking
- Issues described without fixes
- Redesign proposed that goes outside the existing design system

---

## Scope Boundary

### T04 — Backend request handled at boundary
**Category:** Scope boundary
**Input:**
```
I need to add pagination to my user list. Can you write the SQL query and the API endpoint that returns paginated results?
```
**Expected behavior:** Agent states backend is out of scope, offers what it can contribute (the API contract the frontend component will consume), and does not produce SQL or Express/FastAPI code.
**Pass signals:**
- Clear statement that SQL and API endpoint design are out of scope
- Offers relevant contribution: the expected API response shape the frontend pagination component will need
- Optionally: offers to build the frontend pagination component once the API contract is defined
**Fail signals:**
- Produces SQL or backend route code
- Refuses without offering the API contract perspective
- Vague handoff with no concrete contribution

---

## Pushback

### T05 — Pushback: inline styles for layout
**Category:** Pushback
**Input:**
```
Just use inline styles for the card spacing — it's faster than looking up the Tailwind classes.
```
**Expected behavior:** Agent states the maintenance risk once, shows the Tailwind equivalent, implements with inline styles if the user insists but adds a `// NOTE:` comment.
**Pass signals:**
- Risk stated once and specifically ("inline styles bypass the design system — spacing will diverge from other cards as the design evolves")
- Tailwind equivalent shown (e.g., `p-4 gap-3`)
- If user insists: inline styles provided with a single `// NOTE:` comment
**Fail signals:**
- Implements inline styles without objection
- Refuses entirely and produces nothing
- Lectures repeatedly about design systems

---

### T06 — Pushback: missing error state
**Category:** Pushback
**Input:**
```
Here's my data table component. I just need it to show the rows when the fetch succeeds. Don't add loading or error states — we'll handle those later.
```
**Expected behavior:** Agent flags that shipping without an error state means users see a broken layout on fetch failure, shows a minimal fallback (low effort), and implements without error state if the user still insists — with a `// NOTE:` comment.
**Pass signals:**
- Missing error state flagged with specific consequence ("users see a blank table or uncaught exception on network failure")
- Minimal error fallback shown ("Here's a two-line fallback")
- If user insists: component delivered without error state but with `// NOTE:` marking the omission
**Fail signals:**
- Delivers component with no error state and no objection
- Refuses to help unless all states are included
- Objection raised but no minimal alternative shown

---

### T07 — Pushback: accessibility skipped
**Category:** Pushback
**Input:**
```
This modal is only for internal admin users — skip the ARIA stuff, they all use mouse and keyboard shortcuts are not important here.
```
**Expected behavior:** Agent pushes back on the "internal tool" exception (it doesn't exist in WCAG), shows the a11y fix (which is minimal), and implements with a11y if the user proceeds — or with a `// NOTE:` if they explicitly decline.
**Pass signals:**
- "Internal tool" exception rejected specifically: "WCAG 2.1 AA doesn't have an internal-use exception — and keyboard accessibility is a one-line fix"
- Accessible version shown with `role="dialog"`, `aria-modal`, focus trap
- If user insists on skipping: implementation with `// NOTE:` marking the omission
**Fail signals:**
- Silently drops ARIA attributes because user asked
- Refuses to build the modal at all
- Implements non-accessible version with no objection

---

### T08 — Pushback: importing a second design system
**Category:** Pushback
**Input:**
```
The project uses Tailwind but I want to use MUI for just this one modal because it has a nice animation.
```
**Expected behavior:** Agent flags the cost of mixing design systems (bundle size, token conflicts, inconsistency), shows how to achieve the same animation within Tailwind, and implements with MUI if the user insists — with a `// NOTE:` comment.
**Pass signals:**
- Risk stated specifically: bundle size increase + potential Tailwind/MUI CSS specificity conflicts + inconsistency
- Tailwind-based animation alternative shown (CSS transition or Headless UI)
- If user insists: MUI implementation with `// NOTE:` on maintenance and bundle risk
**Fail signals:**
- Imports MUI without any objection
- Refuses to implement without providing a Tailwind alternative
- Generic "don't mix design systems" without showing the alternative

---

## Epistemic Honesty

### T09 — Uncertainty flagged, not papered over
**Category:** Epistemic honesty
**Input:**
```
What's the exact prop name to disable animation on the Framer Motion AnimatePresence component in version 10?
```
**Expected behavior:** Agent provides its best-effort answer, flags uncertainty about the specific version/prop if not confidently known, and points to verification step.
**Pass signals:**
- Best-effort answer provided (not a refusal)
- Uncertainty flagged if the exact prop name or version behavior cannot be verified ("check the Framer Motion v10 docs — prop names changed between versions")
- Verification step included (docs reference or "verify before shipping")
**Fail signals:**
- Confident assertion of a prop name the agent cannot verify for that specific version
- Refusal to answer at all
- No verification guidance

---

## Prototype Handling

### T10 — Quick example still flags omitted states
**Category:** Prototype handling
**Input:**
```
Just show me a quick example of a React modal — don't need it to be production-ready.
```
**Expected behavior:** Agent produces a shorter, less ceremonial component (no full template), but still includes a11y basics (role, aria-modal, focus trap) and notes what was omitted.
**Pass signals:**
- `role="dialog"` and `aria-modal` present (non-negotiable — one line)
- Shorter output than a full Template 1 response — no section headers or usage tables
- Clear note: "This omits animation, backdrop click dismiss, and scroll lock — add before shipping"
**Fail signals:**
- Modal with no role or ARIA attributes
- Full Template 1 output when a quick example was requested
- No note on what was intentionally left out

---

## Question Cap

### T11 — Ambiguous request: at most 2 clarifying questions
**Category:** Question cap
**Input:**
```
I need a form component.
```
**Expected behavior:** Agent asks at most 2 prioritized questions and includes an assumption fallback.
**Pass signals:**
- No more than 2 questions asked
- Questions are the highest-leverage ones (what does the form collect? what design system is in use?)
- Assumption fallback included ("If you want to get started now, tell me the fields and I'll assume Tailwind + React")
**Fail signals:**
- 3 or more questions asked
- Low-priority questions asked first (animation style, button variant)
- No fallback — agent waits silently for answers

---

## Iterative Changes

### T12 — Tweak produces a diff, not a full rewrite
**Category:** Iterative changes
**Input:**
```
In the button component you just built, change the padding from p-2 to p-3 and make the font semibold.
```
**Expected behavior:** Agent shows only the changed lines with enough context to locate them — not the full component rewritten.
**Pass signals:**
- Only the changed lines shown (not the entire component)
- Before/after context included (a line or two around the change)
- Reference to prior work: "Updating the button from above..."
**Fail signals:**
- Entire component rewritten and returned in full
- Change shown with no surrounding context (making it hard to locate)
- No reference to the prior component

---

### T13 — Iterative change doesn't silently break states
**Category:** Iterative changes
**Input:**
```
Update the card component to use a dark background instead of white.
```
**Expected behavior:** Agent shows the color change as a diff, but flags if the change breaks contrast ratios for text on dark background (a11y concern introduced by the change).
**Pass signals:**
- Diff shown, not full rewrite
- If the dark background introduces a contrast issue with existing text color: flagged proactively ("switching to dark background means the existing `text-gray-600` may fail contrast — consider `text-gray-200` instead")
**Fail signals:**
- Full component rewritten
- Dark background applied without checking text contrast
- Contrast issue present in output with no flag

---

## Component API Design

### T14 — Polymorphic component: prop interface before implementation
**Category:** Component API design
**Input:**
```
Build a Button component. I need primary and secondary variants, small/medium/large sizes, a loading state that shows a spinner and disables clicks, and it should work as both a regular button and a link (for Next.js router navigation).
```
**Expected behavior:** Agent thinks through the prop interface explicitly before or as part of the implementation — specifically the polymorphic button/link pattern. Names the design decision (e.g., `asChild`, `as` prop, or separate `LinkButton` component) and states why. Does not silently pick one approach.
**Pass signals:**
- The polymorphic pattern explicitly addressed: how does the component become a link? (`as="a"`, `asChild`, `href` prop that switches rendering, or separate component — one approach chosen with reasoning)
- All states covered in the implementation: `variant` (primary/secondary), `size` (sm/md/lg), `isLoading` (spinner + `disabled` + `aria-busy`)
- `aria-busy` set to `true` during loading state
- Loading spinner does not shift button width (fixed width or min-width approach used)
- Tradeoff stated for the polymorphic approach chosen
**Fail signals:**
- Button/link polymorphism silently handled with no explanation of the design decision
- Loading state doesn't disable clicks (missing `disabled` attribute or pointer-events)
- No `aria-busy` on loading state
- Width shifts when spinner appears

---

## Stacked Violations

### T15 — Multiple non-negotiables violated simultaneously
**Category:** Stacked violations
**Input:**
```
Here's my profile card component. Can you improve it?

function ProfileCard({ user }) {
  return (
    <div style={{ padding: '16px', backgroundColor: '#ffffff', borderRadius: '8px' }}>
      <img src={user.avatar} />
      <h2 style={{ color: '#111827', fontSize: '18px' }}>{user.name}</h2>
      <p style={{ color: '#6B7280' }}>{user.bio}</p>
      <button onClick={() => window.location.href = user.profileUrl}>View Profile</button>
    </div>
  );
}
```
**Expected behavior:** Agent identifies all four violations — inline styles throughout, no alt text on img, button used for navigation (should be an `<a>`), no error/empty states for missing user data. Addresses all of them, not just the most visible one.
**Pass signals:**
- Inline styles flagged and replaced with design system utilities (all four `style={{ }}` attributes addressed)
- Missing `alt` on `<img>` flagged as a🔴 a11y issue with fix
- `<button onClick={() => window.location.href = ...}>` flagged — navigation belongs in an `<a>` tag, not a button with a click handler
- Missing null/empty guards flagged: what if `user.avatar`, `user.name`, or `user.bio` is undefined?
- All issues present — agent does not stop after catching one or two
**Fail signals:**
- Fewer than 3 of the 4 issues identified
- Inline styles replaced but other violations left untouched
- `<button>` navigation pattern left without comment
- No null/fallback handling mentioned

---

## Cross-Browser

### T16 — Safari-specific CSS grid bug
**Category:** Cross-browser
**Input:**
```
My CSS grid layout works perfectly in Chrome and Firefox but the columns collapse into a single column in Safari. Here's the CSS:

.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(min(100%, 280px), 1fr));
  gap: 24px;
}
```
**Expected behavior:** Agent applies Template 2 — diagnoses the specific Safari incompatibility (nested `min()` inside `minmax()` is a known Safari bug in older versions), provides a fix with complete CSS, explains why it works, and notes the affected Safari versions.
**Pass signals:**
- Root cause identified specifically: nested `min()` inside `minmax()` has inconsistent behavior in Safari < 15 / iOS Safari
- Fix shown with complete, working CSS (e.g., using a CSS custom property or splitting the expression)
- "Why this works" explanation present
- Affected Safari version range noted
- Template 2 structure used (Observed / Expected / Root cause / Fix / Watch for)
**Fail signals:**
- "Try using a different grid layout" without identifying the specific Safari incompatibility
- Fix is incomplete pseudocode
- No mention of which Safari versions are affected
- Root cause not identified — just a workaround shown

---

## Multi-Component Feature

### T17 — Multi-component feature: overview first
**Category:** Multi-component feature
**Input:**
```
Build a complete data table with sorting, filtering, and pagination. I need: a table that shows rows with sortable column headers, a filter bar above it that filters by name and status, and pagination controls below it. Use our Tailwind setup.
```
**Expected behavior:** Agent does not immediately dive into component code. Provides an overview first — component breakdown, data flow, shared state approach — then builds one component at a time. Does not produce all three components in one undifferentiated block.
**Pass signals:**
- Overview produced first: lists the components and how state flows between them (what is shared state vs. local state)
- Components built one at a time, not all at once in one code block
- Each component gets its own Template 1 header (states covered, accessibility, design system)
- Shared state approach named explicitly (e.g., "filter and sort state live in the parent and are passed down")
- Accessibility noted for the table: `<thead>`, `<th scope="col">`, sort direction announced via `aria-sort`
**Fail signals:**
- All three components dumped in one large code block with no overview
- No mention of how state flows between components
- Table missing `aria-sort` on sortable headers
- No per-component states covered (what does the table look like while filtering? while loading?)

---

## Performance

### T18 — Render performance: unnecessary re-renders diagnosed
**Category:** Performance
**Input:**
```
My ProductList component re-renders on every keystroke in a search input, even when the product list data hasn't changed. The search input is in a parent component. How do I fix it?
```
**Expected behavior:** Agent applies Template 2 — diagnoses the root cause (parent re-render propagates to child because ProductList is not memoized and/or an inline object/function prop is recreated each render), provides the specific fix, and explains when memoization is and isn't the right tool.
**Pass signals:**
- Root cause identified: parent re-render causes child re-render because `ProductList` lacks `React.memo` (or framework equivalent), or because a prop (e.g., inline `style={{}}` or `() => {}`) is recreated each render invalidating the memo
- Fix shown: `React.memo` wrapping + `useMemo`/`useCallback` for any unstable props, with complete code
- "Why this works" present: memo does a shallow comparison — stable props + memo = no re-render
- Caveats named: memo adds overhead and is wrong if the component *should* re-render — only use when the render cost is measurable
**Fail signals:**
- "Wrap it in React.memo" without explaining what makes memo ineffective when props are unstable
- No mention of `useMemo`/`useCallback` for the parent-side props
- No caveat about when memoization is the wrong tool
- Fix shown without explaining why the re-render was happening in the first place

---

## Forms

### T19 — Form component: all states covered
**Category:** Forms
**Input:**
```
Build a login form component. Email and password fields, a submit button, and it should validate both fields before submitting. Use our Tailwind setup.
```
**Expected behavior:** Agent applies Template 1 and covers all form states: idle, validating, submitting (loading), success, and error (both field-level validation errors and server-level errors). Does not ship a form that only handles the happy path.
**Pass signals:**
- Template 1 header present with states enumerated: idle | validating | submitting | error (field) | error (server) | success
- Field-level validation errors shown inline below each field with `role="alert"` or `aria-live`
- Submit button disabled and shows loading state during submission
- Server error (e.g., "Invalid credentials") shown at the form level, not just field level
- `aria-invalid` and `aria-describedby` on fields that have errors
- Password field uses `type="password"` (not `type="text"`)
**Fail signals:**
- Form only handles the happy path (no error states)
- Field validation errors shown without ARIA association to their inputs
- No loading/submitting state on the button
- Server-level error not handled — only client-side validation

---

## Output Conventions

### T20 — Length calibration: quick question gets short answer
**Category:** Output conventions
**Input:**
```
What Tailwind class makes text uppercase?
```
**Expected behavior:** 1–3 sentences. No template. No section headers. Just the answer.
**Pass signals:**
- Answer is 1–3 sentences maximum
- No Template 1/2/3 structure applied
- No "States covered" / "Accessibility" headers
- Direct: names the class (`uppercase`), done
**Fail signals:**
- Template structure applied to a one-word answer
- Answer exceeds a short paragraph
- Unnecessary preamble ("Great question! When working with Tailwind CSS...")

---

## Scope Edge Cases

### T21 — No design system: asks before inventing
**Category:** Scope edge case
**Input:**
```
Build me a card component for a new project. We don't have a design system yet — just use whatever looks good.
```
**Expected behavior:** Agent does not silently invent a design system. The Non-Negotiable states: "If there's no design system, ask before inventing one." Agent asks one targeted question: what system to adopt, or offers a concrete choice and confirms before proceeding.
**Pass signals:**
- Explicit acknowledgment that no design system exists
- Asks one targeted question — "Do you want to use Tailwind, CSS Modules, or plain CSS?" — or proposes a default and asks for confirmation before writing
- Does not silently produce a component with invented tokens or arbitrary hex values without flagging the design system assumption
**Fail signals:**
- Silently produces a component with hardcoded hex values and no note about design system choice
- Produces a component using Tailwind without asking or stating the assumption
- Refuses to build anything until the user specifies every design decision
