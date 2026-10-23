# SPEC.md — Phase 0

Status: draft, 2026-10-23. This is the spec for **Phase 1, the MVP**: two
people, two cloud agents (Muse and ChatGPT), one streak. `DESIGN.md` covers
the why. This covers exactly what to build.

---

## 1. Vocabulary

| Term | Meaning |
|---|---|
| **Reader** | A person. Has a display name and a time zone |
| **Agent** | Whatever a reader talks to (Muse, ChatGPT, Claude Code…). Each agent gets its own key |
| **Book** | A title in the shared catalog (from Open Library when possible) |
| **Shelf entry** | One reader's relationship to one book: status, format, position, rating |
| **Check-in** | "I read today." Optionally with a book, a position, minutes, a photo |
| **Note** | Something a reader said about a book: a thought, quote, comparison, vibe… |
| **Pair** | Two readers holding each other accountable. One active pair per reader |
| **Day** | A calendar date in a time zone: the reader's for solo, the pair's for paired |
| **Run** | The streak: consecutive counted days, bridged by spare days |

---

## 2. The rules (the deterministic core)

Everything in this section is computed by the core, never by a model. It
ports Build #1's streak algorithm (`bound/supabase/migrations/0019_streak_fixes.sql`),
with one change: **a check-in replaces the minutes goal as the gate.**

### 2.1 What counts as a day

