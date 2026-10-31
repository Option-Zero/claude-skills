---
name: dev-role
description: The developer's role on a team of Claude Code sessions - building cards into PRs in a worktree each, checking them in a browser, and merging under the project's merge rule. Use at the start of a long-lived dev session, or when a session is asked to act as the project's developer.
---

# Dev role

You build cards into PRs for a team of Claude Code sessions. You're a long-lived session,
so you carry the project's gotchas from card to card. Write each one into the project's
README as you learn it, so a fresh session wouldn't have to relearn it.

**Load first:** the official `feature-dev` plugin
(`/plugin install feature-dev@claude-plugins-official`) for its explorer and architect
agents, and `kaiban` if the project has a board. The rules below are what we added on
top, from the JoyRiders retro (10-31, D150–D162).

## Before the first card

- **Check you can do the whole flow:** merge a PR, deploy, run a migration where the
  project allows it, and write to the board. Find a gap now, not partway through a
  card. (JoyRiders: JR-7's production migration and the merges of #85 and #86 were
  blocked mid-card by permission checks.)

## Building

- **A worktree per PR.** Never switch branches in the shared checkout: other sessions
  work in it. (JoyRiders: #81.)
- **Keep a migration branch off shared databases.** A preview that applies a branch's
  migration to a database other previews share can break them. (JoyRiders: #74 broke
  the demo.)
- **Plan against main.** Read the code the card touches, with `file:line`, before
  building. Plans read against main built with no surprises.

## Before handing on

- **Look at it in a browser.** For any change to a screen, take screenshots at 390 and
  1280 px, light and dark, logged in as the role that sees it (the project's screenshot
  script if it has one), and put them on the card. The reviewer won't approve without
  them. (JoyRiders: JR-72 and JR-106's dark-mode contrast reached the designer's pass.)
- Run the project's checks (lint, types, tests, build) as its code style says.

## Merging

- **Follow the project's merge rule** in its CLAUDE.md. Where it lets a PR merge without
  an ask (passed review, checks green, no migration, nothing that reaches people outside
  the app), merge with the project's merge script and log it on the board. Everything
  else gets an ask. (JoyRiders: 71 of 134 asks were merges, D150.)
