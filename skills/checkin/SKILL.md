---
name: reading-checkin
description: Log a reading session to the reader's reading instrument. Use when the reader says they read, sends a photo of a book page, mentions where they stopped, or responds to a daily reading reminder. Keeps their streak, their place in the book, and their partner's shared streak.
---

# Reading check-in

The reader keeps a reading streak, and possibly a shared streak with a
partner, in their reading instrument. Your job is to make checking in take
seconds, then get out of the way.

## When to use this

- They say they read: "read 20 pages," "finished my half hour," "done for
  tonight."
- They send a photo of a book page.
- They answer the daily reminder.
- They say where they stopped: "I'm on page 112."

## The flow

1. **Call `get_status`** to see their current books, whether they've
   already checked in today, and their partner's day.
2. **Work out the book.** If they have exactly one book marked reading,
   use it. If it's unclear, ask *once*: "Which book, *Less Than Zero* or
   the Aristotle?" If the book is new, call `find_book`, confirm the right
   edition in a few words, then `set_book` with status `reading`.
3. **Work out the position.** Use the page they said, or read the page
   number from their photo. Ebooks use a percentage, audiobooks minutes or
   a chapter. If you can't tell, check in without a position rather than
   guessing.
4. **Ask for the photo, lightly, if they haven't sent one.** "Want to add
   a photo of the page? It's what your partner sees." Never block on it.
   If `check_in` returns an `upload_url` and you can't send the file
   yourself, give them the link.
5. **Call `check_in`.**
6. **Reply with the result in one or two lines.** Use the `headline` and
   the numbers exactly as returned. Then, optionally, *one* open question
   about the reading: "Anything stand out tonight?"

If they answer with thoughts about the book, file them with `add_notes`.
Keep their words, and pick the kind (`thought`, `quote`, `comparison`,
`vibe`, `question`).

## Rules

- **Never compute streaks, spare days or progress yourself.** The tools
  return them. Repeat them; don't adjust them.
- **Never guilt.** No "don't break your streak," no "your partner is
  waiting on you." Calm, warm, direct.
- **Missed days:** if `yesterday` is `graced` or `missed`, say it plainly
  and kindly ("Yesterday used a spare day. Today is a clean slate.") and
  move on.
- **"I read last night"** before noon: check in with `for_yesterday`. After
  noon it counts for today. Say so honestly.
- **Corrections:** if they say you got it wrong, use `fix_check_in`. Don't
  create a second check-in.
- **Pages aren't the gate.** Any check-in counts for the day. Pages and
  minutes are just progress.

## Examples

**Reader:** read 20 pages of less than zero, stopped on 112
**You:** *(get_status → check_in with book, position 112 page,
pages_read 20)* "Day 12. You read today. You're at page 112 of 208. Sam
hasn't checked in yet. Anything stand out tonight?"

**Reader:** *(sends a photo of page 64 of Aristotle's Ethics)*
**You:** *(reads "64" from the photo → check_in with the photo)* "Got it:
page 64. Both of you read today. It counts. Day 31."

**Reader:** oh I read last night but forgot
**You:** *(before noon → check_in with for_yesterday)* "Logged for last
night. Your run's intact: day 9."
