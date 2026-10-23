# reading-instrument

Turns the AI agent you already talk to into a reading companion. If you
want, it also holds you and a reading partner to each other.

**Status:** Phase 0 spec drafted ([`SPEC.md`](SPEC.md)). No code yet.
Design and roadmap: [`DESIGN.md`](DESIGN.md).

## What it does

- Nudges you to read every day.
- Takes your check-in: a photo of the page you stopped on.
- Keeps your streak and your place in the book.
- Turns your passing thoughts into notes, and pulls in the notes you took
  elsewhere.
- Goes deeper when you want: Socratic discussion, and recommendations
  grounded in what actually stood out to you.
- Pairs you with a partner: a shared streak, and each other's shelves.

## Principles

1. **The agent is the interface.** No forms.
2. **Skills, one small core, and visuals.** That's the whole instrument.
3. **Numbers are computed, words are generated.**
4. **Solo works. Paired is better.**
5. **You own the record**, and it works with any agent.

## Lineage

Build #2. Build #1 was a standalone Expo app, *Same Chapter* (`../bound`):
built, audited, never shipped. This repo exists to compare the two
approaches.
