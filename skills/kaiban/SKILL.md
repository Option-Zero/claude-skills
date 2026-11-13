---
name: kaiban
description: Set up and work from a kaiban, a live board (a published artifact with its own database) that several Claude Code sessions share with the person they work for. It holds what needs the person (asks they answer with a click), what each session is doing, every to-do and handoff between sessions, and a log of decisions. Use it to start a board for a project, when a session hands work to another, asks the person something, picks up or finishes work, or when the person mentions their board or kaiban.
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
  the system to-dos belong to), `prefix` and `nextT` (cards are named `<prefix>-<num>`; `nextT` is the next number), `stages` (see below), `nextD` and `nextA` (the next log
  numbers), optional `archive` (`{label, href}` to an older log).
- **`needs/<id>`**: something only the person can do.
  `title`, `kind` (`decision` · `action` · `blocker`), `body`, optional `steps` (strings),
  `links` (`[{label, href}]`), `options` (decisions: `[{key, label, detail, rec}]`, one
  marked `rec: true`), `items` (a list of proposals or questions answered in one ask, one
  row each: `[{key, text, detail?, kind?, options?}]`; `kind: "question"` takes its own
  `options` (`[{key, label, rec?}]`, or plain strings) as a dropdown, or a typed answer when it has none, and anything else is a
  proposal answered Accept / Accept with changes / Later / Reject; the answer then carries
  `items: {key: {choice, label, note}}`. The person can answer some rows and leave the rest: the
  answered rows are relayed at once and kept in `answer.items`, and the ask stays `open`, showing only
  the rows left, until the last row is answered. Act on rows as they arrive, but leave the ask open
  and keep its other rows as they are; close it once it's `answered`). Every question, single or
  a row, also offers **Just my note**: the person answers with the note alone, and the answer's
  `choice` is `"note"`. Nothing is preselected; the recommended option keeps its tag. `doneLabel` (actions: the button's words, e.g. "Both are on
  staging"), `blocks` (blockers: what's blocked), `actFor` (the session that acts on the
  answer), `task` (the id of the card it's about: the ask is parked while that card sits in a
  stage no session owns, such as Up next, Later or Done), `parked` (true when the person
  pressed "Not now"), `order`, `created`, `status` (`open` → `answered` → `closed`), `answer` (written
  by the page: `{choice, label, note, at}`), `updated`. Read every note. A note wins over a pick
  it contradicts; if it's unclear which the person meant, ask.
- **`sessions/<name>`**: `name`, `role` (a few words), `model`, `now` (one sentence on what
  it's doing), `state` (`working` · `waiting` · `idle`), `updated`.
- **`tasks/<id>`**: a card. `num` (its short number: the card is called `<prefix>-<num>`, e.g. BK-12), `title`, `detail` (a two- or three-sentence overview for the person), `notes` (what only sessions need: files, PRs, decisions, steps; shown folded under "For sessions"), `links`, `area` (a key of `config.areas`),
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
 {"key": "design", "label": "UI", "owner": "<ui-designer session>"},
 {"key": "build", "label": "Build and review", "owner": "<dev session>"},
 {"key": "done", "label": "Done", "aside": true}]
```

The page shows the Work board at the top, then Needs you, then Sessions and the log.
Later and Done fold to their count and the newest three cards, with "Show all". A handed
card that nobody has picked up says how long it has waited ("handed 40 min ago, not picked
up").

## Setting up a board

1. **Settle the basics before the board goes up**: the project's folder, its repo, the
   session names and the code style. A board published before these are settled carries
   the wrong names into every card and stage.
2. `ListAgents` to learn the session names. Ask the person what to call them, for
   `config.me`, even when the account shows a name: it may not be the one they use.
   Agree the project's two names: a long one for the board's title (Bookshop) and a
   short one of two to four letters (BK). The short one is `config.prefix`, so cards are
   BK-12, and lowercased it starts every session's name: `bk-orchestrator`, `bk-pm`,
   `bk-ui`, `bk-dev`. Ask the person to `/rename` each session to match before the
   board goes up (a session can't rename itself), since stage owners and card owners
   carry the names.
3. Copy `template.html` to your scratchpad, replace `KAIBAN_TITLE` with the project's name
   plus "kaiban" (e.g. "Bookshop kaiban"), and publish it with
   `capabilities: {db: {}, user: {}, comments: {}}`. The publishing session is the relay:
   it stays watching the board (`ArtifactComments` watch) and passes relay comments on. Record the URL in the project's memory.
4. **Give the relay what it needs to pass things on unattended.** Add allow rules for
   `ArtifactComments`, `ArtifactData` and `SendMessage` to the project's
   `.claude/settings.local.json`, and run the relay session in manual permission mode, not
   auto: auto mode has refused relay wake-ups, leaving relays unanswered for hours.
5. Write `config/main` (with stages owned by the sessions that do each step), a `sessions` doc per session (leave `now` empty for the sessions
   to fill), and the first `needs` and `tasks`, in one batch.
6. Add [`session-block.md`](session-block.md) to the project's `CLAUDE.local.md` (create it
   if needed, and add it to `.git/info/exclude` unless it's already ignored), with the URL
   filled in. Every session started in the project reads it. That block, not this skill,
   is what keeps sessions using the board.
7. Message each running session (`SendMessage`) with the board's URL and the block's
   contents, since sessions already running don't reread `CLAUDE.local.md`.
8. **Check each session can do its whole job before the first card**: write the board,
   reach Claude Docs if specs live there, and for dev, merge and deploy. Ask each to try
   and report. A gap found now is one fix; found mid-card, it's a stalled card.
9. **Test the relay end to end.** Post a test ask for one session, answer it on the board,
   and confirm that session got the relay within a minute. Then close the ask.
10. **Agree a merge rule with the person** and put it in the project's `CLAUDE.md`. The
   default to offer: a PR that has passed review, with green checks, no migration and
   nothing that reaches people outside the app (emails, calendar invites, mailing-list
   contacts), merges without an ask and gets a log row; everything else asks.

## Roles

What has worked: long-lived sessions for the roles that carry
context from card to card, and a fresh agent where independence matters more.

- **Orchestrator**: publishes the board, relays its comments, sweeps for stuck handoffs
  and missed relays (below), keeps the skills and the board in shape, runs retros. It's
  named `<short>-orchestrator`, e.g. `bk-orchestrator`.
- **PM, UI designer, dev**: long-lived sessions, each loading its role skill (`pm-role`,
  `ui-designer-role`, `dev-role`). The role skills point to the official plugins and add
  the rules we learned.
- **Reviewer**: a fresh subagent per PR (`reviewer-role`), not a standing session. The dev
  session spawns it for each PR, so one Build and review column covers both.

To move a project's board to a newer template, read the artifact, then publish the new
template (title filled in) with its `url`. The data stays where it is. Check a change to
the board page in a browser before calling it live.

**Skill changes go through a PR.** When the orchestrator changes a skill, it makes a fresh
branch from main and opens the PR in the same turn. It checks the PR is open before saying
the commits are in it, and asks before committing edits another session made.

## The orchestrator's sweep

An idle session wakes when a message arrives, so sessions keep no standing check of their
own. The orchestrator's sweep is the nudge.

- **Every 10 to 15 minutes**, look for cards handed more than 15 minutes ago and not picked
  up (`state: "handed"`, no `pickedUpAt`), and asks `answered` but not closed. Nudge the
  owner with `SendMessage`. If the owner isn't in `ListAgents`, tell the person once, with
  the exact fix (e.g. "start a session and `/rename` it `bk-dev`"). Don't repeat it every
  sweep.
- **Check the relay's own threads.** Re-read the board's relay threads from the last 70
  minutes, resolved ones included. A thread the platform's auto-replier answered or
  resolved wasn't relayed: pass it on now. Never trust an auto-reply that says a session
  "isn't running".
- **Keep it cheap.** Run the sweep on Sonnet in a small fresh context, not in the long
  orchestrator chat, and only while cards are handed out. A plain board-data check runs
  first, so a model wakes only when something is past 15 minutes.
- **Overnight**, the person tells the orchestrator before leaving the team to run, and it
  keeps the sweep going until morning.

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
  the one doing it) and resolves the thread. The page sends a handoff as soon as a move
  saves, so a quick re-drag sends another; the card's place on the board is what counts. A session handing a card to another sends it a
  one-line `SendMessage` itself. If that message fails, the card is still the handoff:
  say so, and ask the orchestrator to pass it on. The orchestrator's sweep catches what
  the relay misses.
- **An auto-reply isn't the relay.** A relay thread the platform's auto-replier answered
  or resolved counts as undelivered. Only a signed "Passed to <session>" from the relay
  means it went on.
- **Comments on a Claude Doc don't reach sessions.** `@Claude` on a Claude Doc starts a
  cloud session with no project context, and nothing in the doc stops it. For anything a
  session must answer there, comment without `@Claude` and tell the session in chat (or
  let its turn check read plain comments). On a canvas, an artifact a session published,
  a comment sent to Claude does reach that session.
- **Sign every comment reply.** Every reply a session posts in a comment thread (on a
  doc, a canvas, an artifact or the board) starts with its session name and a colon:
  `bk-pm: …`. That's how the person tells the real session from a cloud Claude or an
  automatic reply, neither of which has the session's context. An unsigned reply in a
  thread didn't come from a session. It's a convention, not proof: anyone can type a name.
- **A card's type sets how it's handled.** A spike moves through the person's checkpoints, not
  the build stages: its result is new cards (linked back to the spike) and a log row, and
  the spike itself goes to Done. A bug needs steps to reproduce before anyone picks it up.
- **Moving a card on is the handoff.** Design done means moving the card to the next stage
  (Build), not just marking it done. Write all the handoff fields together.
- **A PR waiting on the person is a board ask** with `task` set to the card, never chat
  only.
- **An ask that changes a rule applied across data lists every change it makes**: each
  distinct change, with counts, worst first.
- **Several questions are one ask.** A list of proposals or questions goes on the board
  as one ask with `items`, one row each. List the same questions in chat too, numbered
  and matching the rows, so the person can answer in either place.
- **An answer given in chat needs no ask.** Log the decision; don't post an ask just to
  close it. When a chat answer settles rows of an open ask, trim or close the ask in the
  same write. Times on the board come from the clock (`date -u`), never typed by hand.
- **Save a rule the second time it comes up.** When the person corrects the same thing
  twice, the session adds the rule to where it belongs (the design system, the code
  style) right then and tells them in one line.
- **Query live data before logging a fact about it.** A log row that states a count or a
  state is checked against the data first, not written from memory.
- **A decision that constrains data gets a card** to enforce it in code, not only a log
  row.
- **A decision leaves Needs you and enters the log together.** The acting session sets the
  need's `status: "closed"` and adds the log row in one batch. An answered ask waiting only
  on results isn't a need.
- **Log numbers come from `config`.** In the same batch as the log row, update
  `config/main` to bump `nextD` or `nextA`, pinned with `if_version`. If the batch fails
  because another session took that number, reread `config/main` and retry.
- **Card numbers come from `config` too.** A new card takes `nextT` as its `num`, and the same batch bumps `nextT`, pinned with `if_version`. Refer to cards by their name (BK-12) in messages, log rows and asks.
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
