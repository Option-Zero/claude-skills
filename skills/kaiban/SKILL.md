---
name: kaiban
description: Set up and work from a kaiban, a live board (a published artifact with its own database) that several Claude Code sessions share with the person they work for. It holds what needs the person (asks they answer with a click), what each session is doing, every to-do and handoff between sessions, a map of the system and a log of decisions. Use it to start a board for a project, when a session hands work to another, asks the person something, picks up or finishes work, or when the person mentions their board or kaiban.
---

# Kaiban

One board per project, shared by the person and every Claude Code session working on it.
The person reads it, often on a phone, answers asks with a click, drags to-dos between
groups and hands them to sessions. Sessions write to it as they work. It replaces status
updates in chat, plan pages kept as HTML, and handoffs by message alone.

The page is [`template.html`](template.html) and holds no project content. Everything on
it comes from the artifact's database, which sessions read and write with the
**ArtifactData** tool on the board's URL. Nobody republishes the page to update it.

## The board's data

Five collections. Times are ISO 8601 UTC. People and sessions are named by their session
name (what `ListAgents` shows), and the person by `config.me`.

- **`config/main`** (one doc): `title` (2–4 words), `lede` (one sentence on what the work is
  for and what done means), `me` (the person's name), `areas` (`{key: label}`, the parts of
  the system to-dos belong to), `prefix` and `nextT` (cards are named `<prefix>-<num>`; `nextT` is the next number), `stages` (see below), `map` (see below), `nextD` and `nextA` (the next log
  numbers), optional `archive` (`{label, href}` to an older log).
- **`needs/<id>`**: something only the person can do.
  `title`, `kind` (`decision` · `action` · `blocker`), `body`, optional `steps` (strings),
  `links` (`[{label, href}]`), `options` (decisions: `[{key, label, detail, rec}]`, one
  marked `rec: true`), `items` (a list of proposals or questions answered in one ask, one
  row each: `[{key, text, detail?, kind?, options?}]`; `kind: "question"` takes its own
  `options` as a dropdown, or a typed answer when it has none, and anything else is a
  proposal answered Accept / Accept with changes / Later / Reject; the answer then carries
  `items: {key: {choice, label, note}}`), `doneLabel` (actions: the button's words, e.g. "Both are on
  staging"), `blocks` (blockers: what's blocked), `actFor` (the session that acts on the
  answer), `task` (the id of the card it's about: the ask is parked while that card sits in a
  stage no session owns, such as Up next, Later or Done), `parked` (true when the person
  pressed "Not now"), `order`, `created`, `status` (`open` → `answered` → `closed`), `answer` (written
  by the page: `{choice, label, note, at}`), `updated`.
