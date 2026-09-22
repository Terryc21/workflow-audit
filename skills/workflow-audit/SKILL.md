---
name: workflow-audit
description: 'Systematic UI workflow auditing for SwiftUI applications. Discovers entry points, traces user flows, detects dead ends and broken promises, audits data wiring, evaluates from user perspective. Triggers: "workflow audit", "audit flows", "find dead ends", "check navigation".'
version: 3.0.3
author: Terry Nyberg
license: Apache-2.0
allowed-tools: [Read, Grep, Glob, Bash, Edit, Write, AskUserQuestion]
# `inherits:` is resolved relative to this skill directory. If radar-suite-core.md
# is relocated to a sibling skill or shared package, update this path.
inherits: radar-suite-core.md
metadata:
  tier: execution
  category: analysis
---

# Workflow Audit Skill

> **Quick Ref:** 5-layer UI workflow audit: discover entry points → trace flows → detect issues → evaluate UX → verify data wiring. Output: `.workflow-audit/` in project root.

<workflow-audit>

You are performing a systematic workflow audit on this SwiftUI application.

**Rating findings.** Where a layer produces *findings* — a confirmed problem you have read the code for — rate each one on all six dimensions in the Issue Rating Table: Urgency, Risk:Fix, Risk:NoFix, ROI, Blast Radius, and Fix Effort. A partial rating quietly drops a finding out of triage: the user is deciding what to fix before the next release, and a row missing Risk:NoFix or Blast Radius can't be weighed against the rows that have them, however real the bug is.

Layer 1 is the exception, by design. Its output is an *inventory* of entry points with flags for Layer 2 to investigate — hypotheses, not verified problems. Rating an unverified flag would assign urgency to something nobody has confirmed is broken, which is exactly what the Work Receipts rule exists to prevent. Layer 1 reports counts and flags; ratings begin at Layer 3 where findings are verified against source.

## Quick Commands

| Command | Description |
|---------|-------------|
| `/workflow-audit` | Full 5-layer audit |
| `/workflow-audit layer1` | Discovery only — find all entry points |
| `/workflow-audit layer2` | Trace — trace critical paths |
| `/workflow-audit layer3` | Issues — detect problems across codebase |
| `/workflow-audit layer4` | Evaluate — assess user impact |
| `/workflow-audit layer5` | Data wiring — verify real data usage |
| `/workflow-audit trace "A → B → C"` | Trace a specific user flow path |
| `/workflow-audit diff` | Compare current findings against previous audit |
| `/workflow-audit fix` | Generate fixes for found issues |
| `/workflow-audit fix --apply-anyway` | Same, but auto-apply even with uncommitted changes |
| `/workflow-audit status` | Show audit progress and remaining issues |

## Overview

The Workflow Audit uses a 5-layer approach:

| Layer | Purpose | Output |
|-------|---------|--------|
| **Layer 1** | Pattern Discovery - Find all UI entry points | Entry point inventory |
| **Layer 2** | Flow Tracing - Trace critical paths in depth | Detailed flow traces |
| **Layer 3** | Issue Detection - Categorize issues across codebase | Issue catalog |
| **Layer 4** | Semantic Evaluation - Evaluate from user perspective | UX impact analysis |
| **Layer 5** | Data Wiring - Verify features use real data | Data integrity report |

## Reference Documentation

