---
name: reviewer-role
description: How a PR gets reviewed on a team of Claude Code sessions - a fresh reviewer subagent per PR, given the project's code style and behaviour doc, checking what the PR makes untrue elsewhere. Use when a PR is ready for review, when spawning a reviewer, or when a session is asked to act as the project's reviewer.
---

# Reviewer role

Review is the one role that gains from a fresh agent every time: a read that doesn't
share the author's reasoning, so it doesn't inherit its blind spots. So each PR gets a
**fresh reviewer subagent**, not a long-lived reviewer session. What a long-lived
reviewer carried in memory goes in the project's code style instead. (JoyRiders retro,
10-31, D161.)

**Load first:** the official `pr-review-toolkit` plugin
(`/plugin install pr-review-toolkit@claude-plugins-official`) for its reviewer agents.
The rules below are what we added on top, from the JoyRiders retro (D150–D162).

## Starting a review

Whoever moves a card to review (the dev session, or the orchestrator) spawns a fresh
reviewer, e.g. `pr-review-toolkit:code-reviewer`, and gives it:

- the PR's link and the card (what done looks like),
- the project's code style and its plain-words behaviour doc (e.g. `docs/how-it-works.md`),
- this checklist,
- **not** the author's reasoning: the diff and the card only.

## Checklist

- **What does the PR make untrue elsewhere?** Search the behaviour doc and the decision
  log for what the PR touches. When a field or a source of truth moves, grep every
  reader of the old one, scripts, seed data, tests and docs included. Most misses had
  this shape: the diff was right, but a doc sentence, an old field's readers or a logged
  decision were left wrong. (JoyRiders: JR-113, #54, D136.)
- **Does a helper already do this?** Flag any logic that restates an existing helper or
  rule, such as a status check written inline where a named predicate exists. (JoyRiders:
  JR-65, JR-70, JR-78; the near-copies of the time, domain and form helpers.)
- **No screenshots, no approval** for a change to a screen: 390 and 1280 px, light and
  dark, on the card (see `dev-role`).
- Say plainly what you checked and what you didn't. "Not checked in a browser" is a
  reason to send it back, not a note.

## Every 10 bug cards

The orchestrator spawns one reviewer to read the last 10 bug cards together and look for
a pattern. A pattern becomes a rule in the code style, named after the bugs it came
from. (JoyRiders: JR-116 found ten such rules at the end; they should have come along
the way.)
