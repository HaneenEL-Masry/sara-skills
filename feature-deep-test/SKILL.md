---
name: feature-deep-test
description: Deeply test a single feature against the live app across four lenses — Functional, UI/UX, Performance, and Security — with exhaustive positive, edge, and negative/abuse coverage. Sara discovers the feature's full live surface, builds a governed scenario matrix, runs each lens to real depth herself (not spot checks, not handed off), files confirmed defects via bug-report, and returns one feature verdict with a rich evidence report. Use when the user wants a complete, deep quality pass on ONE feature — "fully test this feature", "deep test the export feature", "test everything about checkout — functional, UX, performance, security, all the edge and negative cases". Not a happy-path flow walk (journey-test), not an acceptance-criteria check (story-test), not a whole-site audit (site-audit), and not a single-lens journey skill — feature-deep-test owns all four lenses itself, scoped to one feature.
---

# Feature Test

Act as a senior quality engineer who proves whether **one whole feature** actually holds up — deeply, across every lens, in the running build.

This skill is the empty cell the other skills don't fill. `journey-test` walks one happy path and judges business value. `story-test` checks one story's acceptance criteria with deliberately shallow lens spot-checks. The `journey-perf` / `journey-sec` / `journey-ux` skills each do *one* deep lens on a *journey*. `site-audit` does all four lenses but across an *entire site*. **`feature-deep-test` takes one feature and runs all four lenses — Functional, UI/UX, Performance, Security — to real depth itself**, with exhaustive positive, edge, and negative/abuse coverage, and rolls everything into a single feature verdict.

It does the deep testing itself. It does **not** delegate the lenses to the journey skills to borrow their depth. The only things it reuses are Sara's plumbing: locator and evidence rules, the `.sara/` knowledge base, `bug-report` for *filing* confirmed defects, and the HTML report style.

Examples:

- `feature-deep-test the export-to-PDF feature`
- `deep test checkout — functional, UI/UX, performance, security, every edge and negative case`
- `fully test the user-invite feature on staging, all lenses, exhaustive`
- `feature-deep-test SENTRA-feature "scope filter" — hammer the role boundaries`

## When To Use

Use this skill when the user wants a **complete, deep quality pass on a single feature**:

- a named feature with a live surface (URL, screen, or component) to exercise
- a feature already described in the `.sara/` knowledge base (Product Kit / Quality Canvas / Value-Quality Map)
- a request to test "everything" about one feature — all four lenses, positive + edge + negative/abuse
- a request to go deep on one feature's guards, performance, or interface while still covering the rest

Do not use this skill for:

- a happy-path business-value walk of a multi-step flow — use `journey-test`
- verifying a single story's acceptance criteria — use `story-test`
- one deep lens on a journey — use `journey-perf`, `journey-sec`, or `journey-ux`
- a whole-site, every-journey four-lens audit — use `site-audit`
- reproducing or verifying one tracked defect — use `bug-test`
- running a saved regression suite — use `agentic-regression`

If the user names a *journey* rather than a feature and wants value judgment, route to `journey-test`. If they name a *story* with acceptance criteria, route to `story-test`. If they want the *whole site*, route to `site-audit`. Say which one you are picking and why before running.

## Core Principle

Optimize for this question:

`Does this whole feature actually hold up in the running build — deeply, across Functional, UI/UX, Performance, and Security, and across its positive, edge, and negative/abuse cases — and where it does not, what is the business consequence?`

Three rules flow from that:

1. **Coverage beats narrative.** A feature-deep-test is complete when every capability the feature owns has its positive path proven, its meaningful edges exercised, and its guards confirmed to hold under negative/abuse cases. `screen loaded` is evidence, not a pass.
2. **Negatives are mandatory, not optional.** A feature is *never* marked verified on positive cases alone when guards exist. Every capability that implies validation, permission, or a guard gets its negative cases actually run.
3. **Depth is the point.** Each lens goes to real depth — this is not a story-scoped spot check. When a lens surfaces something bigger than this feature (a platform-wide perf regression, a systemic authz model flaw), Sara still reports it in full and notes it reaches beyond the feature; she does not shrink the finding to fit the scope.

### Four lenses — what "deep" means

- **Functional (F)** — *verdict-gating.* Every capability × {positive, edge/boundary, negative/abuse}: correct outcomes, data integrity after the action, state transitions, error handling, idempotency/double-submit, concurrency, interrupted flows (back / refresh / expired session), empty/one/many, min/max/over-max, unicode and injection-shaped input as *validation* cases. This is the spine and the source of truth for the verdict.
- **Security (S)** — *verdict-gating.* The feature's own guards **plus** systematic probing: authorization and role boundaries (forced navigation, IDOR), input validation / injection on the authorized target, the captured API request/response, session and token handling, and sensitive-data exposure. A guard that fails to hold is a verdict-deciding failure. Sensitive findings get `sensitive: true` handling.
- **UI/UX (U)** — *observation-only* (ranked follow-ups, does not gate the verdict). **UI:** visual design quality, layout & alignment, spacing, typography & hierarchy, color & contrast, design-system consistency, component states (default/hover/active/focus/disabled/loading/error), responsive rendering incl. mobile width, RTL/Arabic integrity, visual-polish / AI-slop tells. **UX:** Nielsen 10 heuristics, accessibility (WCAG — keyboard, focus order, labels, tap-target size), cognitive walkthrough, feedback & recoverability, empty/error/loading states, cognitive load. The one exception is not a UI/UX downgrade at all: a layout/interaction defect that makes a capability **impossible to complete** is a *Functional* failure, caught by the F lens.
- **Performance (P)** — *observation-only* (ranked follow-ups, does not gate the verdict). Real timing with repeat runs (cold + warm), key-action latency, perceived responsiveness, payload/asset weight, slow-network behavior, jank, and where it degrades — measured numbers with how they were measured, not vibes.

Functional and Security **gate** the feature verdict. UI/UX and Performance are captured deeply and reported as ranked, business-impact-tagged follow-ups — never silently dropped, never the sole reason a feature is marked NOT VERIFIED (unless the user explicitly promotes a lens to gating for that run).

## Inputs

Collect as many of these as the user provides; infer the rest from project context and state the assumptions briefly:

- **the feature** — a name, a live surface (URL / screen / component), or a reference to a feature in the `.sara/` knowledge base
- **target environment or URL** — where the running build lives
- **platform** — web or mobile (native)
- **auth state / roles** — the role(s) needed to exercise the feature, plus any roles required for negative permission cases (e.g. anonymous, wrong-role)
- **test data / accounts** — valid fixtures and deliberately invalid ones; dedicated data for destructive/mutating cases and a cleanup/reset path
- **depth** — exhaustive by default; the user may ask for a quick pass or scale a specific lens up/down
- **focus / priority** — an optional steer ("only the security negatives on export", "run the abuse cases first") that changes case order/priority, not what the matrix owns
- **scope limits** — locale, device/breakpoint, feature-flag state, or sub-capabilities to include/exclude

If the user only names the feature, resolve it (knowledge base + live app) and infer the rest, stating assumptions. If a capability has no observable oracle and no safe inference, ask one focused question before executing rather than guessing — follow Sara's canonical ask + suggest policy. Never run a destructive case without dedicated test data and a cleanup path.

## Feature Discovery

Discovery answers one question before any case runs: **what does this feature actually do, and what is it supposed to do?** It is a *capability-mapping* pass, not a bug-hunt. It runs in two passes, then reconciles into a capability inventory.

**The governing split: discovery maps, execution fires.** Discovery is **non-destructive** — Sara catalogues the surface and *records* every guard, input, and destructive action as a future case. She does **not** fire negative, abuse, or data-mutating cases during discovery. Those run in full during execution (see the Scenario Matrix and Workflow), with dedicated test data, a cleanup/reset path, and sensitive-finding handling. Deferring them is not skipping them — it is the only way to design complete negative cases (you must see the whole surface first) and to run destructive ones safely (you must set up rollback first).

### Pass A — Context read (before touching the app)

Pull what the feature is *supposed* to do. This is the **oracle** — without it Sara can map buttons but cannot judge pass/fail. Read, in this precedence, only what is relevant to this feature:

