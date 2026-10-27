# reading-instrument

## What an Instrument is

The owner calls this kind of project an **Instrument**: small, clean,
single-purpose software that's beautiful and fun to use, not a full-blown
app. The agent a person already uses does the input, conversationally; the
instrument is the shell it fills. Open-source ethos, no ads. If it earns
money, it charges for hosting and sync, never for the data. More
instruments are planned (cooking, car, home maintenance).

The README has the premise. `DESIGN.md` has the full design and build
path.

## Context

- **Blogged, but not from here.** This build is written up on the owner's
  blog. Build notes and blog drafts never go in this repo.
- **Build #1 for reference:** the `SharedReadingApp` repo (clone it next to
  this one as `../bound`). An Expo + Supabase app, shelved. Reuse its rules
  (`README.md` "The streak", `supabase/migrations/0019_streak_fixes.sql`),
  not its architecture and **not its voice**. Its calm, polite copy was
  replaced on purpose; see `VOICE.md`.

## Rules

- **Private now, public later.** Keep personal details (names, devices,
  anyone's actual reading notes) out of tracked files. Real reading data
  never lives in the repo.
- **The product** is the agent as a daily reading companion (nudge → read
  → check in → notes → feedback), with pairing on top. It is not a
  data-archive tool. Backfilling old notes is an optional skill.
- **Architecture:** agent skills + one small core (MCP tools + a CLI;
  hosted, self-hosted or local) + integrations (Omarchy, in-chat cards,
  web, phone widgets).
- **Numbers are computed, words are generated.** An LLM never counts a
  streak or adds up stats. The core does.
- **It has character.** Every user-facing line follows `VOICE.md`: lively
  and specific, never bland, polite filler.
- **Working style:** push back honestly and verify claims (search before
  asserting). The owner asked for this during ideation.
