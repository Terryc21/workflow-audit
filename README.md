# Workflow Audit Skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that audits SwiftUI user workflows for dead ends, broken promises, and UX friction.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (Anthropic's CLI for Claude)
- Xcode (for iOS/macOS projects)

## Install

```bash
claude plugin marketplace add Terryc21/workflow-audit
claude plugin install workflow-audit
```

## Included Skills

| Skill | Command | Description |
|-------|---------|-------------|
| **Workflow Audit** | `/workflow-audit` | 5-layer UI audit: discover entry points, trace flows, detect issues, evaluate UX, verify data wiring |
| **Plan** | `/plan --workflow-audit` | Consume audit findings into a phased fix plan with per-task prompting |
| **Rating System** | *(shared reference)* | Standardized rating format used by both skills |

## Quick Start

1. Open Claude Code in any SwiftUI project
2. Run `/workflow-audit` for a full 5-layer audit
3. Review the findings table
4. Run `/plan --workflow-audit` to generate a phased fix plan

## Layer-by-Layer

Run individual layers when you don't need a full audit:

| Command | What it does |
|---------|-------------|
| `/workflow-audit layer1` | Discovery — find all UI entry points |
| `/workflow-audit layer2` | Trace — follow critical user paths |
| `/workflow-audit layer3` | Issues — detect dead ends, buried buttons, dismiss traps, and more |
| `/workflow-audit layer4` | Evaluate — assess user impact |
| `/workflow-audit layer5` | Data wiring — verify real data usage |
| `/workflow-audit trace "A → B → C"` | Trace a specific user flow path |
| `/workflow-audit diff` | Compare current findings against previous audit |
| `/workflow-audit fix` | Generate fixes for found issues |
| `/workflow-audit status` | Show audit progress |

## How It Works

The workflow audit uses a 5-layer approach:

1. **Pattern Discovery** — Scans for sheet triggers, navigation links, promotion cards, and context menus to build an entry point inventory
2. **Flow Tracing** — Traces critical user paths from entry to completion, documenting each step
3. **Issue Detection** — 20 categories including dead ends, buried buttons, dismiss traps, context dropping, notification fragility, sheet asymmetry, stale context, gesture-only actions, loading traps, mock data, and more. 12 automated grep-based checks with regression canaries.
4. **Semantic Evaluation** — Evaluates from the user's perspective: discoverability, efficiency, feedback, recovery
5. **Data Wiring** — Verifies features use real data, checks for mock/hardcoded values, validates platform parity

Findings are rated using a standardized table format with Urgency, Risk, ROI, Blast Radius, and Fix Effort columns.

## Full Plugin

Want all 22 Xcode development skills (testing, debugging, refactoring, release prep, security audit, and more)?

```bash
claude plugin marketplace add Terryc21/xcode-workflow-skills
claude plugin install xcode-workflow-skills
```

## License

MIT

## Author

Created by [Terry Nyberg](https://github.com/Terryc21)
