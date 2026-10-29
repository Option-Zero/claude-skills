# Option Zero's Claude Code skills

Two skills for working with a team of Claude Code sessions.

- **kaiban**: a live board that several Claude Code sessions share with you. It's a
  published claude.ai artifact with its own database. Each column is a stage owned by a
  session (spec, design, build, review), and dragging a card into a column hands it to
  that session. It also holds your questions to answer with a click, what each session
  is doing, a map of the system and a log of decisions.
- **design-spike**: plan a big piece of work before anyone builds it. Frame the problem,
  weigh several options (always including Option Zero, doing nothing), choose, then
  break the choice into cards. Adapted from
  [The Design Spike](https://medium.com/option-zero/what-is-a-design-spike-7906385663c2).

## Install

In Claude Code:

```
/plugin marketplace add Option-Zero/claude-skills
/plugin install option-zero-skills@option-zero
```

To get updates later, run `/plugin marketplace update option-zero`.

## What you need

- **kaiban**:
  - A claude.ai account with Artifacts, since the board is a published artifact with
    its own database.
  - The sessions running on one machine, so they can message each other.
  - One session that publishes the board and keeps watching it. Comments the board
    sends to Claude (your answers, your drags) reach that session, which passes each
    one on to the session it's for.
  - To start a board, ask a session to "set up a kaiban for this project".
- **design-spike**: nothing extra. With the Claude Docs connector, the spike is written
  as a Claude Doc you can comment on. Without it, the spike is a Markdown file in your
  project. Ask a session to "run a design spike on …".

## License

MIT. See [LICENSE](LICENSE).