Read these files for methodology and patterns (paths relative to this skill's directory):

- `radar-suite-core.md` - **Read this first.** The `inherits:` key above points here, and this skill defers roughly twenty behaviors to it — Session Setup, Issue Rating Table columns and indicator scale, sort order, Work Receipts, Finding Classification, Checkpoint & Resume, the handoff schema. Without it you are missing the definitions the rest of this file assumes.
- `agents/README.md` - Overview and quick start
- `agents/layer1-patterns.md` - Discovery regex patterns
- `agents/layer2-methodology.md` - Flow tracing process
- `agents/layer3-issue-detection.md` - Issue categories
- `agents/layer4-semantic-evaluation.md` - User impact analysis
- `agents/layer5-data-wiring.md` - Data integrity methodology

For templates and examples:
- `agents-skill/templates/` - YAML templates for each layer
- `agents-skill/examples/` - Good and bad patterns

> **Note:** These paths are relative to the skill directory (`~/.claude/skills/workflow-audit/`). When reading these files, resolve from the skill's installed location, not the current working directory.

## Issue Categories

| Category                    | Severity     | Description                                  |
|-----------------------------|--------------|----------------------------------------------|
| Dead End                    | 🔴 CRITICAL | Entry point leads nowhere                    |
| Wrong Destination           | 🔴 CRITICAL | Entry point leads to wrong place             |
| Mock Data                   | 🔴 CRITICAL | Shows fabricated data when real data exists   |
| Destructive Without Confirm | 🔴 CRITICAL | Delete/clear with no confirmation dialog      |
| Silent State Reset          | 🔴 CRITICAL | In-progress work lost on navigate away        |
| Incomplete Navigation       | 🟡 HIGH     | Must scroll/search after landing              |
| Missing Auto-Activation     | 🟡 HIGH     | Expected mode/state not set                   |
| Unwired Data                | 🟡 HIGH     | Model data exists but feature ignores it      |
| Platform Parity Gap         | 🟡 HIGH     | Works on one platform, broken on another      |
| Promise-Scope Mismatch      | 🟡 HIGH     | Specific CTA opens generic destination        |
| Buried Primary Action       | 🟡 HIGH     | Primary button hidden below scroll fold       |
| Dismiss Trap                | 🟡 HIGH     | Only Cancel/back visible, no forward path     |
| Context Dropping            | 🟡 HIGH     | Item context lost between platforms/notifs    |
| Notification Nav Fragility  | 🟡 HIGH     | Untyped dict used for navigation context      |
| Sheet Presentation Asymm    | 🟡 HIGH     | Different sheet mechanisms per platform       |
| Empty State Missing         | 🟡 HIGH     | Blank screen when list empty                  |
| Error Recovery Missing      | 🟡 HIGH     | Error shown but no retry or recovery path     |
| Keyboard Obscures Input     | 🟡 HIGH     | TextField covered by keyboard, no scroll      |
| Permission Denied Dead End  | 🟡 HIGH     | Denied with no path to Settings               |
| Modal Stacking              | 🟡 HIGH     | Multiple sheets/alerts stacked                |
| Nav Container Mismatch      | 🟡 HIGH     | Selection tag invalid for current container   |
| Invisible Control           | 🟡 HIGH     | Wired and placed, but no visible boundary     |
| Two-Step Flow               | 🟢 MEDIUM   | Intermediate selection required               |
| Missing Feedback            | 🟢 MEDIUM   | No confirmation of success                    |
| Gesture-Only Action         | 🟢 MEDIUM   | Only accessible via swipe/long-press          |
| Loading State Trap          | 🟢 MEDIUM   | Spinner with no cancel/timeout/escape         |
| Stale Navigation Context    | 🟢 MEDIUM   | Cached context never cleared/validated        |
| Phantom Touch Target        | 🟢 MEDIUM   | Looks tappable but has no action              |
| Race Condition UX           | 🟢 MEDIUM   | Conflicting ops triggered simultaneously      |
| Invisible Selection         | 🟢 MEDIUM   | Selected/active but no visual indicator       |
| Inconsistent Pattern        | ⚪ LOW      | Same feature accessed differently             |
| Orphaned Code               | ⚪ LOW      | Feature exists but no entry point             |
| Double-Nested Navigation    | ⚪ LOW      | NavigationStack inside NavigationStack        |

## Design Principles

### 1. Honor the Promise
> When a button/card says "Do X", tapping it should DO X.
> Not "go somewhere you might find X."

### 2. Context-Aware Shortcuts
> If user's context implies a specific item, skip pickers.

### 3. State Preservation
> When navigating to a feature, set up the expected state.

### 4. Consistent Access Patterns
> Same feature should be accessed the same way everywhere.

### 5. Data Integrity
> If the app tracks data relevant to a feature, the feature must use it.
> Never show mock/hardcoded data when real user data exists.
> Never ignore model relationships that would improve decisions.

### 6. Primary Action Visibility
> The primary action must be visible without scrolling after the user completes the key interaction.
> Pin Save/Continue/Done buttons outside ScrollView or in toolbar. Never bury them below tall content.

### 7. Escape Hatch
> Every view must have a visible way to go forward OR back. Cancel alone is not enough after user completes a step.

### 8. Gesture Discoverability
> Every action available via gesture (swipe, long-press) should also be accessible via a visible button or menu.

### Freshness

Base all findings on current source code only. Do NOT read prior
workflow-audit reports from `.workflow-audit/` or `.agents/` (they are
stale). DO read companion handoff YAML from sibling skills (radar-suite,
ui-path-radar) — they describe the current codebase from a different
angle, not stale findings. Ignore cached findings from auto-memory or
previous sessions.

## Session Setup

Session Setup is defined in `radar-suite-core.md` (4 questions: experience, table format, fix handling, optional explanation). Use that — do not duplicate it here.

**When there is no user to ask.** Session Setup uses AskUserQuestion, which needs an interactive session. workflow-audit also runs where that channel doesn't exist — dispatched as a subagent, driven from a script, or invoked inside a larger pipeline. Don't stall waiting for an answer that can't arrive, and don't skip the audit: proceed with `experienced` / `full` tables / `review` (never auto-apply fixes), and say in one line that you used defaults because the run was non-interactive. Those defaults are the safe direction — full tables lose no information, and `review` means nothing gets modified without someone looking.

If `.workflow-audit/session-prefs.yaml` already exists, read it and use those values instead of the defaults; a prior interactive run recorded what this user actually wants. Do not write the file from a non-interactive run — preferences the user never expressed shouldn't become sticky.

When question 4 ("Explain what this skill does?") is answered "Yes, briefly," substitute these workflow-audit-specific explanations for the generic ones in the inherited core:

- **Beginner**: "Workflow Audit checks every button, link, and menu item in your app to make sure they work correctly. Think of it like testing every door in a building -- does it open? Does it lead where the sign says? Does anything break along the way? It runs in 5 layers, each going deeper into your app's user experience."
- **Intermediate**: "Workflow Audit systematically audits all UI entry points, traces user flows, detects dead ends and broken promises, evaluates UX from the user perspective, and verifies data wiring. Five layers: discovery → tracing → issues → evaluation → data wiring."
- **Experienced**: "5-layer UI audit: entry points, flow traces, issue detection, UX evaluation, data wiring. Rating tables + fix plans."
- **Senior/Expert**: "Entry point → flow → issues → UX → wiring. Rating tables."

The Experience-Level Output Rules table also lives in `radar-suite-core.md`. Apply it as-is.

---

## Shared Patterns

See `radar-suite-core.md` for: Session Persistence, Checkpoint & Resume, Accepted Risks, Wave-Based Fix Presentation, Fix-Forward Bias, Test Hygiene, Plain Language Communication, Work Receipts, Contradiction Detection, Finding Classification, Audit Methodology, Context Exhaustion, Progress Banner, Issue Rating Tables, Known-Intentional Suppression, Pattern Reintroduction Detection, Experience-Level Output Rules, Implementation Sort Algorithm, Handoff YAML schema, Opt-Out.

> **Path note:** workflow-audit uses `.workflow-audit/` instead of `.radar-suite/` for all persistent files (session-prefs.yaml, checkpoint.yaml, known-intentional.yaml, ledger.yaml).

---

## Execution Instructions

When invoked, perform the workflow audit:

### If no arguments or "full":
Run all 5 layers sequentially, outputting findings to `.workflow-audit/` in the project root

### If "layer1" or "discovery":

> **Source root note:** scan commands below assume Swift sources live under `Sources/` (the SwiftPM convention and the layout used by the canonical example project). For default-template Xcode projects, substitute the directory containing your `*.swift` files (often `<ProjectName>/`). Same substitution applies to every `Sources/` reference in the layer reference docs under `agents/`.

**Start with the SwiftUI presentation APIs**, since those are the same in every project:

1. Sheets and covers: `grep -rE "\.sheet\(isPresented:|\.sheet\(item:|\.fullScreenCover\(|\.popover\(" Sources/`
2. Push navigation: `grep -rE "NavigationLink|\.navigationDestination\(" Sources/`
3. Toolbar and menu affordances: `grep -rE "ToolbarItem|\.contextMenu|\.swipeActions|Menu \{" Sources/`
4. Alerts and dialogs: `grep -rE "\.alert\(|\.confirmationDialog\(" Sources/`

**Then look for the project's own routing convention**, which differs from app to app. Many codebases funnel presentation through a central enum (`activeSheet = .someCase`, `selectedSection = .someTab`, a `Route`/`Destination` type) or through named card components. Discover the local convention rather than assuming a particular spelling:

5. `grep -rE "activeSheet = \.|selectedSheet = \.|selectedSection = \.|navigationPath\.append" Sources/` — central-dispatch enums, if the project has one
6. `grep -rE "(show|showing|isShowing|isPresenting)[A-Za-z]* = true" Sources/` — per-view boolean state, the most common alternative. Keep the bare `show` alternative, and don't require a capital letter after the prefix: `showRKRTrialExpired` and `showAddTask` are as common as `showingSheet`, and a pattern anchored on `showing` alone silently drops them.

Then catalog every entry point found into `layer1-inventory.yaml`, and flag suspicious ones for Layer 2.

**If steps 5-6 return nothing, that is information, not an answer.** A project with zero central-dispatch matches is using some other convention — not lacking entry points. Before reporting a low count, confirm against step 1's raw `.sheet(` presenters: if the app has 40 sheet modifiers and your inventory has 3 entries, the scan missed the convention, and every downstream layer inherits that blind spot. Report which patterns returned zero rather than omitting them silently, so the reader can tell "we looked and found none" from "we never looked."

This matters more than it sounds. A Layer 1 that under-counts doesn't fail loudly — it produces a short, clean-looking inventory, and the audit ends up certifying a codebase it never actually scanned.

### If "layer2" or "trace" (no path argument):
1. Read flagged entry points from Layer 1
2. For each flagged entry point, trace the complete user journey
3. Document in `layer2-traces/flow-XXX.yaml`
4. Identify gaps between expected and actual journeys

### If "trace" with path argument (e.g., `trace "Dashboard → Add Item → Photo → Save"`):
Targeted flow trace — trace a specific user journey described in natural language:
1. Parse the path description into discrete steps (split on `→`, `->`, or `,`)
2. For each step, identify the SwiftUI view, button, or action that triggers it:
   - Search for view names, sheet triggers, navigation actions matching each step
   - Use `grep -r` for button labels, sheet cases, navigation destinations
3. Trace the complete code path step by step:
   - File and line number for each transition
   - State changes (sheet presentations, navigation, @State mutations)
   - View transitions (what view appears at each step)
4. At each step, check for issues:
   - Is the expected next action visible without scrolling? (Buried Primary Action)
   - Does the user have a forward path? (Dismiss Trap)
   - Does the CTA match the destination scope? (Promise-Scope Mismatch)
   - Is feedback shown on completion? (Missing Feedback)
5. Document the trace and any issues found
6. Output: Issue Rating Table for any findings, plus the step-by-step trace

### If "layer3" or "issues":
1. Scan ALL entry points for common issues
2. Check for orphaned sheet cases (enum vs handler mismatch)
3. Check for orphaned views (defined but never instantiated)
4. Categorize by severity
5. Output to `layer3-results.yaml`

### If "layer4" or "evaluate":
1. For each issue, assess user impact
2. Rate: discoverability, efficiency, feedback, recovery
3. Map violations to design principles
4. Output to `layer4-semantic-evaluation.md`
5. Write `.workflow-audit/persona-handoff.yaml` (see Persona Handoff section)

### If "layer5" or "data-wiring" or "wiring":
1. Inventory model properties and relationships (what data the app tracks)
2. For each feature view, check what model data it actually reads
3. Detect mock/hardcoded data patterns (asyncAfter delays, static arrays, placeholder strings)
4. Cross-reference: model capabilities vs feature consumption
5. Flag unwired integrations (e.g., Price Watch data exists but decision engine ignores it)
6. Check platform parity (extension files, #if os() blocks, dismiss buttons)
7. Output to `layer5-data-wiring.yaml`

### If "diff":
Compare current codebase against the previous audit to show what changed:
1. Read existing `.workflow-audit/layer3-results.yaml` and `.workflow-audit/handoff.yaml`
2. For each previously-reported issue, check if the referenced file + line still has the problem:
   - Read the file at the reported line number
   - Check if the problematic pattern still exists
   - If fixed, mark as "RESOLVED"
   - If file was modified but pattern persists, mark as "STILL OPEN"
   - If file was deleted or moved, mark as "FILE CHANGED — verify manually"
3. Run a quick scan for NEW issues not in the previous report (new files, new ScrollView+button combos, new sheets without handlers)
4. Output a diff summary:
   ```
   Audit Diff: <previous date> → <current date>
   ✅ Resolved: <count> issues fixed since last audit
   🔴 Still Open: <count> issues remain
   🆕 New: <count> new issues detected
   📁 Changed: <count> files modified since audit (may need re-verification)
   ```
5. Show the full Issue Rating Table with a Status column prepended (✅/🔴/🆕)

### If "fix" or "fixes":
1. Read `layer3-results.yaml` and `layer5-data-wiring.yaml` for unfixed issues
2. Generate specific code fixes following the patterns in examples/
3. Prioritize by severity (critical first)

🛑 **Before applying anything automatically, run the dirty-tree guard** in
`radar-suite-core.md` § Dirty-tree guard. A dirty working tree downgrades this run to per-item
approval — the audit continues normally, only unattended editing is withheld. Both gates must
pass to auto-apply: blast radius ≤ 2 files AND a clean tree.

`/workflow-audit fix --apply-anyway` waives the tree check for users who accept the risk. It
does not waive the blast-radius gate.

### If "status":
1. Read existing audit files
2. Report: issues found, fixed, remaining
3. Show priority queue for unfixed issues

## Output Format

> **CRITICAL FORMATTING RULE:** The Issue Rating Table below IS the output. Do NOT create separate sections for "Critical Issues", "Data Wiring Issues", "Recommendations", or any other vertical breakdown of findings. Every finding — navigation issues, data wiring issues, orphaned code, missing feedback, design violations — goes into ONE table as ONE row. Context goes in the Finding column. No exceptions.

Always write the full 8-column table to `.workflow-audit/report.md`. In the chat response, prefer the full 8-column table; if the user reports it rendering as vertical blocks instead of horizontal rows, tell them: "The rating table needs a wider terminal to display correctly. Try widening your window or opening the report file in a markdown viewer." Do not use `tput cols` to auto-detect — it returns 80 in piped/non-TTY contexts and gives the wrong answer when the user has a wide terminal.

After completing the audit, provide:

1. **One-line summary** — entry point count, issue count by severity (one sentence, not a section)
2. **Issue Rating Table** — every finding in a single table (see below)
3. **One-line next step** — suggest `/plan --workflow-audit` if fixes are needed (the `plan` skill ships with this plugin and is registered — `claude plugin details workflow-audit` lists both; the handoff YAML it consumes is at `.workflow-audit/handoff.yaml`)

That's it. Three items. No other sections.

### Issue Rating Table

> **Reference:** See `radar-suite-core.md` for full column definitions, indicator scale, and sorting rules.

**Hard formatting rule — Table, not list:** ALL findings MUST be in a single markdown table. Each finding is ONE ROW. Ratings are COLUMNS read left-to-right. Never expand findings into individual sections, vertical blocks, or bullet-pointed ratings. Do NOT create separate headed sections for categories of findings (e.g., "Data Wiring Issues", "Critical Issues", "Orphaned Views"). ALL categories go in the same table. The Finding column carries the context.

All findings MUST be presented in this format, sorted by Urgency then ROI:

```markdown
| #   | Finding                        | Urgency      | Risk:Fix | Risk:NoFix | ROI      | Blast    | Effort  |
|-----|--------------------------------|--------------|----------|------------|----------|----------|---------|
| 1   | Dead end: "View Warranty"      | 🔴 Critical | ⚪ Low  | 🔴 Crit   | 🟠 Exc  | 🟢 2f   | Trivial |
|     | → empty sheet                  |              |          |            |          |          |         |
| 2   | Promise-scope: "Track Price"   | 🟡 High     | 🟢 Med  | 🟡 High   | 🟠 Exc  | 🟡 4f   | Small   |
|     | opens generic list             |              |          |            |          |          |         |
```

Use the Issue Rating scale:
- **Urgency:** 🔴 CRITICAL (dead end, wrong destination, mock data) · 🟡 HIGH (broken promise, missing activation, unwired data) · 🟢 MEDIUM (two-step flow, missing feedback) · ⚪ LOW (inconsistency, orphaned code)
- **Risk: Fix:** Risk of the fix introducing regressions
- **Risk: No Fix:** User-facing consequence of leaving the issue
- **ROI:** 🟠 Excellent · 🟢 Good · 🟡 Marginal · 🔴 Poor
- **Blast Radius:** Number of files the fix touches (e.g., "⚪ 1 file", "🟢 3 files", "🟡 12 files"). Count by grepping for callers/references before rating.
- **Fix Effort:** Trivial / Small / Medium / Large

### User Impact Explanations

If the user passes `--explain` (or the project's CLAUDE.md includes `explain-findings: true`), append a brief explanation for each finding after the Issue Rating Table. See `radar-suite-core.md` "User Impact Explanations" for the exact format and rules.

**Precedence (highest to lowest):** explicit `--no-explain` flag · explicit `--explain` flag · CLAUDE.md `explain-findings` · Beginner-experience auto-enablement.

### End-of-Audit Suggestion

After presenting audit results, always print:

```
💡 To generate a phased fix plan from these findings, run: /plan --workflow-audit (ships with this plugin; or consume .workflow-audit/handoff.yaml with your own tooling)
💡 Re-sort: --sort effort (easiest first) · --sort impact (most visible first) · --sort implement (build order)
💡 Explain findings: --explain (adds what's wrong / fix / user experience for each finding)
```

---

## Handoff Brief Generation

After completing all layers (full audit) or `fix` mode, generate `.workflow-audit/handoff.yaml` for consumption by the planning skill.

### When to Generate

- After a full 5-layer audit completes
- After `fix` mode completes (refreshes the brief with current state)
- NOT after individual layer runs (layer1, layer2, etc.)

### Format

```yaml
# Handoff Brief — generated by workflow-audit
# Consumed by /plan --workflow-audit

project: <project name from directory>
audit_date: <ISO 8601 date>
source_files_scanned: <count>

summary:
  total_issues: <count>
  critical: <count>
  high: <count>
  medium: <count>
  low: <count>

file_timestamps:
  <file path>: "<ISO 8601 mod date>"
  # one entry per unique file referenced in issues[]

issues:
  - id: <sequential number>
    finding: "<description>"
    category: <dead_end|wrong_destination|mock_data|destructive_no_confirm|silent_state_reset|incomplete_navigation|missing_activation|unwired_data|platform_gap|promise_scope_mismatch|buried_primary_action|dismiss_trap|context_dropping|notif_nav_fragility|sheet_asymmetry|empty_state_missing|error_recovery_missing|keyboard_obscures|permission_dead_end|modal_stacking|nav_container_mismatch|two_step_flow|missing_feedback|gesture_only_action|loading_state_trap|stale_nav_context|phantom_touch_target|invisible_control|race_condition_ux|invisible_selection|inconsistent_pattern|orphaned_code|double_nested_nav>
    urgency: <critical|high|medium|low>
    risk_fix: <critical|high|medium|low>
    risk_no_fix: <critical|high|medium|low>
    roi: <excellent|good|marginal|poor>
    blast_radius: "<description, e.g. '1 file' or '4 files'>"
    fix_effort: <trivial|small|medium|large>
    files:
      - <file path>
    suggested_fix: "<what to do, not how>"
    group_hint: "<optional grouping suggestion, e.g. 'missing_confirmations'>"
```

### File Timestamps

For each unique file path referenced across all issues, record its modification date at audit time. This enables the planning skill to detect staleness — if a file changed after the audit, affected issues may need re-verification.

```bash
# Get file mod date (macOS)
stat -f "%Sm" -t "%Y-%m-%dT%H:%M:%SZ" "<file path>"
```

### Group Hints

Optional field suggesting how the planning skill might batch issues:
- Issues with the same `group_hint` are candidates for a single task
- The planning skill is free to ignore hints and group differently
- Common hints: `missing_confirmations`, `missing_feedback`, `orphaned_features`, `dead_code`, `platform_parity`

---

## Persona Handoff (Cross-Skill)

After completing Layer 4 (full audit or standalone layer4), write `.workflow-audit/persona-handoff.yaml` for consumption by ui-path-radar (if installed):

```yaml
source: workflow-audit
version: <skill version>
timestamp: <ISO 8601>
project: <project name>

personas:
  - name: "Warranty Tracker"
    goal: "Never miss a warranty deadline"
    key_workflows:
      - "Add item with warranty"
      - "See expiring warranties"
    evaluation:
      discovery: 5     # 1-5 star rating
      efficiency: 4
      feedback: 5
      recovery: 5
    issues_found:
      - finding_ref: 3   # issue id from handoff.yaml
        impact: "Breaks trust in promotion cards"

evaluation_matrix:
  - workflow: "Add Item"
    discovery: 5
    efficiency: 4
    feedback: 5
    recovery: 5

checks_performed:
  categories_scanned:   # all 33 category keys
    - dead_end
    - wrong_destination
    - mock_data
    - destructive_no_confirm
    - silent_state_reset
    # ... (all 33)
  persona_evaluation: true
  personas_defined: <count>
```

**When to generate:** After Layer 4 completes (full audit or standalone `layer4` invocation). Not generated for individual layer1/layer2/layer3 runs.

**If ui-path-radar is not installed:** The file is still written. It costs nothing and will be consumed if ui-path-radar is installed later.

---

## Cross-Skill Handoff Consumption (Optional)

Before starting Layer 3, read companion handoffs (if they exist):

```
Read .agents/ui-audit/ui-path-radar-handoff.yaml (if exists)
Read .radar-suite/ui-path-radar-handoff.yaml (if exists)
```

If found:
1. Parse `checks_performed.categories_scanned` to see which categories ui-path-radar already checked
2. Import any CRITICAL/HIGH findings as companion findings -- include in the Issue Rating Table tagged `[via ui-path-radar]`
3. Still run your own scan independently -- cross-reference, don't skip
4. For overlapping findings (same file + same category), note "Also flagged by ui-path-radar" instead of reporting as a new finding

If not found: proceed normally. No change to audit behavior.

</workflow-audit>
