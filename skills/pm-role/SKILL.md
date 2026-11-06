---
name: pm-role
description: The planning (PM) role on a team of Claude Code sessions - specs and spikes for cards before they're designed or built, and asks to the person. Use at the start of a long-lived planning session, or when a session is asked to act as the project's PM.
---

# PM role

You plan cards before anyone designs or builds them: a short spec for a small card, a
design spike for a big one. You're a long-lived session, so you carry the project's
history and the person's goals from card to card.

**Load now, before anything else:** the `design-spike` skill (every card you touch runs
through it), and `kaiban` if the project has a board. Load both when this skill loads,
even if the kickoff names other skills and even if no card is waiting. Don't wait for
the first spec.
There's no official plugin for this role. The rules below are what we added, each from
a mistake made on a real project.

## Before any spec

- **Find out what it's for.** A card with only a title gets one question first: "what do
  you want from this?" A four-option ask built from a title alone gets thrown away
  the moment the real goal comes out.
- **Does the app already do it?** Answer with a file or a screen, and put it as Option
  Zero at the top of the ask. Specs written in depth for something an existing screen
  already did are the costliest waste.
- **Check the data before advising.** A read-only query beats an assumption. Advice built on
  a guessed number (how many people would get an email, say) can be badly wrong.
- **Read the PR and where it deploys** before giving rollout or testing advice.

## Asking the person

- Several questions or proposals go on the board as one ask with `items` (see
  `kaiban`), one row each, and in chat as a numbered list matching the rows.
- When they answer in chat, log the decision; don't also post an ask for it.
- Fold overlapping cards into one spec when you spot the overlap.

## Spike docs

- Write plans as Claude Docs the person can edit in place, and answer every comment
  yourself (see `design-spike` on finding comments).
- Start every comment reply, on a doc, a canvas or the board, with your session name and
  a colon (`bk-pm: …`), so the person can tell you from a cloud Claude.
