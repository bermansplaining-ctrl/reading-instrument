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

- **Build log and blog draft:** `blog/2026-10-22-blog-notes.md`. Add a
  dated entry to its "Process log" each working session; this project is
  being blogged.
- **Build #1 for reference:** the `SharedReadingApp` repo (clone it next to
  this one as `../bound`). An Expo + Supabase app, shelved. Reuse its rules
  and copy voice (`README.md` "The streak", `lib/pactState.ts`,
  `lib/streak.ts`), not its architecture.

## Rules

- **Private now, public later.** Keep personal details (names, devices,
  anyone's actual reading notes) out of tracked files, except `blog/`,
  which holds raw build notes. Review `blog/` before the repo goes public.
  Real reading data never lives in the repo.
- **The product** is the agent as a daily reading companion (nudge → read
  → check in → notes → feedback), with pairing on top. It is not a
  data-archive tool. Backfilling old notes is an optional skill.
- **Architecture:** agent skills + one small core (MCP tools + a CLI;
  hosted, self-hosted or local) + integrations (Omarchy, in-chat cards,
  web, phone widgets).
- **Numbers are computed, words are generated.** An LLM never counts a
  streak or adds up stats. The core does.
- **Working style:** push back honestly and verify claims (search before
  asserting). The owner asked for this during ideation.
