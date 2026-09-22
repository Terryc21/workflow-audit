# Changelog

All notable changes to workflow-audit are documented here.

---

## 2026-09-21 — v3.1.0

New detection category: **Invisible Control** (#33). A control that is correctly wired
and correctly placed, but renders with no visible boundary — so the user cannot tell it
is a control.

- **Why the existing 32 could not catch it.** Every one asks a structural question: does
  the wiring exist, does it lead somewhere, is the control reachable. An invisible control
  passes all of them — it is a real `Button`, it fires a real action, it is not below the
  fold, it is not gesture-only. The defect is entirely in the rendered pixels while the
  code reads as correct. Phantom Touch Target (#29) is the inverse case: looks tappable,
  is not.

- **Detection is a luminance delta, not a pattern match.** Identify each styled control
  surface, resolve the surface *behind* it, and compare relative luminance. A fill within
  ~0.05 of its background with no border and no shadow is a finding; so is a foreground
  below WCAG 4.5:1 against its own fill.

- 🛑 **A finding requires a RESOLVED ground — absence of evidence is not a bare
  background.** This was calibrated against a real population before release, and the
  obvious implementation turned out to be the wrong one. A 45-line textual look-back over
  120 `.buttonStyle(.bordered)` sites flagged 84, and **45 of those 84 had a container
  modifier further up the same file**, outside the window: 54% of the flags were artifacts
  of the window size rather than of any measurement. The category now requires that the
  ground be nameable — a container found by walking the whole enclosing view body, an
  explicit `.background(...)` at the call site, or a parent view actually read — and that
  name doubles as the Work Receipt the skill already mandates. Unresolved sites are
  reported as a count, never as rows in the findings table.

- **The threshold is not the delicate part.** On the calibrated population the real case
  sat at delta 0.000 and the nearest benign case at 0.139, so anything from 0.02 to 0.07
  gives identical verdicts. The category says so, and tells a reader who sees wrong
  findings to suspect the ground-resolution step instead.

- 🛑 **A system button style is not a guarantee.** `.buttonStyle(.bordered)` is idiomatic
  and documented, and on macOS its fill can resolve to the same luminance as the window
  ground in light appearance — zero delta, faint border, no visible body. The category
  states explicitly that "it uses a real button style" is not evidence of visibility.

- ⚠️ **Both appearances must be checked.** A pair that contrasts in dark can be identical
  in light. A single-appearance check finds half the cases and reports confidence it has
  not earned.

- ⚠️ **A render harness is not the real surface.** Drawing the control on a test sheet can
  pass a defect the production ground fails, because the harness background is not the
  background.

- **Accessibility interaction noted.** Where a project forbids low-opacity category tints
  (common when red–green discrimination cannot be relied on), a "muted tint" fix for an
  invisible control reintroduces the accessibility problem. The category directs fixes
  toward shape and boundary rather than hue.

Origin: a real audit passed a set of controls as structurally sound while four of them
were, on the running app, indistinguishable from their background. The audit was correct
on every question it asked. This category is the question it was not asking.

Updated: `SKILL.md` (category table, handoff enum, persona-handoff key count),
`agents/layer3-issue-detection.md` (Category 33 with patterns and procedure),
`agents-skill/issue-categories.md` (numbered summary).

---

## 2026-08-06 — v3.0.3

One-line fix to the Layer 1 boolean-state pattern introduced in v3.0.1, caught by a second
round of eval runs.

- **`show*` boolean state was invisible to the scan.** The v3.0.1 pattern required a capital
  letter immediately after the prefix — `(showing|isShowing|isPresenting)[A-Z]...` — so
  `showRKRTrialExpired = true` and `showAddTask = true` matched nothing while
  `showingSheet = true` matched. Bare `show` is now an alternative and the capital is no
  longer required. Measured across a 614-file project: 640 matches before, 715 after — 75
  entry points that the v3.0.1 pattern silently dropped, including a paywall sheet in the
  directory used for testing.

---

## 2026-08-06 — v3.0.2

Core sync. `radar-suite-core.md` had been forked in May and was 427 lines behind the
canonical copy in radar-suite, which meant this skill was missing shared behavior it
already claimed to inherit.

- **Synced `radar-suite-core.md` from radar-suite** (660 → 1,122 lines, 32 sections gained):
  Artifact Lifecycle, Checks-performed reporting, Finding IDs with short titles, Detail
  Block Rendering, Pipeline UX, Severity Scale by Axis, and others. workflow-audit's two
  local adaptations were preserved: the `.radar-suite/` → `.workflow-audit/` path-substitution
  header and the Opt-Out section.
- **Scoped two imported mandates that would have broken a standalone run.** The synced file
  requires an `axis` label plus six coaching fields on every finding, enforced by a schema
  gate whose six rules all key off those fields — and points at
  `skills/radar-suite-axis-classification/SKILL.md`, which ships with radar-suite, not here.
  Applied as written it would have rejected every workflow-audit finding. Both sections are
  now marked radar-suite-only, with workflow-audit's own handoff schema and the Work Receipts
  citation rule named as the equivalent.
- **Corrected a README claim.** The output-format section said "the schema gate rejects
  unattributed claims." That gate is radar-suite's and doesn't apply here. The line now
  describes what workflow-audit actually enforces: a finding isn't emitted unless it cites a
  location the audit read.
- **Header notes that sibling radar skills** named throughout the core file apply only when
  radar-suite is installed alongside; workflow-audit runs standalone without them.

---

## 2026-08-06 — v3.0.1

Portability and correctness fixes, found by running the skill against a real SwiftUI
codebase and comparing six runs (three with the skill, three without) on identical prompts.

- **Layer 1 discovery patterns generalized.** Three of the four prescribed scans were one
  project's private naming rather than SwiftUI APIs — `activeSheet`, `selectedSection`, and
  `PromotionCard` (a component type no other codebase has). Layer 1 now leads with the
  presentation APIs every SwiftUI project shares (`.sheet(isPresented:` / `.sheet(item:`,
  `fullScreenCover`, `popover`, `NavigationLink` / `.navigationDestination`, toolbar /
  `contextMenu` / `swipeActions` / `Menu`, `alert` / `confirmationDialog`), then *discovers*
  the project's routing convention — central enum or per-view booleans — rather than assuming
  a spelling. Measured on one feature directory: old patterns 0 matches, new patterns 36.
- **Added an under-count guard to Layer 1.** If the convention scans return zero, cross-check
  against the raw `.sheet(` presenter count before reporting a small inventory, and state
  which patterns returned zero rather than omitting them. A Layer 1 that under-counts doesn't
  fail loudly — it produces a short, clean-looking inventory, and every later layer inherits
  the blind spot.
- **Fixed a self-contradiction in the rating requirement.** The opening instruction demanded
  ratings on "every finding" unconditionally while naming only four of the table's six
  dimensions. Layer 1 produces an inventory of unverified flags, not findings, so the rule
  collided with the Work Receipts principle. It now names all six dimensions, explains why a
  partial rating drops a row out of triage, and scopes itself to layers that produce verified
  findings.
- **Session Setup now has a non-interactive path.** Setup uses AskUserQuestion, which needs a
  user; the skill also runs as a subagent, from a script, or inside a pipeline. It now
  proceeds with documented safe defaults (`experienced` / `full` / `review`), says it did so,
  reads `session-prefs.yaml` if a prior interactive run left one, and never writes that file
  from a non-interactive run.
- **`radar-suite-core.md` added to the Reference Documentation list**, marked read-first. The
  skill defers roughly twenty behaviors to it, but it was previously reachable only via the
  `inherits:` frontmatter key.

---

## 2026-04-09 — v3.0.0

### Cross-Skill Handoff Protocol

- Writes `.workflow-audit/persona-handoff.yaml` after Layer 4 with personas, D/E/F/R evaluation matrix, and checks_performed
- Reads ui-path-radar handoff (if exists) to import companion findings and note category overlap
- Companion findings tagged `[via ui-path-radar]` in Issue Rating Table
- Handoff is "if exists" -- zero behavior change when ui-path-radar is not installed
- Updated Layer 4 execution to include persona handoff generation

---

## 2026-04-09 — v2.6.0

### Adopted radar-suite-core.md for Infrastructure Parity

- Added `radar-suite-core.md` as local copy (no radar-suite dependency required)
- workflow-audit now inherits: session persistence, checkpoint/resume, wave-based fixes, work receipts, fix-forward bias, known-intentional suppression, pattern reintroduction detection, contradiction detection, context exhaustion guard
- All `.radar-suite/` paths adapted to `.workflow-audit/` in workflow-audit context
- Opt-Out section preserved from rating-system.md
- Deleted `skills/shared/rating-system.md` (superseded by core)
- Updated `skills/plan/SKILL.md` reference to point to `radar-suite-core.md`
- Added Shared Patterns section to SKILL.md referencing all core features

---

## 2026-04-09 — v2.5.0

### 12 New Issue Categories + Table Formatting

**New categories (aligned with ui-path-radar coverage):**
- 🔴 CRITICAL: Destructive Without Confirmation, Silent State Reset
- 🟡 HIGH: Empty State Missing, Error Recovery Missing, Keyboard Obscures Input, Permission Denied Dead End, Modal Stacking, Navigation Container Mismatch
- 🟢 MEDIUM: Phantom Touch Target, Race Condition UX, Invisible Selection
- ⚪ LOW: Double-Nested Navigation

Total categories: 20 → 32. Full detection patterns in `agents/layer3-issue-detection.md` (Categories 21-32).

**Table formatting improvements:**
- Column-aligned Issue Categories table with compact headers
- Shortened 8-column Issue Rating Table headers (Risk:Fix, Risk:NoFix, Blast, Effort)
- Terminal width cue: tells user to widen window if table renders as vertical blocks
- Multi-row finding text supported for long descriptions

---

## 2026-04-09 — v2.4.0

### Experience-Level Session Setup

- Added Session Setup section with experience-level question (Beginner/Intermediate/Experienced/Senior)
- Experience-adapted skill introductions for all 4 levels
- Auto-apply rules: Beginner enables `--explain` and `--sort impact`; Senior defaults to `--sort effort`
- Output rules table covering 7 dimensions (intro, explain, progress banner, finding text, sort, citations, summaries)

---

## 2026-04-09 — v2.3.0

### User Impact Explanations & Sort Modes

**`--explain` / `--no-explain`**
- New toggle: appends a 3-line explanation after each finding in the Issue Rating Table
- Format: What's wrong (one sentence), Fix (one sentence), User experience before/after (one sentence)
- Code-only findings use "Developer experience" instead of "User experience"
- Default: off. Toggle mid-session with `--explain`
- Defined in `skills/shared/rating-system.md` "User Impact Explanations" section

**`--sort` modes**
- `--sort urgency` (default) -- most broken first (Urgency descending, ROI descending)
- `--sort effort` -- easiest safe wins first (Fix Effort ascending, Risk:Fix ascending)
- `--sort impact` -- most user-visible first (Risk:No Fix descending, Urgency descending)
- `--sort implement` -- dependency-aware ordering for sprint planning
- Can be changed mid-session without re-running the audit
- Defined in `skills/shared/rating-system.md` "Sort Modes" section

**End-of-audit suggestion updated**
- Now shows available sort modes and explain toggle

---

## 2026-04-07 — v2.2.0

### Handoff YAML & Rating System

- Handoff brief generation (`.workflow-audit/handoff.yaml`) for consumption by planning skill
- Shared rating system extracted to `skills/shared/rating-system.md`
- Terminal width detection with compact 4-column fallback
- Group hints for batch fix operations

---

## Pre-changelog history

- v2.0 — 5-layer audit architecture (discovery, flow tracing, issue detection, semantic evaluation, data wiring)
- v1.0 — Initial release with entry point discovery and dead-end detection