1. the user's description and any attached spec/story
2. `.sara/knowledge/` — Product Kit / User Journey Map (intended flows), Quality Canvas (quality contracts, expected behaviors), Value-Quality Map (business risks, what matters)
3. `.sara/knowledge/docs/feature-evaluations/` — existing **AI Feature Quality Canvases and feature evaluations** for this feature or its sub-features. Use it to detect that the feature (or a sub-feature) is AI-powered and to **reuse the quality contracts** — always, they are the stable *spec* of correct output that feeds the AI handoff (see *AI-powered features*). **Reuse the evaluation *results* only if they are still fresh for the current build**; if stale, or the AI/prompt/model may have changed, hand off to `evaluate-genai-feature` for a fresh run rather than trusting old scores. Cross-link the bundles via `related_items`. Never present a stale AI evaluation as current.
4. `.sara/regression/` — existing **agentic regression suites** for this feature: read the suite(s) and their maps to **(a) tag** each matrix case that a suite already guards (`covered-by-regression: <suite>`) — still run it in an exhaustive pass (re-verify; the live build may have regressed since the suite last passed), but drop already-guarded cases *first* when a quick/trimmed run forces a cut, and report net-new vs. re-verified coverage separately; and **(b) seed** locator learning from the suite's healed maps (read-only). Reconcile, never silently skip.
5. `.sara/experience/known-issues/` and `.sara/experience/heuristics/` — what tends to break here
6. recent `.sara/features/<slug>/runs/` and related `.sara/journeys/`/`.sara/stories/` runs — prior verdicts and reusable locator maps to *seed* learning (read-only)
7. `.sara/project-context.md`, `.mcp.json`, `CLAUDE.md`, `README.md` — environment, roles, locale rules, login notes

If the knowledge base has nothing on this feature, proceed anyway: infer the oracle from the live UI + the user, and **record in the run that case design was unguided by project quality context** (it lowers confidence in negative/edge selection, it does not invalidate the run).

### Pass B — Live capability map (explore the running app)

Navigate to the feature in the live headed browser (Playwright MCP) or the native app (Appium MCP) and **systematically inventory the real, testable surface** using locator-first reads — the accessibility snapshot, never screenshots-to-read. Discovery is bounded to **Option A: map only what is reachable without side effects.** Sara may:

- open panels/modals/menus, expand sections, follow safe read-only navigation, page through lists, reach empty/loaded states that appear without mutating data
- read every interactive element the feature owns — buttons, fields, toggles, filters, menus, links — and each input's apparent constraints (required, type, min/max length, format)
- note every sub-flow, state, and route the feature can reach, and every role/permission boundary it implies

Sara may **not**, during discovery: submit/create/update/delete, complete a non-idempotent action, fire a wrong-role or forced-navigation attempt, or send invalid/injection input. Each of those is *mapped as a case*, not executed. The happy path itself is **walked in execution**, not discovery (Option A) — so any state that only appears after a real submit is mapped as "downstream state, reached in execution," and the matrix is allowed to grow when execution reveals it (see Scenario Matrix → matrix grows mid-run).

Use the **First-Run Locator Learning** and **Snapshot/Evaluate Economy** discipline: learn the tightest stable locator from observed node metadata on first contact, validate it resolves to exactly one element before saving, and batch read-only reads instead of snapshotting every element. If an expected control is not in the snapshot, take **one** screenshot to inspect the visible UI (weak semantics / icon-only / position-driven layout are common causes) before falling back to coordinates — diagnosis only, not to read state.

### Reconcile → the capability inventory

Merge the two passes: **live surface** (what the feature actually does) + **context** (what it is supposed to do, including invisible guards). Live exploration catches what the docs miss; context catches expected guards you cannot see by looking (e.g. "a non-admin must be blocked" — invisible in the UI, derived from the role model). The output is a numbered **capability inventory** — one row per capability the feature owns:

- `C<n>` id and a one-line name
- the surface (route/screen + the locator(s) that reach it)
- the intended behavior / oracle (from context, or inferred + flagged)
- inputs and apparent constraints
- guards implied (validation / permission / forced-navigation / idempotency / rate)
- destructive? (mutates persisted state → needs test data + cleanup in execution)
- roles relevant
- source: `live`, `context`, or `both`

Persist the inventory as `feature.md` under the feature folder (see Artifact Contract). It is the source of truth that the Scenario Matrix expands into cases.

### Scope gate (because exhaustive is the default)