- **`sessions/<name>`**: `name`, `role` (a few words), `model`, `now` (one sentence on what
  it's doing), `state` (`working` · `waiting` · `idle`), `updated`.
- **`tasks/<id>`**: a card. `num` (its short number: the card is called `<prefix>-<num>`, e.g. JR-12), `title`, `detail` (a two- or three-sentence overview for the person), `notes` (what only sessions need: files, PRs, decisions, steps; shown folded under "For sessions"), `links`, `area` (a key of `config.areas`),
  `stage` (a key of `config.stages`), `owner` (a session name or `config.me`), `state`
  (`handed`: not picked up yet · `doing` · `blocked` · `wontdo`: dropped, kept in Done · `""`), `type` (`spike`: plan the work with the `design-spike` skill, ending in new cards · `bug`: steps to reproduce, may skip a design stage · empty: a story), `waitsOn` (what it's waiting
  for, in words), `from`, `handedAt`, `pickedUpAt`, `order` (lower is higher in its
  column), `created`, `updated`.
- **`log/<id>`**: `num` (`D12` for a decision, `A30` for something that shipped or
  happened), `at`, `headline` (the outcome, shown in bold), `text` (numbers and detail, one
  or two sentences), `links`, `by` (the person's name for their calls, the session's name
  for a default it chose under their go-ahead, which they can overturn), `updated`.

`config.stages` is the board's columns, left to right: `[{key, label, owner?, aside?}]`.
A stage with an `owner` is that session's queue: moving a card into it hands the card to
that session (`owner`, `state: "handed"`, `from`, `handedAt`). A stage with `aside: true`
shows as a box under the board instead of a column, for later work and ideas, and for
Done so finished cards don't take a column. Keys `done` and an aside stage don't count as
open work. A Spec stage, owned by a planning session, is where cards get a design spike
before design or build (see the session block). Clicking a card opens it in a dialog. A
typical set:

```json
[{"key": "later", "label": "Later and ideas", "aside": true},
 {"key": "next", "label": "Up next"},
 {"key": "spec", "label": "Spec", "owner": "<planning session>"},
 {"key": "design", "label": "UI", "owner": "<designer session>"},
 {"key": "build", "label": "Build", "owner": "<dev session>"},
 {"key": "review", "label": "Review", "owner": "<reviewer session>"},
 {"key": "done", "label": "Done", "aside": true}]
```

`config.map` is a list of bands, top to bottom, each `{heading, boxes: [{area, name,
state, pill?, text, links}]}`. `state` is `live`, `changing`, `blocked` or `todo` (or any
word, shown grey); `pill` overrides the pill's words. A box says what that part *is* and
the one live number that matters. It never carries history or a to-do list: the page adds
each box's open to-dos and who is on them from `tasks`.

## Setting up a board

1. `ListAgents` to learn the session names. Ask the person what to call them, for
   `config.me`, even when the account shows a name: it may not be the one they use.
2. Copy `template.html` to your scratchpad, replace `KAIBAN_TITLE` with the project's name
   plus "kaiban" (e.g. "JoyRiders kaiban"), and publish it with
   `capabilities: {db: {}, user: {}, comments: {}}`. The publishing session is the relay:
   it stays watching the board (`ArtifactComments` watch) and passes relay comments on. Record the URL in the project's memory.
3. Write `config/main` (with stages owned by the sessions that do each step), a `sessions` doc per session (leave `now` empty for the sessions
   to fill), and the first `needs` and `tasks`, in one batch.
4. Add [`session-block.md`](session-block.md) to the project's `CLAUDE.local.md` (create it
   if needed, and add it to `.git/info/exclude` unless it's already ignored), with the URL
   filled in. Every session started in the project reads it. That block, not this skill,
   is what keeps sessions using the board.
5. Message each running session (`SendMessage`) with the board's URL and the block's
   contents, since sessions already running don't reread `CLAUDE.local.md`.
6. **Check each session can do its whole job before the first card**: write the board,
   reach Claude Docs if specs live there, and for dev, merge and deploy. Ask each to try
   and report. A gap found now is one fix; found mid-card, it's a stalled card.
7. **Agree a merge rule with the person** and put it in the project's `CLAUDE.md`. The
   default to offer: a PR that has passed review, with green checks, no migration and
   nothing that reaches people outside the app (emails, calendar invites, mailing-list
   contacts), merges without an ask and gets a log row; everything else asks.

## Roles

What worked on JoyRiders (retro, 10-31): long-lived sessions for the roles that carry
context from card to card, and a fresh agent where independence matters more.

- **Orchestrator**: publishes the board, relays its comments, keeps the skills and the
  board in shape, runs retros. Its session name ends in `-orchestrator`.
- **PM, designer, dev**: long-lived sessions, each loading its role skill (`pm-role`,
  `designer-role`, `dev-role`). The role skills point to the official plugins and add
  the rules we learned.
- **Reviewer**: a fresh subagent per PR (`reviewer-role`), not a standing session. The
  board's Review stage can still exist; the session that moves a card there spawns the
  reviewer.

To move a project's board to a newer template, read the artifact, then publish the new
template (title filled in) with its `url`. The data stays where it is.

## Rules that keep it trustworthy

- **The card is the record of a handoff.** A message to the other session is only a nudge.
  A handed card says "not picked up", and the board lists it above the columns, until its
  owner sets `state: "doing"` and `pickedUpAt`.
- **The board can't wake a session, so one session relays.** A write to the board starts
  no turn. Instead, when the person answers an ask or hands a card to a session, the page
  also sends a short comment to Claude ("Kaiban relay to <session>. …"). That wakes the
  session watching the board with comment auto-replies armed (the one that published
  it), the **relay**, usually the orchestrator. The relay passes the text on with
  `SendMessage`, replies only "Passed to <session>" (never what will be done: it isn't
  the one doing it) and resolves the thread. The page waits 10 seconds after a drag
  before sending, so quick re-drags send one handoff, to where the card ends up. A session handing a card to another sends it a
  one-line `SendMessage` itself. Every session also keeps an hourly safety-net check
  (`CronCreate`) in case the relay isn't running; `session-block.md` has the wording.
- **A card's type sets how it's handled.** A spike moves through the person's checkpoints, not
  the build stages: its result is new cards (linked back to the spike) and a log row, and
  the spike itself goes to Done. A bug needs steps to reproduce before anyone picks it up.
- **Moving a card on is the handoff.** Design done means moving the card to the next stage
  (Build), not just marking it done. Write all the handoff fields together.
- **Several questions are one ask.** A list of proposals or questions goes on the board
  as one ask with `items`, not as a numbered list in chat. In chat, ask one question per
  turn.
- **An answer given in chat needs no ask.** Log the decision; don't post an ask just to
  close it. Times on the board come from the clock (`date -u`), never typed by hand.
- **Save a rule the second time it comes up.** When the person corrects the same thing
  twice, the session adds the rule to where it belongs (the design system, the code
  style) right then and tells them in one line.
- **A decision leaves Needs you and enters the log together.** The acting session sets the
  need's `status: "closed"` and adds the log row in one batch. An answered ask waiting only
  on results isn't a need.
- **Log numbers come from `config`.** In the same batch as the log row, update
  `config/main` to bump `nextD` or `nextA`, pinned with `if_version`. If the batch fails
  because another session took that number, reread `config/main` and retry.
- **Card numbers come from `config` too.** A new card takes `nextT` as its `num`, and the same batch bumps `nextT`, pinned with `if_version`. Refer to cards by their name (JR-12) in messages, log rows and asks.
- **Pin every write** with the `if_version` you read. On a conflict, reread and redo the
  change, never overwrite someone else's.
- **Links on everything.** Every PR, preview, page, record and design is a link in `links`,
  never a bare `#123`. When asking for a review, link straight to the view in question.
- **Never send the person to a control you haven't confirmed exists**, and say exactly
  where it is. Better still, have a session do the step.
- **Use the person's domain words**, not the code's. Numbers over adjectives, measured
  rather than guessed, and say which when it's an estimate.
- **Cut means delete.** When the person drops something, delete its task or need. The log
  row recording the cut is the only trace left.
- **Ideas go on the board, not into a session's memory.** A to-do that lives only in one
  session's memory is a dropped handoff waiting to happen: make it a `later` task.
- **The person can drag and edit tasks too.** Treat the board's current state as theirs.
  Don't move a task back to where it was without asking.
