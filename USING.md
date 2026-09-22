# Using workflow-audit

The five layers in detail, how to run less than all of them, what the report contains, and how to
use the method without Claude Code. The [README](README.md) is enough to get started — this is
for when you want the detail.

**Contents**

- [The five layers](#the-five-layers)
- [Other commands](#other-commands)
- [Running less than the whole thing](#running-less-than-the-whole-thing)
- [The report](#the-report)
- [Turning findings into a plan](#turning-findings-into-a-plan)
- [Using it without Claude Code](#using-it-without-claude-code)
- [Pairing with ui-path-radar](#pairing-with-ui-path-radar)

---

## The five layers

Run them one at a time, or chain the lot.

### 1. Discovery — `/workflow-audit layer1`

Lists every way into a feature: sheet triggers, navigation links, toolbar buttons, dashboard
cards, context menus, deep links.

The output is a list you check against your own idea of the app. **If something you built isn't
on it, the layer has just told you nobody can reach it.**

This is the cheapest layer, about ten minutes, and the one to start with.

### 2. Flow tracing — `/workflow-audit layer2`

Takes each way in and follows it forward: this button opens that screen, that screen offers these
three actions, this action leads somewhere else. Every step gets a file and line number.

### 3. Issue detection — `/workflow-audit layer3`

Checks 33 kinds of problem. Among them:

- **Dead ends** — you arrive somewhere with nothing to do and no way back
- **Dismiss traps** — a screen opens with no way to close it
- **Buried buttons** — the thing you came to do is below the fold
- **One-way sheets** — an open path exists, a close path doesn't
- **Stale context** — a screen opens holding data from last time
- **Gesture-only actions** — no keyboard equivalent, which matters on a Mac
- **Loading with no timeout** — a spinner that can spin forever
- **Fake data in a shipping build**
- **Platform gaps** — works on iPhone, quietly missing on Mac
- **Invisible controls** — a real button whose fill matches the surface behind it, so it
  reads as plain text. Wired correctly, placed correctly, and still unfindable

Fourteen of the 33 are automatic searches with tests to stop them silently breaking. The rest
work by listing everything that should be right, then checking each one.

### 4. Semantic evaluation — `/workflow-audit layer4`

Looks at each traced path the way a person would:

- **Can a new user find this** without being told it exists?
- **How many taps** does finishing actually take?
- **Did anything confirm it worked?**
- **Can they back out** of a wrong turn?

### 5. Data wiring — `/workflow-audit layer5`

Checks that features show real data. It flags anything that looks finished on screen but is
reading from a stub, a hardcoded value, or a mock.

This catches the feature that shipped half-done when nobody replaced the placeholder.

---

## Other commands

| Command | What it does |
|---|---|
| `/workflow-audit fix` | Turns findings into a fix plan, ordered into phases |
| `/workflow-audit status` | Where you got to, if an audit was interrupted |
| `/workflow-audit trace "A → B → C"` | Follows one specific path you're wondering about |
| `/workflow-audit diff` | Compares against your last report |

---

## Running less than the whole thing

A full audit on a 200 to 600 file app is a real investment — typically one to three hours of a
Claude Code session. You rarely need all of it.

**Match the layer to what you changed:**

| You just | Run |
|---|---|
| Added a feature with several screens | `layer2` + `layer4` |
| Reworked navigation | `layer1` + `layer3` |
| Changed a model several features read | `layer5` |
| Added Mac-specific or iPad-specific code | `layer3` |
| Are about to ship | the full audit |

**Chasing one path.** When a tester reports something and you want to pin down where it lives:

```
/workflow-audit trace "Dashboard → Stuff Scout → Save"
```

Faster than a full discovery pass.

**Only what's new.** After you've fixed things from a previous report:

```
/workflow-audit diff
```

Loads the last report, re-runs the layers, and shows you only what changed — new problems,
things that came back, rows still open. Useful as a check before release.

**Which to use.** Every run is fresh by default: a scan from scratch, a standalone report. Use
`diff` when you've been actively fixing between runs and don't want to re-read findings you
already dealt with. Use a fresh run when something fundamental changed — new architecture, a new
platform — or the last report is old enough to distrust.

---

## The report

Written to `.agents/research/YYYY-MM-DD-workflow-audit-<slug>.md`.

- **A file and line for every finding.** Nothing is reported unless the audit actually read the
  place it's pointing at, so the report can't claim a problem nobody looked at.
- **An 8-column table:** number, finding, urgency, risk if you fix it, risk if you don't, ROI,
  blast radius, effort. A status column appears when a report is shown again.
- **A suggested fix** where the fix is mechanical.

**It never changes your code.** You decide what to fix, defer, or ignore.

**Reading it.** Eight columns needs a wide window, roughly 150 characters. Narrower and the cells
stack vertically, which is hard to scan. GitHub, GitLab, VS Code's preview, Obsidian, Typora,
Bear, MacDown, iA Writer, and Marked 2 all render it properly. If a table looks broken in a
terminal, the file is fine — the window is too narrow.

---

## Turning findings into a plan

The plugin installs **two** skills. The second one, `plan`, turns a finished audit into an
ordered piece of work.

```
/plan --workflow-audit
```

It reads `.workflow-audit/handoff.yaml` — written when an audit completes — and produces a
phased plan: what to do first, what each phase costs, what could go wrong, how you'd test it,
and how you'd back it out. The plan is shown to you and saved to `.agents/research/`.

Because the audit already rated every finding, `plan` uses those ratings rather than
re-deciding them.

**It says what and where, never how.** A task reads *"Add keyboard dismissal to
AddItemView.swift:45-80. Done when: tapping outside a text field dismisses the keyboard"* —
not a snippet to paste. Code appears only to point at a pattern already in your repo, or to
name an API you'd otherwise have to look up.

**Three ways it can start:**

| Start from | When |
|---|---|
| A workflow audit | `.workflow-audit/handoff.yaml` exists and is under two weeks old |
| A codebase audit | A report sits in `.agents/research/`, under two weeks old |
| Nothing | You describe the work yourself |

It picks on its own, preferring a workflow audit when both are present, and falls back to
asking you when the newest report is over two weeks old rather than planning from stale
findings.

**It checks your git state first.** Uncommitted changes get you a prompt — commit now, or
continue and accept that you can't cleanly revert.

**On a project with no CLAUDE.md**, it takes a quick look at how your code is already written
so the plan matches your conventions rather than inventing its own.

---

## Using it without Claude Code

The method is written down in a file, so you can hand it to Cursor, Windsurf, Copilot Chat, or
anything else that reads files. You lose the automation, not the approach:

```
You are a code auditor for SwiftUI projects (iOS, iPadOS, or macOS). I'm
giving you a skill document that describes a multi-layer UI workflow audit.

1. Read the methodology sections — they define HOW to scan
2. Follow the layer order: Discovery, Flow Tracing, Issue Detection,
   Semantic Evaluation, Data Wiring
3. For each layer, enumerate candidates FIRST, then verify each one.
   Do NOT just search for known anti-patterns.

Key principle: orphaned views and unwired data have no code signature
to search for. You find them by listing everything that SHOULD be
connected, then checking which ones aren't.

Here is the skill document:
[paste contents of skills/workflow-audit/SKILL.md]

Start with Layer 1: list all view files and their navigation connections.
```

**What Claude Code adds:** it does the searching itself rather than asking you to, remembers
where it got to across a long audit, tracks findings between runs, and hands off to the related
skills. The prompt above gets you the thinking; Claude Code does the work.

---

## Pairing with ui-path-radar

[ui-path-radar](https://github.com/Terryc21/radar-suite) covers nearby ground more cheaply:

|  | ui-path-radar | workflow-audit |
|---|---|---|
| **Asks** | Is there a path to this screen at all? | Can someone actually finish what they came for? |
| **Finds** | Unreachable screens, broken back buttons | Dead ends, traps, friction |
| **Costs** | Less | More |

Running both is the normal pattern: ui-path-radar finds the structural holes first, workflow-audit
catches what gets through. They use the same citation format, so the two reports read together.
