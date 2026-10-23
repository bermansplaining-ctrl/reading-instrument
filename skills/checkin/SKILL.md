---
name: reading-checkin
description: Log a reading session to the reader's reading instrument. Use when the reader says they read, sends a photo of a book page, mentions where they stopped, or responds to a daily reading notice. Keeps their streak, their place in the book, and their partner's shared streak.
---

# Reading check-in

The reader keeps a reading streak, and possibly a shared streak with a
partner, in their reading instrument. Checking in should take seconds and
feel like a small pleasure, not a form.

## When to use this

- They say they read: "read 20 pages," "finished my half hour," "done for
  tonight."
- They send a photo of a book page.
- They answer the daily notice.
- They say where they stopped: "I'm on page 112."

## The flow

1. **Call `get_status`** to see their current books, whether they've
   already checked in today, their partner's day, and any house rules.
2. **Work out the book.** If they have exactly one book marked reading,
   use it. If it's unclear, ask *once*: "Which one, *Less Than Zero* or the
   Aristotle?" If the book is new, call `find_book`, confirm the edition in
   a few words, then `set_book` with status `reading`.
3. **Work out the position.** Use the page they said, or read the page
   number from their photo. Ebooks use a percentage, audiobooks minutes or
   a chapter. If you can't tell, check in without a position rather than
   guessing.
4. **Ask for the photo if they haven't sent one** (it's the suggested
   default, and what their partner sees). If a house rule *requires* a
   photo or a minimum number of minutes, ask for exactly what's missing.
   If `check_in` returns an `upload_url` and you can't send the file
   yourself, give them the link.
5. **Call `check_in`.**
6. **Deliver the `line`** the tool returns, in the voice it's written in.
   You can restyle it in that same voice, but keep every number exact.
   Optionally add *one* question about the reading ("Anything stand out
   tonight?").

If they answer with thoughts about the book, file them with `add_notes`.
Keep their words, and pick the kind (`thought`, `quote`, `comparison`,
`vibe`, `question`).

## The voice

**TBD.** See `VOICE.md` in the instrument's repo. Each reader will have
their own voice, and a pair will have a referee voice for shared moments.
Until those are defined, lines come back Plain (facts only).

- **Short:** one to three lines.
- **Never:** spoilers past their page, cruelty, or jokes about the
  relationship itself.

## Rules

- **Never compute streaks, spare days or progress yourself.** The tools
  return them. Repeat them; don't adjust them.
- **"I read last night"** before noon: check in with `for_yesterday`.
  After noon it counts for today. Say so honestly.
- **Corrections:** if they say you got it wrong, use `fix_check_in`. Don't
  create a second check-in.
- **Pages aren't the gate.** Any check-in counts for the day, unless a
  house rule says otherwise.

## Examples (Plain voice, until the characters exist)

**Reader:** read 20 pages of less than zero, stopped on 112
**You:** *(get_status → check_in: book, position 112, pages_read 20)*
"Checked in. Day 12. Page 112 of 208. Sam hasn't checked in yet. Want to
add a photo of the page?"

**Reader:** *(sends a photo of page 64 of Aristotle's Ethics)*
**You:** *(reads "64" from the photo → check_in with the photo)* "Page 64,
photo attached. You both read today: Day 31."

**Reader:** oh I read last night but forgot
**You:** *(before noon → check_in with for_yesterday)* "Logged for last
night. Day 9."
