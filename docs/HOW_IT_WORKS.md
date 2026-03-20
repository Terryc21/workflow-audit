# How Bug Prospector and Workflow Audit Differ from Pattern-Based Plugins

Most code analysis plugins — and most of what Claude does during code review — catches **syntactic** problems. The compiler catches your missing `@MainActor`. A linter flags the force unwrap. Code review spots the retain cycle. These are real issues, but they share a trait: **the code visibly breaks a known rule.**

Bug Prospector and Workflow Audit find the other kind of bug — the kind where the code compiles, passes review, and works fine in every test you've written. Then a real user does something you didn't think of, and it breaks.

| Pattern-Based Tools Find | Bug Prospector / Workflow Audit Find |
|--------------------------|--------------------------------------|
| Missing `@MainActor` | State machine that deadlocks when two sheets open simultaneously |
| Force unwraps | Array that's never empty today but will be after a sync conflict |
| Retain cycles | Data created during onboarding that's never cleaned up on logout |
| `try?` without logging | Error path that resets the data but leaves the UI showing "Saving..." forever |
| Deprecated API usage | Code that works on Apple Silicon but silently fails on Intel Macs |
| Missing accessibility labels | Button without debouncing that creates duplicate records on double-tap |
| Compiler warnings | User who renames an item, deletes it, and watches it reappear from CloudKit |
| Unused imports | Spotlight search that loads 10,000 items into memory to find one |

**The gap these plugins fill:** Your code looks correct. It follows every convention. The compiler is happy. But a user with a slow network, a large inventory, or an unexpected workflow hits a path nobody tested — and something breaks silently.

## Bug Prospector: What Does the Code Assume?

Bug Prospector works **from the code outward** — reading functions, tracing data, and asking "what does this assume, and what happens when that assumption is wrong?" It applies 7 analytical lenses:

1. **Assumption Audit** — List every implicit assumption, evaluate what breaks when violated
2. **State Machine Analysis** — Find states that deadlock, overlap, or never reset
3. **Boundary Conditions** — Zero items, empty strings, max values, off-by-one
4. **Data Lifecycle Tracing** — Follow data from creation to deletion, find orphans and gaps
5. **Error Path Exerciser** — Take every error branch and see if the UI recovers
6. **Time-Dependent Bugs** — Timezone shifts, rapid taps, slow networks, first launch after months
7. **Platform Divergence** — Apple Silicon vs Intel, iOS vs macOS, iPad multitasking

The grep patterns are search accelerators, not the analysis. The analysis is the framework: read the code, name the assumption, evaluate the violation, check for guards, classify.

## Workflow Audit: What Does the User Experience?

Workflow Audit works **from the user inward** — starting from what's visible on screen, following what the user taps, and finding where the experience breaks. Most code analysis works inside-out (start from code, find code problems). Workflow Audit works outside-in.

The 5-layer approach mirrors how a real user encounters your app:

**Layer 1 — Discovery:** What can the user actually find? Scans for every entry point — sheet triggers, navigation links, toolbar buttons, context menus, promotion cards. If a feature exists but has no visible entry point, users will never reach it.

**Layer 2 — Flow Tracing:** What happens when they tap? Follows each entry point through its complete flow — every screen, every state change, every branch. Maps the path a user actually walks, not the path the developer intended.

**Layer 3 — Issue Detection:** Where does the experience break? 20 categories of UX defects that only surface when you trace the full journey: dead ends (tapped a button, got an empty sheet), broken promises (card says "Track Price" but opens a generic list), dismiss traps (sheet with no close button on macOS), buried actions (primary CTA below the fold), stale context (navigated from search results but the view lost the search term).

**Layer 4 — Semantic Evaluation:** How does it feel? Evaluates from the user's perspective — not "is the code correct" but "is this discoverable, efficient, forgiving?" A feature can be technically complete but practically invisible.

**Layer 5 — Data Wiring:** Is it real? Verifies that the UI is connected to actual data, not mock values, hardcoded strings, or placeholder content that shipped by accident.

The key insight: **a bug that only appears when you follow the user's path won't be found by tools that only look at the code.** A view might be perfectly implemented but unreachable. A button might work correctly but lead somewhere the user didn't expect. A flow might function on iOS but trap macOS users with no dismiss button.

## How They Work Together

Bug Prospector finds code that **will break** under conditions nobody tested. Workflow Audit finds experiences that **are already broken** but nobody noticed because no one walked the full path.

Run Workflow Audit to find what's broken now. Run Bug Prospector to find what will break next.

## What Makes the Methodology Different

**They teach Claude HOW to think, not just what to grep for.** The lenses and layers are analytical frameworks that guide reasoning about intent and assumptions, not pattern-matching against a rule list.

**They verify before reporting.** Both require reading the actual code around a finding and checking for existing guards before classifying anything. A grep match is a lead, not a verdict.

**They classify instead of pass/fail.** BUG / FRAGILE / OK / REVIEW reflects that many findings need human judgment. "Works today but breaks under foreseeable conditions" is a different category than "crashes now."

**They're interactive.** Scope selection, lens selection, phase gates with opt-out, fix-or-defer decisions — the user controls what gets analyzed and what gets fixed.

**They close the loop.** Analysis → rated report → fix workflow → build verification in one session. Most audit plugins stop at the report.

## See Also

- [Bug Prospector](https://github.com/Terryc21/bug-prospector) — Find bugs that pattern-based scanners miss
- [Workflow Audit](https://github.com/Terryc21/workflow-audit) — Audit SwiftUI user workflows for dead ends and UX friction
- [Xcode Workflow Skills](https://github.com/Terryc21/xcode-workflow-skills) — Full 22-skill suite (includes both)
