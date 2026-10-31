---
name: design-spike
description: Plan a big chunk of work with the person as a design spike - frame the problem, set requirements, weigh several solutions (always including Option Zero, doing nothing), let the person choose, then break the choice into proposed work that other sessions can pick up. Use when the person wants to plan, scope or think through something large or unclear before anyone builds it, says "spike" or "design spike", or brings a problem that has more than one reasonable answer. Not for small changes with one obvious fix.
---

# Design spike

A design spike is a light structure for deciding what to build before building it. You
understand the problem in depth first, and only then generate and weigh solutions. It
guards against **solutioneering**: grabbing the first idea and working backwards to the
problem it solves. Claude sessions are especially prone to this, because writing code is
cheap and quick. A spike costs a little time up front and often ends in a smaller
problem, a better solution, or a deliberate choice to do nothing.

Adapted from Jaime Curtis's [The Design Spike](https://medium.com/option-zero/what-is-a-design-spike-7906385663c2).

## Who does what

- **You (the session running the spike)** do the legwork: read the code, gather context,
  draft each section, flesh out options with real numbers, and break the chosen option
  into work. You can hand research to subagents (an Explore agent to map the code, one
  per option to cost it out), but you write the document.
- **The person** decides. They agree the framing, choose the solution and accept the
  breakdown. Their checkpoints replace the article's review meetings.
- **Other sessions** (designer, dev, reviewer) are the team. They read the finished spike
  and pick up its proposed work. Ask one for an opinion only where its knowledge matters,
  e.g. the reviewer on risk, and put its answer in the doc.

## The document

One document per spike, written for readers who arrive cold: the person next week, or a
session that has never seen this conversation. Put it where the person can read it on a
phone and comment:

- a **Claude Doc** when this session has the Claude Docs connector (load its skill first);
- otherwise a Markdown file at `docs/spikes/<yyyy-mm-dd>-<slug>.md` in the project.

Link it from the project's board if it has one (a kaiban task or log row).

### Answering comments on the doc

The person reviews the spike by commenting on the doc, and **the session running the
spike answers every comment.** Don't leave it to the doc: an @Claude comment on a Claude
Doc starts a fresh cloud session that has none of the project, the repo or this
conversation. It answers anyway, and its answers are guesses. Seen on JoyRiders, 10-28.

