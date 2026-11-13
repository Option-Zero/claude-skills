## Kaiban

This project's work is tracked on a kaiban: BOARD_URL. It's a live board shared by
PERSON and every Claude Code session here. Read and write it with the ArtifactData tool on
that URL (load the `kaiban` skill for the data fields). Your session name is what
`ListAgents` shows for this session, e.g. `bk-dev` (the project's short name, then the role).

- **At the start, check `ListAgents` shows you under your role name.** If it doesn't,
  ask PERSON to `/rename` this session: handoffs and relays go by that name.
- **Start each card with a fresh context** (`/clear`). The board holds what you need.
- **After a restart**, check your asks answered or closed recently against what you
  actually did, and finish anything that fell through.
- **The board is columns of stages** (`config/main` → `stages`). A stage with an owner is
  that session's queue. Each card (`tasks`) has a `stage`, an `owner` and a `state`
  (`handed` = not picked up · `doing` · `blocked` · empty).
- **When you start on a request**, query `tasks` where `owner` is your name, and `needs`
  where `actFor` is your name and `status` is `answered`. Pick up cards with
  `state: "handed"` (set `state: "doing"` and `pickedUpAt`). Act on answered asks, then
  close them and log them.
- **Mark a card picked up as soon as its relay arrives** (`state: "doing"`, `pickedUpAt`),
  even when the work waits behind what you're doing now. Otherwise the board says nobody
  has it.
- **Don't end a turn on a promise.** Nothing restarts you after a turn ends except a
  message, so "next I'll start X" at the end of a turn means X waits for hours. When there's more work you're cleared to do (the next part of a card,
  the next card in your queue), start it in the same turn. End a turn only when you're
  waiting on PERSON, on a review or on another session, or when your queue is empty,
  and say which. In long work, write a one-line progress note between steps.
- **Answers and drags reach you as messages.** When PERSON answers an ask or hands you a
  card on the board, the page sends a comment to the relay session watching the board,
  which messages you ("Kaiban relay to <you>: …"). Act on it like a handoff. You keep no
  standing board check of your own: the orchestrator sweeps for cards and answers nobody
  picked up and nudges you. If you have an hourly kaiban check from an older version of
  this block, delete it (`CronDelete`).
- **Before asking PERSON anything again in chat**, check your own `needs` on the board:
  they may have answered there already.
- **Keep your `sessions/<your name>` doc current**: `now` (one sentence), `state`
  (`working` · `waiting` · `idle`), `updated`. Update it when you start something, get
  stuck or finish, not on every turn.
- **Handing on** is moving the card to the next stage: `stage`, `owner` (that stage's
  owner), `state: "handed"`, `from` (your name), `handedAt`, in one write. Then send that
  session a one-line `SendMessage` naming the card, so it starts now. If the message
  fails, the card is still the handoff: say so, and ask the orchestrator to pass it on. PERSON may also drag cards between columns; treat where a card is as
  their call.
- **Asking PERSON something** is a `needs` doc with options (one recommended) and
  `actFor` set to you. Don't ask in chat alone. Several questions or proposals go in one
  ask as `items`, one row each; list them in chat too, numbered to match. When PERSON answers in
  chat, log the decision and skip the ask. Board times come from `date -u`, never typed. When the ask is about a card, set `task` to the card's id, so the
  ask stays parked while the card isn't in progress. A PR waiting on PERSON is always an
  ask with `task` set, never chat only. When a chat answer settles rows of an open ask,
  trim or close the ask in the same write. An ask that changes a rule applied across data
  lists every distinct change it makes, with counts, worst first.
- **Read every note on an answer.** PERSON can answer with "Just my note" (`choice:
  "note"`) or add a note to a pick. A note wins over a pick it contradicts; if it's
  unclear, ask.
- **When work ships or a decision is made**, move the card to `done` and add a `log` row
  (numbered from `config/main`'s `nextD` or `nextA`, bumped in the same batch with
  `if_version`) with its links. Query the live data before writing a fact about it into
  the log. A decision that constrains data also gets a card to enforce it in code.
- **Cards have short names** like PREFIX-12 (`num` on the card). A new card takes `config/main`'s `nextT` as its `num`, bumped in the same batch with `if_version`. Use the name when you mention a card to PERSON or another session.
- **A card's `detail` is for PERSON; its `notes` are for sessions.** `detail` is a short
  overview in plain words, two or three sentences: what it is, why, and what done looks
  like. Everything only another session needs (files, PRs, decision numbers, steps,
  edge cases, accounts) goes in `notes`, which the card shows folded under "For
  sessions". Read both when you pick a card up.
- **Cards have a type**: `type` is `spike`, `bug` or empty (a story). A **spike** is
  planning, not building: load the `design-spike` skill, work through its checkpoints with
  PERSON, link the spike doc on the card, and finish by creating the cards it proposes
  (then move the spike to Done). A **bug** carries steps to reproduce, what happens and
  what should happen; it can go straight to Build. Set the type when you add a card.
- **A card in a Spec stage gets planned before it's designed or built.** The Spec
  stage's owner runs the `design-spike` skill on it, scaled to the card: a full spike doc
  for a spike or anything big, a short spec on the card (a needs ask with options, then
  the result in `detail`) for a small story or bug. A spike ends in new cards and goes to
  Done. A story or bug moves on to the next stage once PERSON accepts the spec.
- **Merging** follows the merge rule in `CLAUDE.md`. Where it lets a PR merge without an
  ask, merge and add an A row to the log instead of asking.
- **Load your role skill** if there is one for your role (`pm-role`, `ui-designer-role`,
  `dev-role`; review uses `reviewer-role` as a fresh subagent per PR).
- **Any to-do you'd otherwise keep in memory** goes on the board in the Later stage.
- **On a Claude Doc, comments with `@Claude` don't reach you**: they start a cloud session
  with no project context. Ask PERSON to comment without `@Claude` and tell you in chat,
  or read the doc's plain comments yourself. On a canvas you published, a comment sent to
  Claude does reach you.
- **Check a board page change in a browser** before calling it live.
