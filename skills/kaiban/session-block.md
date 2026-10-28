## Kaiban

This project's work is tracked on a kaiban: BOARD_URL. It's a live board shared by
PERSON and every Claude Code session here. Read and write it with the ArtifactData tool on
that URL (load the `kaiban` skill for the data fields). Your session name is what
`ListAgents` shows for this session, e.g. `joyriders-dev`.

- **The board is columns of stages** (`config/main` → `stages`). A stage with an owner is
  that session's queue. Each card (`tasks`) has a `stage`, an `owner` and a `state`
  (`handed` = not picked up · `doing` · `blocked` · empty).
- **When you start on a request**, query `tasks` where `owner` is your name, and `needs`
  where `actFor` is your name and `status` is `answered`. Pick up cards with
  `state: "handed"` (set `state: "doing"` and `pickedUpAt`). Act on answered asks, then
  close them and log them.
- **Answers and drags reach you as messages.** When PERSON answers an ask or hands you a
  card on the board, the page sends a comment to the relay session watching the board,
  which messages you ("Kaiban relay to <you>: …"). Act on it like a handoff.
- **Keep an hourly safety-net check** in case the relay isn't running: at the start of any
  request, if `CronList` shows no kaiban check, create one with `CronCreate` (recurring,
  hourly on an off-minute, e.g. `17 * * * *`) with this prompt: "Kaiban check: on
  BOARD_URL, query tasks where owner is <your name> and state is handed, and needs where
  actFor is <your name> and status is answered. Pick up or act on anything there. If
  there's nothing, end the turn in one line." It fires only while you're idle and lasts 7
  days; this rule recreates it after that. If you have a 10-minute check from before,
  delete it (`CronDelete`) and make the hourly one.
- **Before asking PERSON anything again in chat**, check your own `needs` on the board:
  they may have answered there already.
- **Keep your `sessions/<your name>` doc current**: `now` (one sentence), `state`
  (`working` · `waiting` · `idle`), `updated`. Update it when you start something, get
  stuck or finish, not on every turn.
- **Handing on** is moving the card to the next stage: `stage`, `owner` (that stage's
  owner), `state: "handed"`, `from` (your name), `handedAt`, in one write. Then send that
  session a one-line `SendMessage` naming the card, so it starts now rather than at its
  next check. PERSON may also drag cards between columns; treat where a card is as
  their call.
- **Asking PERSON something** is a `needs` doc with options (one recommended) and
  `actFor` set to you. Don't ask in chat alone.
- **When work ships or a decision is made**, move the card to `done` and add a `log` row
  (numbered from `config/main`'s `nextD` or `nextA`, bumped in the same batch with
  `if_version`) with its links.
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
- **Any to-do you'd otherwise keep in memory** goes on the board in the Later stage.