- **Ask the person to paste the doc's link back to you.** Right after creating the doc,
  ask them in chat (one line) to paste its link so you can watch it. A session can only
  arm comment replies for a link the person gave in their own message. When they paste
  it, start a watch (`ArtifactComments`, `action: "watch"`, the doc's URL) and check
  that the watch listing shows replies armed. Then their @Claude comments wake this
  session rather than a cloud one. Watches end with the session, so if you're restarted
  mid-spike, ask for the link again.
- **Keep a slower check as a backstop**, for comments without @Claude and for times the
  watch isn't armed. Make a recurring `CronCreate` check every 10 minutes on an
  off-minute (e.g. `3,13,23,33,43,53 * * * *`). Its prompt: read this doc's comments
  and answer any new ones. Also check at the start of every turn the person sends you.
  Delete the cron when the spike is Done or Dropped.
- **Find new comments** with the Docs `query` (`object: "utterance"`), `under` the
  **tab** (`{"object": "file", "id": <tab id>}`), not the tab's body node: listing under
  the body stopped returning new threads on JoyRiders (10-30). Don't trust `afterSeq` or
  the watch alone: look at every thread's last comment, and any thread whose last word
  is the person's is waiting on you, @Claude or not. Keep the last `seq` you handled
  where the next check finds it (the kaiban card's `commentSeq`, or a line in the spike
  doc's working notes) as a shortcut, not as the test.
- **Answer each one** in its thread (Docs `create`, an utterance whose parent is the
  thread's first comment), about 40 words. A question gets an answer, checked against
  the code. A change gets made in the doc, and the reply says what changed. Start your
  first reply in a thread with your session name, so the person knows it's the session
  with the context.
- **Check what the other session did.** If a cloud session answered or edited before
  you, read its replies and edits (Docs `read` with `sinceRev`). Keep what's right, fix
  what's wrong against the code, and say so in the thread.
- **Tell the person once**, when you open the doc, that you'll answer comments here:
  right away once the watch is armed, otherwise within about 10 minutes.

Sections, in this order. Skip any that don't apply, and add any that help. Readers
matter more than the template.

1. **Status.** One line at the top: `Framing` · `Choosing` · `Breaking down` · `Done`
   (with links to the resulting cards) · `Dropped` (and why). Keep it current, since
   several sessions may read the doc.
2. **Goal.** What the spike is for. Learning something (can Neon's API do X?) is a
   different spike from choosing a solution and producing work.
3. **Context.** What a reader needs in order to have an opinion: how things work today,
   with `file:line` links, real numbers (rows, rides, emails a week), and earlier
   decisions that constrain this one. Measure where you can, and say so when you're
   estimating.
4. **Problem.** The problem in the person's words, then **reframed**. The best outcome is
   finding you don't need to solve it, and the next best is finding an easier problem
   that gets the same result. ("The elevators are too slow" becomes "people are annoyed
   by waiting".) List what's **out of scope**.
5. **Requirements.** Constraints that limit the options: cost, deadlines, the project's
   rules (CLAUDE.md, code style, security and data rules), what must keep working, and
   who has to agree (e.g. the club).
6. **Options.** At least two, plus **Option Zero: do nothing**, always listed and
   weighed honestly. Doing nothing is sometimes the best answer, and naming it makes the
   other options earn their cost. For each option: what it is, in a few sentences, what
   it costs (rough size, number of PRs, risk), what it gets right and wrong against the
   requirements, and what's unknown. Mark one as recommended, and say why.
7. **Decision.** The chosen option, who chose it and when, and the reasons in a sentence
   or two. Keep the losing options in the doc: they explain the choice later.
8. **Proposed work.** The chosen option broken into pieces another session can pick up
   cold. Each piece gets a title in the person's words, what done looks like, which
   stage it starts in (design or build), what it depends on, and its size. Prefer pieces
   that ship and can be checked on their own.

## Process

Two checks, then three checkpoints with the person. A big spike stops at each one, while a small one can
fold them into a single message. Each checkpoint is a question with options, one of
them recommended. On a project with a kaiban, ask it as a `needs` doc so the person can
answer with a click; otherwise ask in chat.

0. **Before anything else, two checks.** A card with only a title gets one question
   first: "what do you want from this?" And answer "does the app already do this?"
   with a file or a screen, written as Option Zero at the top of whatever you ask next.
   On JoyRiders, one spec was thrown away minutes after a four-option ask built from a
   title alone, and another was specced in depth before anyone saw the app already
   did the job.
1. **Frame.** Draft Goal, Context, Problem (with the reframe) and Requirements. Then ask:
   *is this the right problem?* The options might be "Yes", "Solve the reframed
   problem instead", "Narrow it to …" and "Drop it". Don't draft solutions until the
   framing is agreed: a wrong framing wastes every option written after it.
2. **Choose.** Draft the Options, including Option Zero, with your recommendation. Ask
   the person to pick one, or to ask for more options. After two rounds of options
   without a pick, ask whether today's version is good enough before drafting a third.
   Throwaway probes are fine here to
   answer a specific unknown (a query, an API call, a ten-line prototype in a worktree),
   but no real building yet.
3. **Break down.** Write the Decision and the Proposed work, then ask the person to
   accept the breakdown. When they do, turn each piece into a card (on a kaiban: in Up
   next or Later, linked to the spike, never handed to a session until the person says
   so). Log the decision, and set Status to Done with links to the cards.

Messy is normal. Notes can start as a jumble and only become a story by the end. Tidy
the document before each checkpoint, not before every edit.

## Scale it to the problem

The structure is light on purpose. A big spike might be a long doc with three rounds of
comments over a few days. A small one might be three bullets in chat (the problem, the
options with Option Zero, a recommendation) settled with one click. What never changes
is the commitment: frame the problem first, consider more than one way to solve it
(including not solving it), and choose deliberately.

## Habits

- **No production code before the Choose checkpoint.** Probes that answer a question are
  fine, and so are notes on what they found.
- **Ground options in the code.** Name the files each option touches, and check that
  claims about existing behaviour are true before relying on them.
- **Use the person's words.** Write in the project's domain words, not the code's
  names, and put numbers in place of adjectives.
- **Say what would change your mind** for the recommended option, so the person can
  push on the weak spot.
- **A dropped spike is a result.** If the answer is Option Zero, write down why, log it,
  and close the spike. That record stops the same idea coming back unexamined.