- **Solo:** a day counts if the reader has at least one check-in on that
  date (reader's time zone).
- **Paired:** a day counts if **both** members have at least one check-in
  on that date (pair's time zone). One partner alone never counts for the
  pair, though it still counts for their own solo run.
- Multiple check-ins on the same day count once. Minutes and pages are
  progress, not the gate, unless a house rule makes them one (§2.7).

### 2.2 Whose clock

- A check-in stores `created_at` (UTC). Its **solo day** is that moment in
  the reader's time zone. Its **pair day** is that moment in the pair's time
  zone.
- The pair's time zone is the creator's at pairing time, and can be changed
  later. Known limitation: long-distance partners in different zones share
  one clock (same as Build #1).

### 2.3 Late check-ins

"I read last night but forgot to tell you." A check-in can be marked
`for_yesterday` if it's logged **before 12:00 local time** the next day. It
counts for yesterday and is shown as *logged this morning*. After noon,
spare days are what cover a gap.

### 2.4 The run (ported algorithm)

Walk forward from the first counted day to **yesterday**:

```
perfect = 0; spares = 0; earn = 0; graced = []
for each day from first_counted_day to yesterday:
  if day counts:
    perfect += 1; earn += 1
    if earn >= 7: spares = min(spares + 1, 3); earn = 0
  elif spares > 0:
    spares -= 1; graced.append(day); earn = 0
  else:
    perfect = 0; spares = 0; earn = 0; graced = []
if today counts: perfect += 1          # today is never a miss, only pending

run (shown to people) = perfect + len(graced)
```

Also report `yesterday`: `met`, `graced`, `missed` or `no_data`, which
drives the recovery copy.

### 2.5 Test cases (port these first)

| # | Situation | Expected |
|---|---|---|
| T1 | First check-in ever | run 1 |
| T2 | Checked in yesterday, not yet today | run unchanged, `yesterday: met` (today is grace) |
| T3 | 7 consecutive counted days | 1 spare earned |
| T4 | A miss with 1 spare | run continues, spare spent, day listed in `graced`, run = perfect + graced |
| T5 | A miss with 0 spares | run resets (0, or 1 if today counts) |
| T6 | 28 straight days | spares capped at 3 |
| T7 | A spare is spent | earn counter resets to 0 |
| T8 | Pair: only one partner checked in | pair day not counted; that partner's solo run still counts |
| T9 | Check-in at 23:30 reader-local that's 00:30 pair-local | counts for the pair's *next* day |
| T10 | `for_yesterday` at 11:00 / at 13:00 | counts for yesterday / rejected, counts for today |
| T11 | Two people join the same code at the same instant | exactly one succeeds |
| T12 | Pair ends | past pair days still computable; ex-partner can't see current shelf |
| T13 | A check-in is deleted | run recomputed |
| T14 | Three check-ins on one day | counts once |
| T15 | A house rule is accepted today | applies from tomorrow; today is judged by the old rules |
| T16 | `photo_required` is in effect; a check-in has no photo | the day doesn't count until a check-in with a photo arrives |
| T17 | `min_minutes: 20` with check-ins of 10 + 15 minutes | counts (minutes are summed across the day) |
| T18 | `min_minutes` is in effect; a check-in has no minutes | doesn't count toward the minimum; the agent asks how long |

### 2.6 Day states and the `line`

The core returns a machine-readable `state` plus a `line`: the moment
written in the reader's chosen voice, with the numbers filled in. The
instrument has a character. **`VOICE.md` is the source for every line**,
and the agent performs it (it may restyle the line but must keep the
numbers).

States: `neither`, `me_done`, `partner_done`, `both_done`, `solo_done`,
`solo_open`, plus the events `recovered` (yesterday covered by a spare),
`reset` (the run ended), `milestone`, `first_ever`, `late`, `finished`,
`abandoned`, `pair_joined`, `rule_proposed`, `rule_accepted`.

### 2.7 House rules

The default is **photo suggested**: the agent asks for one, and a check-in
without a photo still counts. On top of that, readers can tighten their own
rules, and pairs can agree on rules for the pair. Customizing it is part of
the fun.

| Rule | Effect |
|---|---|
| `photo_required` | A day only counts once a check-in with a photo exists |
| `min_minutes: N` | The day's check-in minutes, summed, must reach N |
| `min_pages: N` | The day's pages read, summed, must reach N |

- **Solo:** the reader sets their own rules.
- **Paired:** either partner proposes, the other seconds through their
  agent, and the rule binds both of them for the pair streak.
- **Rules never apply retroactively.** A change takes effect from the next
  day, and each day is judged by the rules in effect that day. This is
  Build #1's goal-locking lesson: you can't soften the bar at 23:50 to save
  a run.

---

## 3. Data model

Postgres on Supabase. Columns are a sketch; types will be settled in the
first migration.

| Table | Columns | Notes |
|---|---|---|
| `readers` | id, display_name, timezone, email?, voice (`registrar`, `coach`, `gremlin`, `plain`, `custom`), voice_custom?, created_at, deleted_at | Not tied to Supabase Auth in the MVP. Agents authenticate with keys |
| `api_keys` | id, reader_id, label (`muse`, `chatgpt`, `terminal`…), key_hash, prefix, created_at, last_used_at, revoked_at | One key per agent per reader. Only the hash is stored |
| `books` | id, open_library_key?, title, author, year?, page_count?, cover_url?, created_at | Shared catalog, deduplicated by Open Library key |
| `shelf_entries` | reader_id, book_id, status (`reading`, `finished`, `paused`, `abandoned`, `want`), format (`print`, `ebook`, `audio`), position?, position_unit (`page`, `percent`, `minute`, `chapter`), page_count_override?, started_on?, finished_on?, rating? (0.5–5 in half stars), favorite, updated_at | Progress is format-aware, so audiobooks work |
| `checkins` | id, reader_id, book_id?, created_at, for_yesterday, position?, position_unit?, pages_read?, minutes?, note?, photo_id?, source (key label), deleted_at | The gate. Days are derived, not stored |
| `photos` | id, reader_id, checkin_id, storage_path, mime, bytes, width, height, status (`pending`, `ready`), created_at, deleted_at | Stored with location data stripped (§4) |
| `notes` | id, reader_id, book_id, checkin_id?, kind (`thought`, `quote`, `comparison`, `vibe`, `question`, `summary`), body, page?, source (`conversation`, `ingested`), source_ref?, created_at, deleted_at | Private to the reader in the MVP |
| `pairs` | id, invite_code (6 chars, no look-alike characters), timezone, created_by, created_at, ended_at | |
| `pair_members` | pair_id, reader_id, joined_at, left_at | Max 2, enforced with a row lock on join (T11) |
| `rules` | id, scope (`reader` or `pair`), scope_id, rule (`photo_required`, `min_minutes`, `min_pages`), value?, proposed_by, seconded_by?, effective_from, retired_from? | Versioned. Each day is judged by the rules in effect that day (§2.7) |
| `lines_shown` | reader_id, moment, variant, shown_on | So the same line never repeats two days running (VOICE.md §5) |
| `reminders` | reader_id, local_time, channel (`agent`, `email`, `off`), last_sent_on | |
| `share_pages` | token, reader_id, created_at, revoked_at | Private links for the web page (§8) |

---

## 4. Photos

**Visibility:** the reader and their active partner. Never public.
Deletable by whoever took it.

**Privacy on upload:** strip EXIF metadata. iPhone photos carry GPS
coordinates, and a photo of your nightstand should never reveal where your
nightstand is.

**Three ways in, one backend.** Whichever the agent can do:

| Path | Who | How |
|---|---|---|
| **File parameter** | ChatGPT | `check_in.photo` is declared in `_meta["openai/fileParams"]`. ChatGPT passes a short-lived `download_url` (about 5 minutes) and the core fetches it immediately |
| **Direct upload** | Agents with file access (Muse's cloud VM, terminal agents) | `check_in` returns a signed, single-use `upload_url` (15 min). The agent `PUT`s the bytes |
| **Upload page** | Anyone, as a fallback | The same `upload_url` opened in a browser shows one button: *Add today's photo* |

Accept JPEG, PNG, HEIC and WebP up to 10 MB. The default house rule is
*photo suggested*: the agent asks, and a check-in without a photo still
counts. A reader or pair can make photos required (§2.7).

**When a pair ends:** check-in *history* stays (who read when), but each
person's photos become visible only to them again.

---

## 5. Identity and auth (MVP)

- **API keys:** `ri_` + 32 random characters, one per agent per reader,
  shown once, stored hashed, revocable.
- **Sending the key:** as `Authorization: Bearer <key>` (Muse's preferred
  method), or in the URL path, `/mcp/k/<key>`, for clients that can't set
  headers. A secret URL is acceptable for two people. A public listing
  (Phase 5) needs proper OAuth.
- **Issuing keys:** a small owner script creates a reader and their keys.
  There's no public signup in the MVP.

---

## 6. The MCP surface

One endpoint: a Supabase Edge Function at `/functions/v1/mcp`, using the
official MCP TypeScript SDK over streamable HTTP. Every tool acts as the
reader the key belongs to.

### 6.1 Server instructions

MCP servers can send `instructions` that clients may add to the agent's
system prompt. ChatGPT recommends the **first 512 characters stand on
their own**. Draft:

> You're connected to the reader's reading instrument: their streak, their
> place in each book, their notes, and (if paired) a shared streak with
> their partner. When they mention reading, call `get_status`. When they
> say they read, call `check_in` and ask for a photo of the page they
> stopped on. Never compute streaks yourself. Speak as the Instrument, in
> the reader's chosen voice, using the `line` each tool returns: brief,
> lively, numbers exact, never cruel, never a spoiler.

The full `skills/checkin/SKILL.md` follows that paragraph.

### 6.2 Tools

| Tool | Use when | Input | Returns |
|---|---|---|---|
| `get_status` | Start of any reading conversation; after a reminder | — | Today (checked in?), solo run, pair run + partner's today (book, photo), current books with progress, house rules, `state` + `line` |
| `find_book` | The reader names a book the shelf doesn't have | `query` (title, author or ISBN) | Up to 5 candidates: title, author, year, pages, cover. The reader's shelf is searched first |
| `set_book` | Starting, finishing, pausing or abandoning a book; rating; favoriting | `book` (id or Open Library key), `status`, `format?`, `page_count?`, `rating?`, `favorite?` | The shelf entry |
| `check_in` | "I read" | `book?`, `position?` + `unit`, `pages_read?`, `minutes?`, `photo?`, `note?`, `for_yesterday?` | Check-in id, which day it counted for, whether the day's rules are met, updated runs, progress, `state` + `line`, and `upload_url` if no photo was attached |
| `fix_check_in` | The reader corrects a mistake | `checkin_id`, then a field to change or `delete: true` | The updated runs |
| `add_notes` | The reader shares thoughts, quotes, comparisons, vibes | `book`, `notes[]`: `kind`, `body`, `page?`, `source_ref?` | Note ids |
| `get_notes` | Recall: "what did I think of…" | `book?`, `kinds?`, `query?`, `limit?` | The reader's notes |
| `get_shelf` | "What am I reading?" / "What's she reading?" | `who` (`me` or `partner`), `status?` | Books with progress, ratings, favorites |
| `pair_invite` | "Invite someone to read with me" | — | Code, join link, QR image URL |
| `pair_join` | The reader has a code | `code` | Pair status, including the partner's name |
| `set_reminder` | "Remind me at 9" | `time` (HH:MM), `channel` (`agent`, `email` or `off`) | Stored setting, plus the exact daily task for the agent to create in its own scheduler |
| `set_voice` | "Talk to me like a coach" / "be a noir detective" | `voice` (`registrar`, `coach`, `gremlin`, `plain` or `custom`), `description?` (for custom) | The new voice, demonstrated in one line |
| `propose_rule` | "Let's make photos required" / "20 minutes minimum" | `rule`, `value?`, `scope` (`me` or `pair`) | Solo: active from tomorrow. Pair: pending until the partner seconds it |
| `answer_rule` | The partner responds to a proposed rule | `rule_id`, `accept` (yes or no) | The rule's status and when it takes effect |

**Defaults:**
- `check_in` without a book uses the reader's only `reading` book if there
  is exactly one. Otherwise the agent asks.
- A position past the page count is clamped and flagged, not rejected.

---

## 7. Reminders

**Phase 1 uses each agent's own scheduler.** ChatGPT has scheduled tasks
(the owner is on Pro). Meta added recurring tasks to its assistant in July
2026, which is to be verified in Muse.

`set_reminder` returns the task text for the agent to schedule, in the
reader's voice (VOICE.md, "Notices"). For example:

> Every day at 21:00: call `get_status`. If I haven't checked in, send me
> the notice it returns ("NOTICE: Reading hours are now in session…") and
> ask me to report back with the page I stopped on or a photo of it.

The escalating second and final notices need the core to send them, so
they come after Phase 1.

**Fallback, only if an agent's scheduler proves unreliable:** a `pg_cron`
job every 15 minutes emails readers whose reminder time has passed, who
haven't checked in, and who haven't already been reminded today.

---

## 8. The web page (first integration)

`GET /p/<token>`: one private page per pair (or per reader when solo).
Phone-friendly, so it can be saved to a home screen.

- Two columns, you and your partner: run, today's state, current book with
  a progress bar, today's photo, and the last 7 days as dots.
- Build #1's visual language carries over: blackberry = you, teal = your
  partner, marigold = shared, and the bookmark pair as today's state.
- Read-only. The link is the credential, and it can be revoked.

---

## 9. Pairing

1. A reader asks their agent to invite someone. `pair_invite` returns a
   6-character code, a join link and a QR code.
2. The partner tells their agent "join with code ABC123," and `pair_join`
   runs it.
3. The join takes a row lock: no third member, ever (T11). One active pair
   per reader in the MVP.
4. Leaving ends the pair for both. History stays computable; the
   ex-partner loses access to the current shelf (T12). Photos go back to
   private (§4).

---

## 10. Onboarding runbooks (MVP)

**Owner, on ChatGPT Pro.** Turn on developer mode and add the MCP server
URL (secret-path form). Check `get_status` in a chat, then ask ChatGPT to
create the daily reminder task. (Menu names change often. Verify in the
spike.)

**Partner, on Muse.**
1. In a Muse chat: *"Connect a custom connector. It's an MCP server at
   `<url>`. It's my reading instrument; use it whenever I talk about
   reading."*
2. Enter the key when Muse asks, in its secure credential prompt, not in
   the chat.
3. Muse usually saves the connection as a skill.
4. *"Every day at 9pm, remind me to read."*
5. *"Join my reading pair with code ABC123."*

---

## 11. Phase 1 spikes: prove these first

Each one is yes/no, tested before building anything that depends on it.

| # | Question | If no |
|---|---|---|
| S1 | Does a Supabase Edge Function serve MCP over streamable HTTP to **both** ChatGPT and Muse? | Small container on a VPS instead |
| S2 | Do ChatGPT (secret path) and Muse (bearer header) authenticate? | Try the other key method on each |
| S3 | Can each agent deliver the photo? ChatGPT via file parameter, Muse via a `PUT` from its VM, and the upload page on iPhone Safari | Upload page as the default |
| S4 | Can the edge runtime strip EXIF (and handle HEIC)? | Convert or strip in a separate small worker |
| S5 | Do Muse's recurring task and ChatGPT's scheduled task fire daily and lead into a check-in? | Email fallback (§7) |
| S6 | Does each agent follow the server instructions without a pasted skill? | Paste the skill once at setup |

---

## 12. Not in Phase 1

Taste and recommendations, Socratic discussion, note ingestion from other
apps, backfill, the Omarchy integration (Phase 2), sharing individual notes,
groups or multiple pairs, public signup, OAuth, payments.

**Before anyone outside the two of us uses it:** account deletion and a
data export. Build #1 learned these are launch blockers, not
nice-to-haves.

---

## 13. Decisions and open questions

Decided 2026-10-23:

- Late check-ins count for yesterday until **noon** (§2.3).
- When a pair ends, **photos go back to private** (§4).
- **House rules:** photo suggested by default; photo-required and minimums
  available (§2.7).
- Ratings in **half stars**.
- **The voice has a character** (`VOICE.md`), replacing Build #1's calm
  tone.

Open:

1. Is the Registrar the right default character, or should the first
   check-in let each reader pick a voice?
2. Should a pair be able to set one shared voice for pair moments (the
   referee), separate from each person's own?
