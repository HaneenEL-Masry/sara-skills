# feature-deep-test — Sara skill

Deeply test a **single feature** against the live app across four lenses —
**Functional · UI/UX · Performance · Security** — with exhaustive positive,
edge, and negative/abuse coverage, returning one feature verdict plus
evidence and an optional HTML report.

> Part of [Sara](https://github.com/) — WE WILL's business-care quality agent.
> This is the empty cell the other skills don't fill: not a quick page sweep,
> not a happy-path flow, not a story's acceptance criteria, not a whole-site
> audit — **one feature, all four lenses, full depth.**

## What it does

- **Discovers** the feature's full live surface (two passes: context read + a
  non-destructive live capability map) before any case runs.
- **Builds a governed scenario matrix** — positive / edge / negative cases per
  capability, each tagged with the lens(es) it serves. The matrix can grow
  mid-run when execution reveals new branches (every addition is logged).
- **Runs all four lenses itself** to real depth — not spot checks, not handed
  off to other skills:
  - **Functional** — correct outcomes, data integrity, state/transitions,
    idempotency, concurrency, interrupted flow, feature-level E2E + downstream
    effects.
  - **Security** — auth/role boundaries, IDOR, forced navigation, injection as
    input-validation, captured API request/response, session/token, data
    exposure (with `sensitive: true` handling).
  - **UI/UX** — visual craft, design-system consistency, component states,
    responsive + mobile + RTL, Nielsen heuristics, WCAG accessibility.
  - **Performance** — real timing (cold + warm, median of repeats), latency,
    perceived responsiveness, payload weight, slow-network, degradation points.
- **Cross-cutting**: Localization (correctness gates, RTL/layout observes) and
  AI-powered detection (hands AI-output quality to `evaluate-genai-feature`).

## Verdict model

**Functional + Security gate the verdict** → `FEATURE VERIFIED` /
`PARTIALLY VERIFIED` / `NOT VERIFIED`. **UI/UX + Performance are ranked,
business-impact-tagged follow-ups** — they don't gate unless the user promotes
a lens, a defect makes a capability impossible to complete (then it's
Functional), or the feature states an explicit UX/perf requirement.

## Use it

- `/feature-deep-test <feature>` — e.g. `deep test the export feature`
- Or ask Sara naturally: *"test everything about checkout — functional, UX,
  performance, security, all the edge and negative cases."*

## Not for (route elsewhere)

| Want | Use instead |
|------|-------------|
| Quick defect hunt on a page | `SWEEP` |
| One happy-path flow's business value | `journey-test` |
| One story's acceptance criteria | `story-test` |
| One deep lens on a journey | `journey-perf` / `journey-sec` / `journey-ux` |
| The entire site | `site-audit` |

## Artifacts

Self-contained under the project knowledge base:

```
.sara/features/<slug>/
  feature.md      # capability inventory (source of truth)
  matrix.md       # scenario matrix (+ logged mid-run additions)
  maps/<platform>.yaml   # locator map (fingerprinted)
  runs/feature-deep-test/<platform>/<timestamp>/   # evidence, screens, bugs, sensitive
```

Plus a `features` entry in `.sara/index.json` and an opt-in HTML report in
`sara-reports/`.

## Install

Copy the `feature-deep-test/` folder into your Sara skills directory:

- Claude: `~/.claude/skills/feature-deep-test/`
- Codex: `~/.codex/skills/feature-deep-test/`

To make it a first-class Sara mode (menu + routing), add `FEATURE_DEEP_TEST` to
your `sara` agent + `/sara` skill router (capability menu, routing rules, skill
dependencies).

## Requires

- **Playwright MCP** (web) or **Appium MCP** (mobile) for live, locator-first
  driving — headed browser for web.
- Reads project context from `.sara/` (Quality Canvas, regression suites,
  feature evaluations, known issues) when present; degrades gracefully when not.
- `bug-report` skill for filing confirmed defects.

## License

Add your license of choice (e.g. MIT) before publishing.