After the inventory and the proposed matrix size are known, and **before execution spends the time**, surface the scope to the user: e.g. *"9 capabilities → ~74 cases across 4 lenses (exhaustive). Run all, trim, or focus a lens/capability?"* For an explicit quick run, skip the gate and proceed with the reduced budget. Honor any focus/priority steer here (it reorders/prioritizes cases; it never removes the matrix's ownership of a case). This gate is the one human checkpoint between mapping and firing — keep it short and decision-grade.

## The Scenario Matrix

The matrix turns the capability inventory into an explicit, numbered set of cases. It is the contract for the run: every case traces to a capability, and every capability is accounted for. Write it to `matrix.md` before execution.

### From capability to cases

For each capability `C<n>`, derive cases tagged by **class**:

- **Positive (P)** — the capability working on the expected path. At least one per capability; more only when there are genuinely distinct valid paths.
- **Edge (E)** — boundary and unusual-but-valid inputs/states: empty, min, max, just-over-max, zero, very long, special characters, unicode, leading/trailing spaces, duplicate, concurrent, slow network, back/refresh mid-flow, expired-session re-entry, locale/timezone boundaries, list with zero/one/many items, first/last item.
- **Negative / abuse (N)** — inputs and actions that must be rejected or guarded: invalid format, wrong credentials, missing required field, **unauthorized role**, **forced/tampered navigation (IDOR, direct URL to a guarded step)**, **injection-shaped input** (as input-validation against the authorized target), double-submit, out-of-order steps, replaying a one-time action. Every guard the inventory recorded becomes at least one N-case. **These are mandatory** — a feature is never verified on positive cases alone when guards exist.

Each case gets:

- a stable id `C<n>-<class><k>` (e.g. `C2-N1`)
- a one-line intent
- the concrete input / precondition
- the expected outcome (**the oracle**) — for P/E the correct result, for N the correct rejection/guard
- the **lens tag(s)** it serves: **F / U / P / S**
- a `destructive?` flag and, if set, the test-data fixture + cleanup/reset path it will use
- the role it runs as

### Lens tags vs dedicated lens probes

Lens tags are **attributes of a case, not separate cases**. One submit case can be tagged `F+U+P+S` and is still **one case** — the lenses are extra observations captured during that single interaction (functional oracle, UI/UX read, timing, guard check). Never split one interaction into per-lens cases; it inflates the count and the budget dishonestly.

Because this skill runs UI/UX and Performance to *real depth* (not story-scoped spot checks), the matrix also carries **dedicated lens probes** where depth needs its own case — these are still numbered rows, tagged with their lens, traced to a capability or the feature surface:

- **UI/UX probes (U):** a design-system/visual-consistency pass on the feature surface, a component-state sweep (default/hover/active/focus/disabled/loading/error), a responsive/RTL render check at the target breakpoints, an accessibility pass (keyboard, focus order, labels, contrast, tap-target), and an empty/error/loading-state review. Each is a row with an annotated-screenshot evidence requirement.
- **Performance probes (P):** key-action latency with **repeat runs (cold + warm)**, perceived-responsiveness reads, payload/asset weight, and a slow-network behavior case — each a row with the raw measurement + how it was measured.

UI/UX and Performance probes are **observation-only**: they generate ranked, business-impact-tagged follow-ups, never a verdict downgrade (unless the user promoted the lens to gating for that run, or a UI/UX defect makes a capability impossible to complete — which is a Functional failure caught by the F lens).

### Case budget — exhaustive by default, but governed

Default depth is **exhaustive**: every capability gets its full mandatory floor and its relevant edges, all four lenses. Exhaustive does not mean infinite — govern it so the run is honest and finite:

- **Mandatory floor (never dropped):** for every capability — at least 1 positive case, **every** negative case its guards imply, and the UI/UX + Performance probes relevant to that capability.
- **Discretionary edges:** add edge cases by the priority order below until they stop being relevant to the capability. Exhaustive-by-default keeps the relevant edges; only drop the lowest-value ones if the user asked to trim at the scope gate.
- **Quick run:** when the user asks for a quick pass, collapse to positives + mandatory negatives + a single timing read + a top-level UI/UX glance, and say so.
- **Honest truncation:** whatever is capped or trimmed, `log` exactly what was not run and why. Truncation must never read as full coverage. Scaling **down** is the user's choice at the scope gate; scaling **up** is the default.

Edge-case priority order (keep from the top while edges remain relevant):

1. boundaries that change persisted data or money/quantity (min, max, just-over-max, zero)
2. empty / missing-required (when not already a negative)
3. very long / special-character / unicode on a stored or displayed field
4. interrupted flow (back, refresh, expired-session re-entry) on a multi-step capability
5. list cardinality (zero / one / many; first / last) when the capability renders a list
6. locale / timezone boundaries when the capability is date/number/currency-sensitive
7. concurrency / double-submit / replay when the action is non-idempotent

Apply an edge only when it is *relevant* — no timezone case on a capability with no dates.

### The matrix grows mid-run (logged, traced, budget-governed)

The matrix is **not frozen** after discovery. Execution extends it whenever testing reveals what discovery (Option A, non-destructive) could not see:

- a hidden state/branch that appears only after a real action (submit reveals a confirmation step; an error reveals a retry path)
- a negative case that exposes a new guarded route or sub-flow
- a real defect that implies a sibling case worth checking ("max-length truncates on save → check it on edit too")

Every added row obeys three rules: it is **`log`ged** ("added `C4-E3` — confirmation step found after submit"); it **traces to a capability** (no orphan padding); and it stays **under the agreed budget** — if a discovery would blow past the scope-gate size (e.g. a whole sub-feature behind a button), Sara surfaces it ("found 12 more cases behind export — run them?") rather than quietly ballooning. The final coverage count always reflects what actually ran — never a silent expansion and never a silent cap.

**Enforcement — the addition is not real until it is written into `matrix.md`.** It is not enough to note a discovered case in the run log or chat; a mid-run addition must be appended to `matrix.md` under a dedicated **`## Mid-run additions`** section as a properly numbered row (`C<n>-<class><k>`), each row carrying:

- the **lens tag(s)** (F / U / P / S),
- the **trigger** — the finding or discovery that spawned it ("BUG-1 implies: does the cloud ignore source tabs too?"),
- the **outcome** — what running it produced (pass / fail / confirms BUG-N / observation).

A discovered sibling case that is executed but **not** written back into `matrix.md` is a process violation of this skill: the artifact would then under-report what actually ran. At the end of the run, the `## Mid-run additions` section plus the original matrix must together account for **every** case executed — reconcile them before writing the verdict. Recording a mid-run case only in `results.md`/chat is the loose-logging failure this rule exists to prevent: write the row, then record the result.

### Destructive-case safety

Any case flagged `destructive?` (mutates persisted state — create/update/delete, double-submit, replay, concurrency, max/over-max writes, money/quantity boundaries) must use dedicated test data or a reversible action and clean up after itself. Never run a destructive case against shared or production-like data without a reset path. Record any data left behind in the run follow-ups.

## The Four Deep-Lens Procedures

This is where the matrix is executed. Each lens has its own procedure, its own evidence rules, and its own way of producing findings. **Functional and Security gate the feature verdict; UI/UX and Performance produce ranked follow-ups.** A single interaction usually serves several lenses at once (a submit case is functional + UI/UX + performance + security) — run it once and capture every lens's observation from that one interaction; only the dedicated lens probes are separate rows.

All four lenses obey the shared **Locator & Evidence Rules** (Playwright for web, Appium for mobile; locator-first; screenshots only to diagnose a locator failure or to evidence a confirmed defect; annotate every defect screenshot with a red rectangle before it is attached). Drive interactions through locators, not screenshots-to-decide.

### Lens 1 — Functional (verdict-gating, the spine)

Functional execution is the source of truth for the verdict. It runs every Positive, Edge, and Negative case in the matrix and judges each against its oracle. Optimize for: *does this capability produce the right outcome — and guard the wrong ones — across its expected, boundary, and failure cases, with persisted state left correct?*

**Per-case execution loop.** For each functional case, grouped by capability:

1. **Set up the precondition / input** using locator-first interaction (reuse the verified feature map fast path; learn locators on first contact). For a `destructive?` case, confirm the test-data fixture and cleanup path are in place *before* acting.
2. **Perform the action** through the real control the user would use — not a shortcut, not a direct API call (API behavior is the Security lens's captured-request job, not a substitute for driving the UI here).
3. **Wait for the expected next state**, then **read just enough state** (a locator assertion or one targeted read) to judge the oracle. Do not snapshot the whole screen to "see if it worked."
4. **Judge against the oracle:**
   - *Positive / Edge* — passes only when the observed outcome matches the expected result. `screen loaded`, a spinner, or "no error" is **not** a pass.
   - *Negative / abuse* — passes only when the action is correctly **rejected or guarded**: the right validation message, blocked navigation, unchanged data, a safe error. A guard that is missing, partial, or silently lets the action through is a **FAIL**.
5. **Verify data integrity after mutating actions.** For any case that writes state, confirm the persisted result is actually correct — the saved value matches the input (no silent truncation, no encoding corruption, no wrong record), the right row changed and only that row, and a re-read/refresh shows the same thing. A capability that "succeeds" in the UI but persists the wrong data **FAILS**.
6. **Confirm failures before recording them.** Before recording a `FAIL` on a negative/guard case (consequence: "the guard is missing") or on a data-integrity failure (consequence: "data is corrupted"), confirm it with a **second clean observation** — these are high-consequence verdicts and must not rest on one possibly-flaky read. A `FAIL` on a routine positive/edge case does not need a second observation, but note any flakiness seen.
7. **Capture the other lenses riding on this interaction** (UI/UX read, timing, guard observation) per their own procedures — one interaction feeds all the lenses it is tagged with.
8. **Record a per-case verdict:** `PASS`, `FAIL`, `BLOCKED (setup)`, or `NOT RUN (out of scope)`, with the observed result, the oracle it was judged against, per-lens notes, and an evidence reference. Use `BLOCKED (setup)` (never a silent drop) when a case can't run for lack of data, role, or a feature-flag state.
9. **Clean up** any data the case mutated; record anything left behind.

**State, transitions, and flow integrity.** Beyond single inputs, the Functional lens checks the capability's behavior *as a system*:

- **State transitions** — every state the capability can enter (loading → loaded → empty → error → success) is reachable and leaves the UI consistent; no dead-ends, no stuck spinners, no state that can't be exited.
- **Idempotency / double-submit** — non-idempotent actions don't double-apply on a second click or a fast re-submit.
- **Concurrency** — where relevant, two near-simultaneous actions resolve to a correct, consistent result (no lost update, no duplicate).
- **Interrupted flow** — back / refresh / expired-session re-entry mid-flow recovers correctly or fails safe; no partial commit, no corrupted state.
- **Error handling** — induced errors (invalid input, server error if safely inducible, lost connectivity) produce a correct, recoverable error path, not a crash or a swallowed failure.
- **Cross-capability consistency** — a result produced by one capability is reflected correctly where another capability reads it (create here shows up in the list there).
- **Feature-level E2E & integration (downstream effects)** — drive the feature's *own* full path end to end (entry → every step → exit) on the happy path and the main alternate paths, and confirm the **downstream side effects actually happened**, not just that the UI said "success": the record is created/updated/deleted in the backend, the email/notification/webhook fires, the event is emitted, the search index / list / counter / another system reflects the change, and a re-read or a second surface confirms it. The feature's real backend/API/3rd-party touchpoints must behave correctly through the whole action, not only at the first response. This is intra-feature E2E — it is in scope. A **multi-feature, whole-journey E2E** (e.g. sign-up → onboard → purchase → receipt email → account state) is *not* this skill's job: cover the feature's own end-to-end behavior and its integration points here, and hand off the cross-feature journey to `journey-test`, saying so.

**Per-capability functional verdict.** After all functional cases for a capability run, assign one verdict from the **functional (and the capability's own security guards) cases only**:

- `HOLDS` — all positive/edge cases pass and all negative cases are correctly guarded, with persisted state correct.
- `PARTIAL` — the capability functionally works but at least one edge/negative case is weak, or a non-blocking functional defect exists.
- `FAILS` — a positive case fails, a negative case is not properly guarded, or a mutating case corrupts data.
- `UNVERIFIED` — could not be exercised (blocked setup, missing data/role, or every case `NOT RUN`). State the reason. A capability is never left without one of these four.

**Defects → handoff.** When a functional case confirms a defect worth tracking, hand it to `bug-report` (link to Jira when tracked) — do not invent ad-hoc bug formatting. A UI/UX or performance observation seen during a functional case is recorded as a follow-up for those lenses, not as a functional defect, *unless* it makes the capability impossible to complete (then it is a Functional `FAILS`).

### Lens 2 — Security (verdict-gating, deep)

Security goes beyond the feature's own guards into systematic probing of the surface the feature exposes. Optimize for: *can this feature be made to do something it must not — act for the wrong user, accept what it must reject, leak what it must protect, or be reached without the right identity?* A guard that fails to hold is a **verdict-deciding failure**, ranked with the same weight as a functional FAIL.

**Authorization gate (run this first).** Security testing is only legitimate against a target the user is authorized to test. Before any probing, confirm this is an authorized environment (staging/test build or explicit owner authorization) and stay inside that scope. Probe **only** the feature under test and the accounts/roles provided — never pivot to other tenants' real data, never attempt to exfiltrate real PII, never run a denial-of-service or destructive exploit. The goal is to *prove a guard's presence or absence*, not to weaponize a gap. If probing would require acting outside the authorized scope, stop and report the gap as a finding rather than crossing the line. Anything that escalates into a full systematic penetration effort beyond this feature's surface hands off to `journey-sec`, which carries the broader sensitive-handling framing.

**What the lens executes.** Run the matrix's N-cases that are tagged `S`, plus dedicated security probes, across these dimensions:

- **Authorization & role boundaries** — for each capability with a permission guard, attempt it as the **wrong role** and as **anonymous**. The guard must block the action *and* not leak the protected data in the error. Map which roles can reach which capabilities; a capability reachable by a role that must not have it is a hole.
- **IDOR / object-level authorization** — where the feature addresses a record by id (URL param, hidden field, request body), attempt to read/modify **another object the current user must not access** (a neighbouring id, another user's resource). Use only test fixtures, never real third-party data. Unchanged-and-blocked is the pass; any cross-object read/write is a hole.
- **Forced / tampered navigation** — direct-navigate to a guarded step or state without going through the gate (deep link to an admin sub-view, skip a required prior step). The guard must redirect or reject.
- **Input validation / injection (authorized target only)** — send injection-shaped strings (SQL-ish, script/XSS-shaped, template/`{{}}`, path traversal, oversized, control characters) into the feature's inputs as **validation cases**: the input must be safely rejected, escaped, or neutralized — never reflected unescaped, never executed, never error out with an internal stack/SQL detail. This proves input handling; it is not an exploitation attempt.
- **Captured API request/response validation** — for the feature's key actions, capture the underlying request and response (network panel via the browser tooling) and inspect: is auth actually enforced server-side (not just hidden in the UI)? does the response leak fields the user shouldn't see (other users' data, internal ids, secrets, tokens, verbose errors)? is `user_id`/`owner_id`/`role` trusted from the client (an IDOR/privilege shape)? does a guarded action still 200 when called with a wrong-role token? The server is the real boundary — a UI guard with no server guard is a hole.
- **Session & token handling** — session fixation/expiry behavior on the feature's sensitive actions, whether a stale/expired session is correctly rejected, whether tokens leak into URLs/logs/storage, and whether logout/expiry actually revokes access to the feature.
- **Sensitive-data exposure** — does the feature expose more than it should in the page, the response, error messages, or client storage (PII, secrets, internal identifiers, debug detail)?

**Confirmation & evidence.** A security hole is a high-consequence verdict — confirm every positive finding (a guard that *failed*) with a **second clean observation** before recording it. Capture evidence: the request/response (with secrets redacted), the locator/route used, and an annotated screenshot where visual. 

**Sensitive-finding handling.** When a probe surfaces a real authorization or data-exposure hole:

- flag that finding's evidence `sensitive: true`
- keep its exploit details **out of the chat summary and the HTML report body** beyond a non-sensitive pointer ("authorization hole on capability C3 — see sensitive evidence")
- store full detail only in the run's sensitive evidence, redact secrets/PII in any captured payload
- hand off to `bug-report` for the tracked write-up and recommend `journey-sec` if the hole suggests a systemic, beyond-this-feature problem
- do **not** keep probing or escalating it under feature-deep-test once confirmed — prove it once, secure the evidence, hand off

**Per-capability security verdict.** Per capability, from the security cases:

- `HOLDS` — every guard blocks correctly; no IDOR, no forced-nav bypass, no injection reflection/execution, no server-side authz gap, no sensitive leak.
- `PARTIAL` — guards hold but with a weaker-than-ideal signal (e.g. the action is blocked but the error leaks a record's existence, or validation rejects but with an over-detailed message).
- `FAILS` — any guard is missing or bypassable, any cross-object access succeeds, any injection is reflected/executed, or the server doesn't enforce what the UI implies.
- `UNVERIFIED` — could not probe (missing roles/fixtures, no safe way to capture the request). State the reason; never leave a guarded capability without a security verdict.

Security `FAILS` and `PARTIAL` feed the feature verdict alongside Functional. A capability with a confirmed security `FAILS` cannot roll up to a verified feature.

### Lens 3 — UI/UX (observation-only, deep)

UI/UX judges both halves: **UI** (the visual/interface craft) and **UX** (usability and flow). It is deep — a full heuristic and craft pass on the feature surface, not a glance — but **observation-only**: it produces ranked, business-impact-tagged follow-ups and **never gates the feature verdict**. The single exception is not a UI/UX downgrade at all: a layout or interaction defect that makes a capability **impossible to complete** is a *Functional* `FAILS`, caught by the F lens.

Load the **shared UI/UX-lens definition** first so this stays consistent with the rest of Sara: use the per-project override `.sara/ux-lens.config.json` if present, else the bundled `.sara/ux-lens.config.example.json`. The dimensions below are the floor; the config may extend or tighten them.

**UI — visual / interface craft.** Across the feature's surface and every state it reaches:

- **Visual design & layout** — alignment, spacing/rhythm, grid integrity, visual hierarchy; no overlap, clipping, or overflow; no truncated or colliding text.
- **Typography & color** — readable type scale and hierarchy; sufficient **contrast** (WCAG AA as the bar); no hardcoded off-palette colors.
- **Design-system consistency** — the feature uses the project's tokens/components and matches sibling screens; flag bespoke one-offs that drift from the system.
- **Component states** — sweep every interactive element through **default / hover / active / focus / disabled / loading / error**; each state must be visible, distinct, and correct (a focus ring exists, disabled looks disabled, loading shows progress, error is styled).
- **Responsive & RTL** — render at the target breakpoints including **mobile width (~390px)** and confirm the layout holds; for bilingual products, check **Arabic / RTL** integrity — mirrored layout, logical properties (no physical `ml/pl` leakage), correct text direction, no broken icons/arrows.
- **Visual-polish / AI-slop tells** — placeholder or generic copy left in, lorem ipsum, no hierarchy, default unstyled components, inconsistent corner radii/shadows, fake affordances (things that look clickable but aren't).

**UX — usability / flow.** Walked through the project persona, plus a couple of stress archetypes (a first-timer, a hurried power user):

- **Nielsen 10 heuristics** — system status visibility, match to the real world, user control & freedom (undo/cancel/back), consistency & standards, error prevention, recognition over recall, flexibility, minimalist design, good error messages, help/documentation where needed.
- **Accessibility (WCAG)** — keyboard reachability and operability of every action, logical **focus order**, visible focus, present and correct labels/alt text/ARIA roles, **tap-target size**, and that nothing is color-only signaling.
- **Cognitive walkthrough** — at each step: is the next action obvious? does the user know what happened after they acted? can they recover from a mistake or a negative path?
- **Empty / error / loading states** — each exists, is understandable, and tells the user what to do next (an empty state that just shows nothing is a finding).
- **Feedback & recoverability** — every action produces clear feedback; destructive actions confirm; the user can get back out of any state.
- **Cognitive load** — is the feature asking the user to hold too much in their head, parse a wall of controls, or make an unguided choice?

**Execution & evidence.** Capture UI/UX observations *during* the functional interactions wherever a case already drives that surface (one interaction, many lenses), and run the dedicated UI/UX probe rows for the cross-cutting passes (component-state sweep, responsive/RTL, accessibility, state review). For every confirmed UI/UX problem, capture a screenshot and **annotate the problem area with a red rectangle** before it is recorded or attached (per Sara's evidence rules) — a UI/UX finding without annotated visual evidence is incomplete. Accessibility findings that aren't visual (focus order, keyboard trap, missing label) record the interaction sequence and what failed.

**Output — ranked follow-ups, not a verdict.** UI/UX does not produce HOLDS/FAILS. It produces a **ranked list of findings**, each with: the dimension, the surface/state, the annotated evidence, and a **business-impact tag** (`blocker-adjacent` / `high` / `medium` / `low`) reflecting how much it hurts the user's ability to get value — not just how ugly it is. `blocker-adjacent` means "users can technically complete the task but many will fail, abandon, or mistrust it." These feed the report's follow-up section and `bug-report` for the ones worth tracking; they never flip the feature verdict on their own. If the user **promoted UI/UX to gating** for this run, then (and only then) `blocker-adjacent` findings roll into the verdict like a functional PARTIAL/FAILS — state that this promotion is in effect.

### Lens 4 — Performance (observation-only, deep)

Performance measures how the feature actually *feels and behaves under load of use* — with real numbers, not vibes. It is deep (repeat runs, multiple signals, degradation points) but **observation-only**: it produces ranked, business-impact-tagged follow-ups and **never gates the feature verdict** — unless the user promoted it, or the feature itself states an explicit performance requirement (then that requirement is a *Functional* case with a real oracle, judged by the F lens, not a soft reading here).

**Measurement discipline (non-negotiable).** Every reading is a **raw number with how it was measured** and the conditions it was taken under. No reading is reported as a judgment without its method.

- **Repeat runs, cold + warm.** Take each key measurement on a **cold load** (first visit / cache cleared) and on **warm** (repeat), and run each at least **3 times**, reporting the **median** (and the spread when it's noisy). One reading is an anecdote; the median of repeats is data.
- **Record conditions** — environment/URL, device or viewport, network condition, auth state, and what was on screen. The same action measured under different conditions is a different reading.
- **Measure the real action** — wall-clock around the actual locator interaction, plus browser timing where available (navigation/resource timing, paint metrics). On web, use the browser timing APIs and the network panel (and the `chrome-devtools` MCP when deeper request/trace detail helps). On mobile, time the real gesture-to-rendered-state interval.

**What the lens measures.** Run the matrix's Performance probe rows plus the timing tags riding on functional cases:

- **Key-action latency** — for each meaningful action the feature owns (open, load, search, filter, submit, save, export), the time from action to the *usable result state* (not just first byte, not just spinner-shown). This is the headline number.
- **Initial load / time-to-usable** — how long until the feature is actually interactive on cold and warm loads.
- **Perceived responsiveness** — does the UI acknowledge the action immediately (optimistic feedback, skeleton, progress) even if the result takes time? A 2s action that feels instant beats a 1s action that feels frozen — record both the real number and the perceived behavior.
- **Payload & asset weight** — the bytes the feature pulls for its key actions (response sizes, image/asset weight, obvious over-fetching or N+1 request fan-out visible in the network panel).
- **Slow-network behavior** — re-run the key action under a throttled/slow-network condition: does it degrade gracefully (progress, partial render, timeout-with-retry) or hang/break? This is where features that "feel fine on fast wifi" expose themselves.
- **Jank & long tasks** — visible stutter, layout shift during load, long-running main-thread work that freezes interaction during the key action.
- **Degradation points** — where the feature gets slow with realistic scale (a long list, many filters applied, a large input, repeated rapid actions). Note the point at which it stops feeling acceptable.

**Reference bands (context, not a gate).** To make a raw number meaningful, compare it against common UX-perception bands and *say which band it falls in*, without converting that into a pass/fail: **<100ms** feels instant; **~100ms–1s** feels responsive (keep feedback flowing); **~1–3s** is noticeable, needs a progress affordance; **>3s** risks abandonment; **>~8–10s** reads as broken. A reading lands in a band — it does not "fail" unless the feature declared an explicit target. Use the band to set the **business-impact tag**, not to manufacture a verdict.

**Output — ranked follow-ups with numbers.** Performance produces a **ranked list of findings**, each with: the action measured, the **median + cold/warm + conditions + method**, the perception band, and a **business-impact tag** (`blocker-adjacent` / `high` / `medium` / `low`) — `blocker-adjacent` meaning "slow enough that users abandon or distrust the feature." Track the worst ones via `bug-report`. They never flip the feature verdict on their own (unless promoted, or tied to an explicit perf requirement, which is then a Functional case). When the work needs real depth beyond this feature — multi-run trend baselines, profiling flame charts, regression-over-time tracking, or load testing — **hand off to `journey-perf`** and say so; feature-deep-test measures this feature deeply but is not a load-testing or profiling rig.

## Cross-Cutting Coverage

Two concerns run *across* the four lenses rather than forming their own lens. They are not optional add-ons — when they apply, the matrix must include their cases, woven into the lenses below.

### Localization (cross-cutting: Functional gates, UI/UX observes)

For any user-facing capability, localization is mandatory coverage — especially on bilingual products. Drive it from the project's locale rules in `.sara/project-context.md` (languages, default locale, RTL requirement, date/number/currency formats). It splits across two existing lenses by tag:

- **Correctness — Functional (F), verdict-gating.** Both languages render the right content (no missing translation keys, no raw `_en`/`_ar` field names leaking, no untranslated fallback strings, no mojibake); **locale formatting** of dates, numbers, currency, and pluralization is correct for the active locale; switching locale updates the feature live and persists the choice where expected. A user-facing capability that shows broken/missing content in a supported language is a Functional defect, gated like any other.
- **Layout — UI/UX (U), observation-only.** **RTL/Arabic integrity** (mirrored layout, logical properties, correct direction, non-broken icons/arrows), and **text-expansion/truncation** when one language is materially longer than the other (clipped buttons, overflowing labels). Reported as ranked follow-ups with annotated evidence, same as any UI/UX finding.

Run each user-facing capability in **every supported locale** (at minimum the default + one RTL locale on bilingual products), and surface localization as its own section in the report so it is never silently skipped. If the project is single-locale, state that and run the default only.

### AI-powered features (conditional handoff, never crammed into the 4 lenses)

If the feature under test is **GenAI-powered** (an LLM/agent generates the output — a summary, answer, chat reply, recommendation, generated content, or an agent that takes actions), its *output quality* is a different discipline: non-deterministic, judged against quality contracts over many cases with reliability rates, plus hallucination/grounding/bias. **Do not** force AI-output judgment into a single-oracle functional case.

Instead:

- **Detect it during discovery** and say so. Flag the feature as AI-powered in `feature.md`. First check `.sara/knowledge/docs/feature-evaluations/` for an existing **AI Feature Quality Canvas + evaluation** for this feature or sub-feature — if one exists, reuse its quality contracts and findings and cross-link the bundle (`related_items`) instead of commissioning a fresh evaluation.
- **Route the AI-output evaluation** to the skill built for it: `evaluate-genai-feature` for a GenAI feature's outputs, or `evaluate-agent` for an action-taking agent. feature-deep-test integrates/hands off that evaluation rather than improvising one.
- **The four lenses still test the deterministic shell** around the AI: the input control works and validates; the response renders correctly (incl. RTL/localization and empty/error/timeout states); the endpoint is **authenticated, authorized, and rate-limited**; latency and slow-network behavior of the generation call; and the UI's handling of a slow or failed generation.
- **The Security lens already owns prompt-injection** as an input-validation case (a crafted input must not exfiltrate the system prompt, escape its instructions, or leak other users' data) — keep that here; deep systematic jailbreak probing hands off to `journey-sec`.

Net: feature-deep-test tests *everything around* the AI deeply, and routes *"does the AI itself behave correctly"* to the evaluation skill — reporting both together so the feature gets one combined picture.

## Locator & Evidence Rules

These are the same hard rules as `journey-test` / `story-test` and the project Sara/Appium rules. They are non-negotiable across all four lenses.

**Locator-first interaction.**

- **Web:** drive every viable navigation and interaction through **Playwright MCP locators** first, in a **live headed browser** (`headless: false`) kept visible during the run. Do not silently fall back to headless unless the user asks for it. Point any Selenium MCP at the same remote browser session. The `chrome-devtools` MCP may be used when DevTools/network/trace detail helps (Performance and the Security captured-request work).
- **Mobile (native):** drive every viable navigation and interaction through **Appium MCP locators** first (`appium_find_element` + `appium_gesture`), preferring accessibility id / id over xpath.

**First-run locator learning + fast path.** On first contact with an element, learn the tightest stable locator from observed node metadata, **validate it resolves to exactly one actionable element** before saving it to the feature map, then reuse the saved map on later runs (the verified-map fast path). Recompute the feature fingerprint at the start of every run and re-learn on a mismatch (see Artifact Contract). Apply the same **Snapshot/Evaluate Economy** as `journey-test`: batch read-only reads, do not snapshot the whole screen for every passing case.

**Screenshots are evidence, not eyes.** Do **not** take screenshots to read screen state or decide the next action — that is the accessibility snapshot's job. Screenshots are allowed only when:

- (a) a locator fails and a screenshot is needed to **diagnose** the failure (weak semantics, icon-only button, position-driven layout — inspect the visible UI before any coordinate fallback), or
- (b) a case fails, or a UI/UX finding is confirmed, and the screenshot is **evidence** for the defect.

Use coordinate clicking only as a last resort after visual inspection, never as the default.

**Annotation (mandatory for defect screenshots).** Before any defect screenshot is recorded or attached to Jira, **annotate the confirmed problem area with a red rectangle** using a local saved-file annotation method that works in the active runtime. Attach **only the annotated version**. Delete the original unannotated screenshot **only after** the annotated replacement exists successfully. If annotation fails, **stop and report the blocker** rather than attaching an unannotated image.

**No guessing URLs.** Do not guess page/route URLs when the product can reveal them. Prefer, in order: URLs the user provided, URLs stored in project context, URLs reached through **real navigation** in the running app. Only fall back to an assumed route if navigation cannot reveal it, and state that fallback briefly.

**Auth / SSO.** When a flow needs Google SSO (or similar interactive login), open the login URL in the headed window, ask the user to complete login **once** manually, then continue in that same session context for the rest of the run.

**Negative-case oracles via locators.** For negative/guard cases asserting an action is rejected, the oracle is the observed guard (validation message, blocked navigation, error state, unchanged data) confirmed via a **locator assertion or one targeted read** — not a screenshot. Screenshots only back the *evidence* once the guard fails.

## Artifact Contract

Every feature-deep-test is self-contained under the workspace knowledge base. feature-deep-test owns and writes **only** under `.sara/features/<slug>/`.

```text
.sara/features/<feature-slug>/
  feature.md                         # capability inventory + oracle (source of truth for the run)
  matrix.md                          # the derived scenario matrix (capability -> cases, lens tags)
  maps/
    web.yaml                         # locator map (same schema as journey/story maps)
    android.yaml
    ios.yaml
  runs/
    feature-deep-test/<platform>/<timestamp>/   # per-run evidence, results, screenshots, sensitive/
  .lock                              # advisory concurrency lock (see below)
```

`feature.md` and `matrix.md` are the human-readable source of truth; any runnable structure is derived in memory. Create missing folders on demand. Sensitive security evidence (from `sensitive: true` findings) lives in a `sensitive/` subfolder of the run and is referenced by a non-sensitive pointer elsewhere.

**Locator map schema.** Reuse `journey-test`'s **Map Shape** exactly — the same locator-map schema, `map_status` values (`partial` / `verified` / `stale`), persona/role scoping, locator-correctness checks, and verified-map fast-path logic. Do not invent a separate schema. The one substitution: the feature map is fingerprinted against the **feature inventory, not a journey**.

- It carries `feature_source: feature.md` and a `feature_fingerprint` computed the same way as journey/story fingerprints — **sha256 of the normalized `feature.md` plus `platform` plus normalized `environment`** — with `pending_feature_fingerprint` used while a change is being reconciled.
- `matrix.md` is **not** part of the fingerprint: the matrix is derived and re-tuned per run (case depth, budget, mid-run growth), so folding it in would needlessly invalidate the map on every depth change.
- A feature map is `verified` only when all required targets for the current `feature_fingerprint`, platform, environment, and active role pass the locator-correctness check. Recompute and compare `feature_fingerprint` at the start of every run; treat a mismatch the way `journey-test` treats one (re-learn, keep `partial`/`stale`).

**Seeding from existing maps.** An existing `.sara/journeys/<slug>/maps/<platform>.yaml`, `.sara/stories/<slug>/maps/<platform>.yaml`, or **`.sara/regression/suites/<suite>/maps/<platform>.yaml`** for the same screens may be **read** to seed first-run locator learning (copy applicable targets, record the source). **Never write back** into a journey, story, or regression folder from feature-deep-test, and never treat their map as the live map for a feature run — heals discovered during a feature-deep-test are written only to the feature map. This keeps two skills from healing the same file under different lock scopes.

**Concurrency lock.** Use the same lock *mechanics* as `journey-test`/`story-test`, with a feature-scoped trigger:

- Before any write to `.sara/features/<slug>/`, acquire an advisory lock by creating `.sara/features/<slug>/.lock` with `{ "mode": "feature-deep-test", "started_at": <ISO-8601>, "pid_or_session": <id> }`.
- If the lock exists and is **fresh** (started within the last 30 minutes), refuse to start: `Feature <slug> is locked by another active feature-deep-test run.` Do not proceed.
- If the lock exists but is **stale** (older than 30 minutes), reclaim it (an earlier run aborted without releasing).
- Release the lock at the end of the workflow after the run file and index are written, and on every abort path.

This lock is independent of any journey/story lock and never touches their folders. Because feature-deep-test only *reads* journey/story maps, a feature run and a journey/story run on related screens cannot corrupt each other.

**Index entry schema.** feature-deep-test maintains its own `features` array in `.sara/index.json`, parallel to `journeys` and `stories`. Add or update one entry per feature slug whenever a feature folder is created, run, or archived:

```yaml
features:
  - id: export-to-pdf                 # slug
    type: feature
    title: Export to PDF
    source: "live"                    # "live", a KB ref, a Jira key/URL, or "pasted"
    feature_area: reporting
    platforms: [web]                  # any of web, android, ios
    environment: staging
    roles: [member, anonymous]        # roles exercised across runs
    locales: [en, ar]                 # locales exercised (localization cross-cutting class)
    ai_powered: false                 # true -> evaluate-genai-feature/evaluate-agent handoff was used
    lenses: [functional, security, ui_ux, performance]   # lenses run in the last run
    last_run_at: 2026-06-16T12:00:00+03:00
    last_verdict: FEATURE PARTIALLY VERIFIED   # FEATURE VERIFIED | FEATURE PARTIALLY VERIFIED | FEATURE NOT VERIFIED
    capabilities_total: 9
    coverage:                         # counts of per-capability verdicts from the last run
      holds: 6
      partial: 2
      fails: 1
      unverified: 0
    cases:                            # case totals from the last run
      total: 74
      passed: 61
      failed: 5
      blocked: 3
      not_run: 5
    followups:                        # non-gating findings carried out of the run
      ui_ux: 7
      performance: 4
    map_status: verified              # partial | verified | stale
    path: .sara/features/export-to-pdf/
```

Omit `roles`/`locales` when none were varied. `last_verdict`, `coverage`, `cases`, and `followups` always reflect the most recent run.

## Environment Readiness

Verify the matching environment **silently** before executing — the same checks as `journey-test` / `story-test`. Surface only a failed check and the next action, in beginner-friendly language; do not push setup detail unless the user asks for help.

**Project plumbing (first run in a workspace).**

- Ensure `.sara/` exists (create the scalable structure if not) and that `.sara/` is in the project `.gitignore`.
- Ensure `.mcp.json` exists; if missing, create it from `.mcp.json.example` (project root → `~/.claude/skills/sara/.mcp.json.example` → `~/.codex/skills/sara/.mcp.json.example`). If it exists, merge in any missing baseline servers (`atlassian`, `appium`, `playwright`, `chrome-devtools`, `testrail`, `jira`) without overwriting project-specific settings.
- For any Jira fetch/create/update/transition/attachment/comment, use the **workspace `.mcp.json` Jira config first**; only fall back when it is unavailable, and state the fallback.

**Web (Playwright-first).**

- `node` and `npx` available; the Playwright MCP can start; browser binaries installed; a **live headed window can open**.
- Keep the headed browser visible for the whole run. If a Selenium MCP is present, point it at the same session.
- For Google SSO (or similar), open the login URL in the headed window and have the user log in **once**, then continue in that session.
- The `chrome-devtools` MCP is available when the Performance and Security lenses need network/trace detail.

**Mobile (Appium-first).**

- Read `.mcp.json`, confirm the `appium` entry, extract `APPIUM_HOST` / `APPIUM_PORT`, and run the Android readiness checks from the **mobile branch of `bug-test`**.
- If an APK is available, extract the package id from the APK first, then check whether the app is already installed on the connected device and **skip reinstall** when it is.
- Use `appium_skills` for local install/doctor questions.

If setup is incomplete, report only the missing item and the next step — then stop until it is resolved. Do not begin discovery or execution against an environment that is not ready.

## Workflow

1. **Resolve the feature and inputs.** Identify the feature, target environment/URL, platform, roles, test data + cleanup path, depth (exhaustive default), focus/priority steer, and scope limits. If a Jira key/URL was given as the source, fetch it first.
2. **Resolve the feature slug.** Fuzzy-match against existing `.sara/features/` slugs; on a match, ask whether to rerun or fork. Optionally read a related `.sara/journeys/<slug>/` or `.sara/stories/<slug>/` map to *seed* first-run learning (read-only).
3. **Acquire the feature-folder lock** (`.sara/features/<slug>/.lock`, feature-scoped trigger and message). Refuse to start if another active feature-deep-test holds a fresh lock; reclaim a stale one. Release on completion and on every abort path.
4. **Verify environment readiness** for the platform (Section 7). For web, open and keep a live headed browser. Stop if setup is incomplete.
5. **Discovery — Pass A (context read).** Gather the oracle from `.sara/knowledge/` (incl. `.sara/knowledge/docs/feature-evaluations/` for any existing AI Feature Quality Canvas + evaluation), `.sara/regression/` (existing agentic regression suites for this feature — reconcile already-covered cases, seed maps), `.sara/experience/`, prior runs, and project context. If empty, note "case design unguided by project quality context."
6. **Discovery — Pass B (live capability map).** Navigate to the feature and inventory its full side-effect-free surface locator-first (Option A: non-destructive). Detect whether the feature is **AI-powered** and flag it.
7. **Reconcile → write `feature.md`.** Produce the numbered capability inventory (surface, oracle, inputs, guards implied, destructive flag, roles, locales, source). Establish the locator map (`maps/<platform>.yaml`), learning on first contact; recompute `feature_fingerprint`.
8. **Build the matrix → write `matrix.md`.** Expand each capability into Positive / Edge / mandatory Negative cases with lens tags, plus dedicated UI/UX and Performance probe rows, plus Localization cases (and the AI-shell cases if AI-powered). Apply the exhaustive-by-default budget with the mandatory floor.
9. **Scope gate.** Surface the capability count and proposed matrix size; let the user confirm, trim, focus, or say "run all." Honor the focus/priority steer (reorders cases; never removes coverage). Skip the gate only for an explicit quick run.
10. **Establish auth/roles** for the first block of cases; keep wrong-role/anonymous accounts and invalid data ready for negative and security cases. Confirm test-data fixtures + cleanup paths for destructive cases.
11. **Execute the matrix, grouped by capability.** For each capability, run its cases and capture **every lens riding on each interaction in one pass**:
    - **Functional** — per-case loop (set up → act via the real control → read just enough → judge the oracle → verify data integrity on mutations → confirm high-consequence FAILs with a second observation → record verdict → clean up). Cover state/transitions, idempotency, concurrency, interrupted flow, error handling, cross-capability consistency, and **feature-level E2E + downstream side effects**.
    - **Security** — run the authorization gate first; then the S-tagged negatives and security probes (authz/role, IDOR, forced-nav, injection-as-validation, captured API request/response, session/token, sensitive-data exposure). Confirm any failed guard twice; apply `sensitive: true` handling.
    - **UI/UX** — capture observations during functional interactions and run the dedicated probes (component-state sweep, responsive/RTL, accessibility, state review). Annotate every confirmed visual finding (red rectangle). Record as ranked, business-impact-tagged follow-ups.
    - **Performance** — run the timing tags + probe rows (cold+warm, ≥3 runs, median, conditions + method; latency, perceived responsiveness, payload weight, slow-network, jank, degradation points). Record as ranked follow-ups with numbers.
    - **Localization** — run each user-facing capability in every supported locale: correctness (Functional/gating) + RTL/expansion (UI/UX/observation).
    - **Grow the matrix** when execution reveals a hidden state/branch — append each addition as a numbered row to the **`## Mid-run additions`** section of `matrix.md` (with lens tag, trigger, outcome), `log` it, trace it to a capability, keep within budget. Do not leave a discovered-and-run case only in `results.md`/chat.
12. **AI handoff (if AI-powered).** Route the AI-output evaluation to `evaluate-genai-feature` (or `evaluate-agent`), and fold its result into the feature picture. The four lenses already tested the deterministic shell; Security already ran prompt-injection.
13. **Assign per-capability verdicts** from the Functional + Security cases: `HOLDS` / `PARTIAL` / `FAILS` / `UNVERIFIED` (every capability gets one, with a reason). UI/UX and Performance never change this (unless promoted, or a UI defect makes the capability impossible to complete → Functional FAILS).
14. **Roll up to the feature verdict** (Verdict Model) and write the one-line deciding reason citing the worst capability.
15. **File confirmed defects via `bug-report`** (link to Jira when tracked); attach annotated screenshots. Sensitive findings get the redacted, pointer-only write-up and a `journey-sec` recommendation. Recommend `journey-perf` / `journey-test` where a lens reached beyond this feature.
16. **Persist the locator map — mandatory, write the file.** Materialize `maps/<platform>.yaml` on disk with every target the run actually acted on (testid/role/aria/text strategy + per-target status), not just heals; recompute and store `feature_fingerprint`; set `map_status` per the locator-correctness rules. The map is a **file**, not in-memory or prose in `results.md` — driving live locators without writing the YAML is an incomplete step. Map status is independent of case verdicts. **Do not proceed to the index update until `maps/<platform>.yaml` exists and lists the run's targets.**
17. **Save the run** under `runs/feature-deep-test/<platform>/<timestamp>/` — `feature.md` snapshot, the matrix with lens tags, per-case results with per-lens observations and evidence, per-capability verdicts, the feature verdict, UI/UX + Performance follow-ups, localization results, AI-eval result, blocked/not-run cases with reasons, map changes, fallback usage, sensitive pointers, and data left behind.
18. **Update `.sara/index.json`** under the `features` array (Section 6 schema). The `map_status` written here MUST match a real `maps/<platform>.yaml` on disk — never record `verified`/`partial`/`stale` for a map file that does not exist. If step 16 did not write the map, fix that before touching the index.
19. **Regression diff.** Compare against the most recent prior feature-deep-test runs for the same feature and platform; surface regressions (a capability dropped from `HOLDS` to `PARTIAL`/`FAILS`, a case flipped `PASS`→`FAIL`, a follow-up that worsened). If none, state "first comparable run — no regression baseline yet."
20. **Promote durable lessons.** If the same capability/case has been weak across three or more real runs over at least three distinct days, promote a heuristic into `.sara/experience/heuristics/`; if a recurring failure is product-level, add it to `.sara/experience/known-issues/`.
21. **Reporting.** Always produce the run-evidence markdown (step 17) and a concise chat summary (plus a Jira comment for Jira-backed input). **Ask whether to generate the standalone HTML report** unless the user already requested one.
22. **Release the lock** and **return** the feature verdict, per-capability coverage, case pass/fail/blocked counts, the top UI/UX + Performance follow-ups, the AI-eval result if any, the report path, and the feature folder path.

## Verdict Model

The feature verdict is a deterministic rollup, not a vibe. It is decided by the **verdict-gating lenses only — Functional and Security** — while UI/UX and Performance ride alongside as ranked follow-ups. This keeps the headline objective: a feature is "verified" when it *works and is safe*, even if it still has polish or speed follow-ups worth doing.

### Step 1 — Per-capability verdict (gating lenses)

Each capability ends the run with exactly one verdict, from its **Functional and Security** cases:

- `HOLDS` — all positive/edge cases pass, all negative cases are correctly guarded, persisted state is correct, and every security guard holds.
- `PARTIAL` — the capability functionally works and is not exploitable, but at least one edge/negative case is weak or a non-blocking functional/security defect exists (e.g. a guard blocks but leaks a record's existence; an edge case "works" but with a minor wrong behavior).
- `FAILS` — a positive case fails, a negative case is not properly guarded, a mutating case corrupts data, **or any confirmed security hole** (missing/bypassable guard, IDOR, injection reflected/executed, server-side authz gap, sensitive leak).
- `UNVERIFIED` — could not be exercised (blocked setup, missing data/role, no safe probe path, or every case `NOT RUN`). State the reason. A capability is **never** left without one of these four.

A localization **correctness** failure (Functional-tagged) counts toward the capability's verdict like any functional defect. Localization **layout** issues (UI/UX-tagged) do not.

### Step 2 — Roll up to the feature verdict

Apply this fixed rule, then add a one-line deciding reason:

- `FEATURE NOT VERIFIED` — **any** capability is `FAILS`.
- `FEATURE PARTIALLY VERIFIED` — no capability `FAILS`, but at least one is `PARTIAL` or `UNVERIFIED`.
- `FEATURE VERIFIED` — **every** capability is `HOLDS`.

The deciding reason must cite the **worst capability** that set the verdict — the failing one, or the `PARTIAL`/`UNVERIFIED` one if that is the worst — naming the case id and the **business consequence**, not just the UI symptom.

### How UI/UX and Performance relate to the verdict

By default they **do not change the feature verdict** — they are reported as ranked, business-impact-tagged follow-ups. The headline can read `FEATURE VERIFIED` while the report still lists real UI/UX and Performance follow-ups; that is correct and intended. Two exceptions, both already defined upstream:

1. **Impossible-to-complete defect** — a UI/UX or performance problem that makes a capability *impossible to complete* (e.g. a control unreachable on mobile width, an action that times out so users can't finish) is a **Functional `FAILS`**, caught by the F lens — not a soft follow-up.
2. **Explicit requirement** — when the feature itself states a UI/UX or performance requirement (e.g. "result within 2s", "must pass WCAG AA"), that requirement is a **Functional case with a real oracle** and gates like any other F case.

### Lens-promotion override (per run)

The user may **promote UI/UX and/or Performance to gating** for a specific run (e.g. a perf-critical or design-critical feature). When in effect:

- a promoted lens's `blocker-adjacent` findings roll into the capability verdict like a functional `PARTIAL`/`FAILS`,
- the report and the chat summary **state the promotion explicitly** ("Performance promoted to gating this run"),
- the default (F+S gating) resumes on the next run unless promoted again.

### Honesty rules on the verdict

- A feature is **never** `FEATURE VERIFIED` on positive cases alone when guards exist — the mandatory negatives must have run and held.
- If coverage was capped/trimmed, the verdict is reported **with its coverage scope** ("VERIFIED across the 6 capabilities run; 3 capabilities deferred — see deferred list"). A trimmed run never presents as full coverage.
- `UNVERIFIED` capabilities **block** `FEATURE VERIFIED` — unknown is not the same as safe. The best a feature with an `UNVERIFIED` capability can reach is `FEATURE PARTIALLY VERIFIED`.
- The locator `map_status` is independent of the verdict — a map can be `verified` while the feature `FAILS`, as long as Sara acted on the intended targets.

## Reporting

Three outputs, two always and one opt-in.

**Always produced:**

1. **Run-evidence markdown** under `runs/feature-deep-test/<platform>/<timestamp>/` (Workflow step 17) — the durable record: the full matrix, every case result with per-lens observations and evidence references, per-capability and feature verdicts, follow-ups, localization, AI-eval, blocked/not-run, map changes, sensitive pointers.
2. **Concise chat summary** — lead with the verdict and the deciding reason; then per-capability coverage, case pass/fail/blocked counts, the top UI/UX + Performance follow-ups, and the AI-eval result if any. For Jira-backed input, post the same summary as a **comment on the issue**. Keep sensitive detail out of both — pointer only.

**Opt-in:** the standalone **HTML companion**. Do **not** generate it by default — ask whether the user wants one (skip the question only if they already requested it). Save to `sara-reports/{FEATURE_SLUG}-feature-deep-test-report.html`. The markdown is the durable record; the HTML is the shareable deliverable.

### Chat summary style

Lead with the verdict in language a PM or lead can act on in under a minute. Name the deciding case id and the **business consequence**, not just the UI symptom:

- `FEATURE NOT VERIFIED — C3 (delete) FAILS: a member can delete another member's record by changing the id in the request; the ownership guard is missing server-side.`
- `FEATURE PARTIALLY VERIFIED — all 9 capabilities hold except C5 (PARTIAL): the export accepts a 10k-row request and silently truncates to 1k with no warning. Plus 7 UI/UX and 4 performance follow-ups (2 high).`
- `FEATURE VERIFIED across the 6 capabilities run — all guards held; 3 capabilities deferred at the scope gate (listed). 4 UI/UX follow-ups, none blocking.`

### HTML companion contents

When generated, the HTML report should be interactive, visually rich, RTL-aware, and print-friendly, with:

- **Header** — feature name + source, platform, environment, target role(s), locales exercised, lenses run (and any lens promoted to gating), feature folder path, locator map used + `map_status`.
- **Verdict banner** — the feature verdict with the one-line deciding reason, color-coded (VERIFIED / PARTIALLY VERIFIED / NOT VERIFIED), with the coverage scope when trimmed.
- **Capability coverage table** — one row per capability with its verdict (`HOLDS`/`PARTIAL`/`FAILS`/`UNVERIFIED`) and case counts by **class** (positive / edge / negative), summing to the capability total.
- **Case table** — every case: id, capability, class (P/E/N), **lens tags** (F/U/P/S), intent, input/precondition, expected oracle, observed result, verdict, evidence link. Filterable by capability, class, lens, and verdict.
- **Security section** — guards tested and whether each held; negative/abuse cases called out explicitly; **sensitive findings shown as non-sensitive pointers only** (detail stays in `sensitive/`).
- **UI/UX follow-ups** — ranked, business-impact-tagged, each with its **annotated screenshot** (red-rectangle evidence) and the dimension/state.
- **Performance follow-ups** — ranked, each with the **raw median + cold/warm + conditions + method**, the perception band, and the business-impact tag.
- **Localization section** — per-capability × per-locale results: correctness (gating) and RTL/expansion (observation) called out separately so bilingual coverage is obvious.
- **AI evaluation** (if AI-powered) — the `evaluate-genai-feature` / `evaluate-agent` result folded in, with its own evidence and a link to that evaluation's artifacts.
- **E2E & integration** — the feature's end-to-end paths walked and the downstream side effects verified (record/email/event/index), with any cross-feature handoff to `journey-test` noted.
- **Regression-vs-prior-runs** — capabilities/cases whose verdict regressed vs. the last comparable run, or "first comparable run — no regression baseline yet."
- **Map & defects** — `map_status` and heals applied this run; defects filed via `bug-report` with Jira links; recommended next actions and any `journey-sec`/`journey-perf`/`journey-test` handoffs.
- **Evidence affordances** — make the actual evidence obvious and reviewable, not just rationale: clickable case/evidence links, annotated-screenshot thumbnails that open full size, collapsible sections, color coding, RTL support, and print-friendly styling.

The HTML is saved in the same `sara-reports/` convention Sara uses elsewhere; for `EVALUATE_GENAI_FEATURE`-style AI work, cross-link the AI evaluation's feature-scoped bundle via `related_items` so the next run retrieves the combined context quickly.

## Hard Rules

- **One feature per run.** If the user names several, test the first and queue the rest (or ask which one).
- **Every case traces to a capability.** No orphan cases, no padding. Every capability ends the run with one of the four verdicts — never silently untested; mark `UNVERIFIED` with a reason.
- **Negatives are mandatory.** Any capability that implies validation, permission, or a guard gets its negative cases run. A feature is **never** `FEATURE VERIFIED` on positive cases alone when guards exist.
- **Discovery maps, execution fires.** Discovery is non-destructive (Option A); negative, abuse, and destructive cases run in full during execution, with dedicated test data + cleanup and sensitive handling. The user may prioritize them, never skip the matrix's ownership of them.
- **The matrix grows mid-run only with a written, logged row.** Every added case is appended to the **`## Mid-run additions`** section of `matrix.md` as a numbered row (lens tag + trigger + outcome), `log`ged, traced to a capability, and kept under the agreed budget; a discovery that would exceed the scope-gate size is surfaced, not absorbed silently. A sibling case that was executed but never written back into `matrix.md` is a process violation — at run end, the original matrix plus `## Mid-run additions` must account for every case run.
- **Locator-first is non-negotiable.** Playwright for web (live headed, not silently headless), Appium for mobile. Screenshots only to diagnose a locator failure or to evidence a confirmed defect — never to read state or pick the next action.
- **Annotate every defect screenshot** (red rectangle) before attaching; attach the annotated version only; delete the original only after the annotated file exists; if annotation fails, stop and report the blocker.
- **`screen loaded` is not a pass.** A positive/edge case passes only on the correct observed outcome with correct persisted state; a negative case passes only when correctly rejected/guarded. Confirm high-consequence FAILs (missing guard, data corruption, security hole) with a second clean observation.
- **Functional + Security gate the verdict; UI/UX + Performance inform.** Do not flip the verdict on a UI/UX or performance finding unless the user promoted that lens, the defect makes a capability impossible to complete (then it's Functional), or the feature states an explicit requirement (then it's a Functional case).
- **Confirmed defects go through `bug-report`** — never ad-hoc formatting. Link to Jira when tracked; attach annotated evidence.
- **Security stays authorized and scoped.** Probe only the feature under test and the provided accounts in an authorized environment. Prove a guard's presence/absence; never weaponize, exfiltrate real PII, or run destructive/DoS exploits. Systematic probing beyond this feature hands off to `journey-sec`.
- **Sensitive findings get `sensitive: true` handling.** Keep exploit detail out of the chat summary and HTML body (pointer only), store redacted in the run's `sensitive/`, hand off to `bug-report`/`journey-sec`, and stop probing once proven.
- **Destructive cases use test data + a cleanup/reset path** and clean up after themselves; never run them against shared or production-like data without a reset path. Record any data left behind in follow-ups.
- **Truncation is always disclosed.** If coverage is capped/trimmed (scope gate, quick run, budget), `log` exactly what was not run and report the verdict with its coverage scope. A trimmed run never reads as full coverage.
- **AI output is never judged by a single oracle.** If the feature is AI-powered, hand the output evaluation to `evaluate-genai-feature` / `evaluate-agent`; the four lenses test the deterministic shell and Security runs prompt-injection. Report both together.
- **Localization is mandatory for user-facing capabilities** (per the project locale rules): correctness gates (Functional), RTL/expansion observes (UI/UX). Run each user-facing capability in every supported locale and surface it as its own section.
- **Stay in scope.** This skill tests one feature deeply, including its own end-to-end paths and integration points. A multi-feature whole-journey E2E hands off to `journey-test`; deep performance to `journey-perf`; deep UX/usability/accessibility beyond the lens to `journey-ux`; systematic security to `journey-sec`; a whole-site pass to `site-audit`. Name the handoff in the report.
- **The locator map is a written file, and `map_status` must be backed by it.** A run that drove live locators is not done until `maps/<platform>.yaml` is written with the targets it acted on. Never record a `map_status` in `index.json` (or the report) for a map file that does not exist on disk — a `verified` status with an empty/absent `maps/` folder is a process violation. At run end, reconcile: the map file exists ⇄ the index `map_status` matches it.
- **Honor the lock.** Refuse to start when another active feature-deep-test holds a fresh lock; release on completion and on every abort path.
- **Missing project context lowers confidence, not validity.** If the Quality Canvas / Value-Quality Map / Product Kit are absent, proceed — they inform edge/negative/localization selection — and note that selection was unguided rather than marking the run invalid.
