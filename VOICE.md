# VOICE.md — the instrument's character

**Status: TBD.** A first draft character (a deadpan "Registrar" running a
government office for reading) was written and rejected on 2026-10-23. It
made the owner wince. Starting over.

What's settled is the *requirement* and the *structure*. The character
itself is still open.

---

## 1. The requirement

Build #1's voice was calm, warm, direct, and forgettable. This instrument
needs **character**: a sticky, lively voice people would miss, not copy
they skim or immediately want to override.

---

## 2. Structure (decided)

- **Each reader has their own voice** for their own moments: check-ins,
  notices, milestones. Customization is part of the fun, including
  describing your own ("talk like a noir detective").
- **A pair has a referee voice** for shared moments: who filed first, a
  sealed day, a proposed house rule, the shared streak. It's separate from
  each person's own voice, and both partners hear the same one.
- **Plain** (facts only, no character) exists as a fallback until the
  characters are defined, and stays available for anyone who wants it.

---

## 3. Guardrails (carried over, revisit with the character)

- **Short:** one to three lines.
- **Numbers exact.** Streaks, pages and spare days come from the core,
  verbatim.
- **Varied:** never the same line two days running for the same moment.
- **No spoilers** past the reader's position.
- **No cruelty**, and no jokes about the relationship itself.

---

## 4. Mechanics (decided)

- The core returns a `line` with every result: the moment, written in the
  right voice (the reader's own, or the pair's referee), with numbers
  filled in.
- The agent performs it. It may restyle the line in the same voice but
  must keep every number.
- Custom voices: the core returns the Plain line plus the voice
  description, and the agent restyles it.

---

## 5. To do

- [ ] Define the default reader voice(s).
- [ ] Define the referee voice.
- [ ] Write the line bank for every moment in `SPEC.md` §2.6.
