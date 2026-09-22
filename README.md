# workflow-audit

![Version](https://img.shields.io/github/v/tag/Terryc21/workflow-audit?label=version) ![Last commit](https://img.shields.io/github/last-commit/Terryc21/workflow-audit) ![Stars](https://img.shields.io/github/stars/Terryc21/workflow-audit?style=flat) ![Issues](https://img.shields.io/github/issues/Terryc21/workflow-audit) ![License](https://img.shields.io/github/license/Terryc21/workflow-audit) ![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet)

**Can someone actually use the thing you built?**

Most code checkers read your app one file at a time and ask whether each line is written
correctly. workflow-audit asks a different question: *can a person get from here to there and
finish what they came to do?*

It walks your SwiftUI app the way a user would, and tells you where they'd get stuck.

*4 min read · [the five layers and every command](USING.md) · [what changed](CHANGELOG.md)*

---

## The bug it exists to find

**You build a screen. Nothing opens it.**

Every line of that screen is correct. It compiles. It would pass review. The only thing wrong is
that the path to it was never connected — so the feature you built isn't in the shipped app, and
nothing anywhere told you.

**Or: your app works out a number, and a screen displays a number, and the two were never wired
together.** Both files are fine. The user sees a blank space, or a stale value presented as
confidently as a real one.

No rule-checker can catch either. Nothing is written incorrectly. **The problem lives in the
space between the files** — in the walk, not the lines.

That's what this does. It starts at every door into a feature — a button, a menu item, a card —
and follows each one forward. At every step it asks what a person would ask: *Can I get here at
all? Is the button I need visible without scrolling? Did anything tell me it worked? I've changed
my mind, can I get out?*

---

## Try it

Type these into Claude Code, not Terminal. **One at a time** — pasting both gives a confusing SSH
error, because the second line gets read as part of the first command.

```
/plugin marketplace add Terryc21/workflow-audit
```

```
/plugin install workflow-audit@workflow-audit
```

Then open a SwiftUI project and run the cheapest layer:

```
/workflow-audit layer1
```

About ten minutes. It lists every way into every feature in your app, and you check that list
against your own idea of what's there. **Anything you built that isn't on the list is something
nobody can reach.**

That's a real result before you commit to a full audit.

> **This is SwiftUI-only.** On a UIKit or non-Apple project it won't find anything useful.

**[See a real report →](skills/workflow-audit/examples/2026-04-15-workflow-audit-stuffolio.md)**
A full five-layer audit on a shipping app, with findings and fix plans.

---

## What it looks at

Five passes, each runnable on its own:

| | Layer | It finds |
|---|---|---|
| 1 | **Discovery** | Every way into a feature — and anything with no way in |
| 2 | **Flow tracing** | Where each path actually leads, step by step |
| 3 | **Issue detection** | 33 kinds of problem: dead ends, screens that won't close, buttons below the fold, spinners with no timeout, fake data left in |
| 4 | **How it feels** | Can a new user find this? How many taps? Did anything confirm it worked? Can they back out? |
| 5 | **Real data** | Features that look finished but read from a placeholder nobody replaced |

**[All five in detail, plus the other commands →](USING.md)**

---

## Then: what to do about it

The plugin installs a second skill. When an audit finishes, it hands its findings over:

```
/plan --workflow-audit
```

You get an ordered plan — what to fix first, what each phase costs, what could break, how
you'd test it, and how you'd undo it. It says *what* and *where*, never *how*: a task points
at `AddItemView.swift:45-80` and says when it's done, rather than handing you code to paste.

It works without an audit too — point it at a codebase report, or just describe the work.

**[More on planning →](USING.md#turning-findings-into-a-plan)**

---

## Alongside your linter, not instead of it

A linter checks whether each file is written correctly — cheap, fast, on every save. Keep it.

This checks whether the paths through your app work. It costs more and you run it before a
release, not on every save.

| A linter is better at | workflow-audit is better at |
|---|---|
| Every save | Before a release |
| Style and pattern problems | Connection and wiring problems |
| One file at a time | What happens between files |
| Hundreds of settled rules | A newer, narrower question |

The two bugs at the top of this page are the reason both are worth having. Neither breaks a rule,
so nothing flags them. Neither crashes. **The app just quietly does less than you built.**

---

## What it can't do

It reads your code; it doesn't run it. So it won't find:

- **Wrong-but-reachable.** It can tell you a button exists, that people can reach it, that its
  handler runs and reads real data. It cannot tell you the handler does the right thing.
- **Correct code, wrong pixels.** A control can be wired right, placed right, and still be
  invisible or unreadable on screen. v3.1.0 catches the common case by comparing a control's
  fill against the surface behind it (Invisible Control), but that is a measurement of declared
  colors, not of what the compositor actually drew. If a control looks wrong to you and the
  audit says it is fine, trust your eyes.
- **Anything that only happens at runtime** — timing problems, memory pressure, animation glitches.
- **Problems nobody has described yet.** A clean audit means none of the 33 known kinds. New
  shapes get added in later releases.
- **Whether your design is any good.** It flags a button that looks buried; whether that's
  actually a problem is your call.

**Treat findings as leads, not a to-do list.**

For the rest: linters catch single-file problems, profilers catch threading and memory, and tests
catch wrong logic. This covers the gap between them.

---

## Pairs with ui-path-radar

[ui-path-radar](https://github.com/Terryc21/radar-suite) asks "is there a path to this screen at
all" — cheaper, structural, catches unreachable screens and broken back buttons. workflow-audit
asks "can someone finish what they came for" — costlier, deeper, catches dead ends and friction.

Running both is the normal pattern. [Details →](USING.md#pairing-with-ui-path-radar)

---

## Where it stands

**v3.0.3.** Used through real App Store releases on [Stuffolio](https://stuffolio.app)
([App Store](https://apps.apple.com/app/stuffolio/id6757168677)), a 600-file SwiftUI app.

**You'll need** a SwiftUI project and Claude Code — or any AI tool that reads files, since
[the method works as a plain prompt](USING.md#using-it-without-claude-code) in Cursor, Windsurf,
or Copilot Chat.

**Keeping it current:** `/plugin update workflow-audit`. Check the [CHANGELOG](CHANGELOG.md)
before an audit you're relying on.

**[How the method works, and why →](docs/HOW_IT_WORKS.md)** ·
**[Contributing →](CONTRIBUTING.md)**

---

## Related skills

[**bug-echo**](https://github.com/Terryc21/bug-echo) — find the same bug elsewhere after a fix ·
[**bug-prospector**](https://github.com/Terryc21/bug-prospector) — hunt for bugs before a release ·
[**radar-suite**](https://github.com/Terryc21/radar-suite) — six skills tracing user paths, including ui-path-radar ·
[**unforget**](https://github.com/Terryc21/unforget) — one file for everything you deferred ·
[**prompter**](https://github.com/Terryc21/prompter) — rewrite prompts before running them ·
[**skill-reviewer**](https://github.com/Terryc21/skill-reviewer) — candid reviews of other skills ·
[**tutorial-creator**](https://github.com/Terryc21/tutorial-creator) — lessons from your own code

---

**New to Claude Code?** A *skill* is a set of written instructions Claude Code knows how to
follow. Type `/workflow-audit`, and it walks your app and writes you a report with file and line
numbers for everything it found. Nothing to memorise.

**Terry Nyberg**, [Coffee & Code LLC](https://stuffolio.app/). Built while shipping
[Stuffolio](https://stuffolio.app) through real App Store submissions. If workflow-audit catches
a real bug for you, [a coffee](https://buymeacoffee.com/stuffolio) is appreciated — though a note
about what worked or didn't is worth more.

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-FFDD00?style=flat&logo=buy-me-a-coffee&logoColor=black)](https://www.buymeacoffee.com/stuffolio)

Apache 2.0 — see [LICENSE](LICENSE) and [NOTICE](NOTICE).
