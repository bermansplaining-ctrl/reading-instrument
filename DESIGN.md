# DESIGN.md — reading-instrument

Rewritten 2026-10-23. The first draft (archive ingestion, nightly
gatherers) misread the product. This version is built from the owner's own
walkthroughs.

Status: Phase 0 (design).

---

## 0. What it is

**An instrument that turns the AI agent you already talk to into a reading
companion.** It nudges you to read every day. It takes your check-in (a
photo of the page you stopped on). It keeps your streak and your place in
the book, turns your passing thoughts into notes, pulls in the notes you
took elsewhere, and goes deeper when you want it to. If you have a reading
partner, it holds you to each other.

**Solo works. Paired is better.**

It ships as three things: **a set of agent skills, one small core, and
visuals.** It is not an app you open to fill in.

---

## 1. The loop, as the owner described it

### A. Fiction: *Less Than Zero*

1. **Nudge.** At your agreed time, your agent reminds you to read.
2. **Read.** Half an hour, the book in your hands.
3. **Check in.** "Read 20 pages, stopped on 112," or just a photo of the
   page. The streak ticks. The agent knows you're at 112 of 208.
4. **Talk.** No formal notes, just thoughts: the POV, the flat affect, how
   it reminds you of other works, a line that stuck. The agent files them
   on the book as thoughts, quotes, comparisons and vibes.
5. **Later.** "Recommend me a movie." The agent knows what stood out to you
   in what you read, not just which titles you finished.

### B. Nonfiction: Aristotle

1. You read with notes on paper, in Apple Notes, or typed on the laptop.
2. **Check in**, then: "my notes for today are in that Apple Note / this
   Drive doc / this photo."
3. The agent **ingests** them, links them to today's session, and the core
   stores them.
4. If you want more: **Socratic mode**, meaning questions, counterarguments
   and connections, grounded in *your* notes.

### C. Paired

1. **Invite** by link or QR code. Your partner joins from whatever agent
   they use.
2. You each see the other's streak, current book and check-in photo.
3. **The shared streak** counts a day when *both* of you check in (Build
   #1's mutual gate).
4. You can pass each other books, and recommendations draw on both your
   tastes.
5. Partners see each other's streak, current book and check-in photos.
   Notes stay private unless you share one.

---

## 2. Anatomy: skills, core, visuals

### Skills: what the agent knows how to do

Portable `SKILL.md` files, written once and used by any agent that supports
skills, or embedded as instructions where an agent doesn't.

| Skill | Triggered by | Does |
|---|---|---|
| `checkin` | "I read," or a photo of a page | Logs the session, page and proof; replies with streak + progress |
| `remind` | Setup, or "remind me at 9" | Schedules the daily nudge with the agent's own scheduler, falling back to the core |
| `notes` | Talking about the book | Files thoughts, quotes, comparisons and vibes on the right book |
| `ingest` | "My notes are in …" | Pulls notes from a photo, a doc, a note or a file and links them to the session |
| `discuss` | "Let's go deeper" | Socratic conversation grounded in your notes |
| `taste` | "What should I read/watch next?" | Computed signals plus interpretation; recommendations that show their reasoning |
| `pair` | Invite, "how's she doing?" | Joins pairs, shows partner status, sends nudges, passes books |
| `backfill` *(optional)* | "Import my old notes" | Brings in an existing archive (e.g. years of Drive notes) |

### Core: the source of truth

One small server holding books, sessions, streaks (solo and paired),
progress, notes and photos. It's exposed as **MCP tools**, the standard way
chat apps and agents call outside tools, plus a **CLI** for terminal
harnesses.

- **Numbers are computed, words are generated.** Streaks, progress and
  stats are computed here, never by a model.
- **Self-host or hosted.** It runs on your own machine for the Omarchy
  crowd, or as a hosted service for everyone else. Same code.
- **The record is exportable**, always. You own it.

### Integrations: same data, many surfaces

Agents are where you *talk* to the instrument. Integrations are where you
*see* it. Each integration is a thin client of the core.

| Integration | Shows |
|---|---|
| **Web page** (MVP) | Both streaks, current shelves, today's check-ins. Works on any phone as a home-screen bookmark |
| **Omarchy** | A Waybar streak module, plus a dashboard: current shelf, favorites shelf, partner list with their streaks |
| **Inside chat** | Interactive cards for the streak, shelf and progress, rendered in the conversation (ChatGPT apps, the open MCP Apps standard) |
| **Phone widgets** | Android home-screen widgets (Nothing OS and others), later iOS. Each one is its own small integration |

---

## 3. The streak and proof of life

- **The gate is "checked in today,"** not pages and not minutes. Pages
  differ by edition, and audiobooks have none (Build #1's lesson). Pages and
  minutes are logged as *progress*, not as the gate.
- **Proof is a photo of the page you stopped on.** One photo gives you
  proof, the page number (the agent reads it) and the book. It's the
  fastest check-in possible. Optional when solo; the default when paired,
  because it's what your partner sees, the way a Strava map or a snap is.
- **Spare days are earned** (from Build #1): miss a day without losing the
  run, so ordinary life doesn't kill the streak.
- **Honest limits:** this is social proof, not anti-cheat. Photos are
  personal data: visible to your partner, never public, deletable by
  whoever took them.

---

## 4. Where it runs (platform reality, October 2026)

### Why the core has to be online

ChatGPT and Muse run in their companies' clouds. They can only reach a tool
that's on the public internet over HTTPS. A server on your laptop is
invisible to them. Pairing needs a shared place for two people anyway. So
the core is hosted from the MVP onward.

### Who hosts it

| Who | Setup | For |
|---|---|---|
| **Nobody** (local mode) | The core runs on your own machine | Solo, terminal-only users. Later |
| **The user** (self-host) | Their own small server | Omarchy folks who want cloud agents or pairing without depending on us. Optional |
| **Us** (hosted) | Connect your agent with a URL and a key. Nothing to install | Everyone else. The public version, and the eventual paid tier |

For the MVP, "us" means one small hosted backend the owner runs for two
people, most likely a Supabase project on the free tier (see §8).

### Agents

| Agent | How it connects | Daily nudge | Status |
|---|---|---|---|
| **Meta Muse** (MVP, partner) | Custom connector, created by asking Muse in chat. Works best with an MCP server over HTTPS plus an API key (bearer token); OAuth-only services are clunky. Muse usually saves it as a skill | Meta AI added recurring tasks in July 2026 (select markets): verify in the app. Fallback: the core emails the reminder | ✓ MVP. Meta doesn't review custom connectors. A payment card is required even on the free tier, and connector use counts against the usage meter |
| **ChatGPT** (MVP, owner on Pro) | An app: an MCP server + UI components. Private testing first, directory listing later (check its auth requirements) | Scheduled tasks (Go/Plus/Pro and up, push notifications) | ✓ MVP |
| **Claude Code, Pi** | Skills + the same MCP endpoint, or a CLI | Desktop notification | Phase 2 |
| **Hermes Agent** | Skills + MCP; messaging gateway | Built-in scheduler | Later |
| **Claude app** | Connector (MCP) | Unverified | Plausible |
| **Grok** | Unknown | | Not a target |

---

## 5. Build path

| Phase | Build | Done when |
|---|---|---|
| **0. Design** | Record schema, skill text, MCP tool list, hosting choice | The plan holds up against both walkthroughs in §1 |
| **1. MVP: two agents, one streak** | Hosted core (MCP over HTTPS, per-person API keys); `checkin`, `notes`, `remind` and `pair` as connector instructions; photo check-in; one web page showing both streaks and shelves | Two weeks of both people checking in, one from **Muse** and one from **ChatGPT**, without either opening a form |
| **2. Terminal + Omarchy** | Skills for Claude Code and Pi; Waybar module and dashboard | The owner logs from the terminal and sees it in the bar |
| **3. Depth** | `ingest` (photos, docs, notes), `discuss`, `taste`/recommendations, optional `backfill` | The companion half is complete |
| **4. Omarchy release** | Public repo, self-host docs, skills published to skill hubs; r/omarchy, X, Show HN | The community version |
| **5. Public release** | Hosted for everyone, ChatGPT app directory, Muse connector, more integrations (Android widgets and others), privacy policy and terms; paid tier = hosting + sync + pairing | Usable without GitHub |

**Why the MVP is two cloud agents:** the first pair is already a mixed
test, and both of their agents (Muse and ChatGPT) live in the cloud. So the
hosted core comes first. The public version isn't a separate product either:
it's the same core, opened up and listed.

**Decided: start clean, and port Build #1's rules rather than its tables.**
Its backend is shaped around a phone app: login sessions, per-user database
rules, minutes as the streak gate, a three-pact limit, 25 migrations of
history. The new core has different bones: agents calling with API keys,
check-ins as the gate, photos as first-class data. Carry the hard-won rules
over as tests:

- The day belongs to the pair's time zone.
- Today is grace: an unfinished today never breaks a live run.
- Spare days are earned (one per 7 completed days, up to 3).
- Joining a pair is race-safe (no three-person pairs).
- Shared history survives a pair ending.

---

## 6. Prior art (searched October 2026)

**The pieces exist. The combination doesn't, as far as we could find.**

| Category | Examples | What they do | What they don't |
|---|---|---|---|
| Reading apps with AI added | Basmo (ChatBook), Bookly, Cognito, BookPal, Chapterly | Timers, streaks, AI Q&A and summaries about the book; Chapterly does Socratic questioning | App-first, i.e. Build #1 plus a chatbot. The AI knows the *book*, not *you*. No agent integration, no partner |
| Platform AI in e-readers | Kindle "Ask this Book," "Story So Far" | Spoiler-free answers that know your position | Kindle ebooks only, so it can't see paper books. No conversation-built notes, no streak |
| Reading data inside agents | Readwise's official MCP server (2026); a community Goodreads MCP | Claude and ChatGPT can search your highlights and library | Digital highlights, not a daily ritual. No check-in, streak or partner. **The closest incumbent: one feature away** |
| Agent skills | Bookworm, Reading Log, reading-logs, Book Study | Bash loggers; a reading coach | Thin: generic logging, no visuals, no proof, no pair. Proof that a SKILL.md alone is a weekend project |
| Social and accountability | BookBuds, StoryGraph buddy reads, Fable, BuddyUp | Friend streaks, live sessions, clubs, generic partner habits | No agent, no conversation-built notes |

What it means:

- **The gap exists mostly because the plug-in surfaces are months old.**
  ChatGPT apps (renamed from connectors, Dec 2025), Readwise's MCP server
  (2026), Muse custom connectors (Sept 2026). The window is open now. It
  won't stay open.
- **Readwise, Amazon and Meta are each one feature away from a slice of
  this.** What none of them cover together: **paper books, notes built
  from conversation, a partner, and working with any agent.**
- **A skill file alone is not a product.** The value is in the core (the
  streak, the pair, the record) and the integrations.

---

## 7. Honest value and risks

**Strong:**

- The ritual (nudge → read → check in → feedback) is proven by Duolingo,
  Snapchat and Strava. The new part is that the agent runs it.
- A photo check-in may be the lowest-friction reading log in the category.
- Conversation as note-taking solves "I never take the notes I mean to."
- The taste record is portable. It makes *every* agent better at knowing
  you as a reader, and it travels with you.
- The partner turns a habit tool into a relationship ritual.

**Risks:**

1. **You're a guest in someone else's house.** OpenAI and Meta control
   distribution and could ship a native reading mode. The defense: works
   with any agent, you own the record, and the pair network.
2. **The nudge is the weakest link.** If the reminder doesn't fire, the
   loop dies. Never depend on one agent's scheduler; the core sends its own
   fallback.
3. **Check-in friction.** It has to take 15 seconds or less. Photo first.
4. **Unverified platform details:** Muse's developer portal (secondary
   sources only), and what each ChatGPT plan allows for private apps.
5. **Hosting brings back some business obligations:** a privacy policy,
   data deletion, app review. Lighter than an App Store launch, but not
   zero.
6. **Intimate data.** Photos of your nightstand and your private thoughts
   about books. The defaults have to be private.

---

## 8. Open questions

1. ~~Which agents for the MVP?~~ Answered: the partner tests on Muse, the
   owner on ChatGPT Pro.
2. Hosting. **Leaning Supabase** (owner, 2026-10-23): one shared hosted
   backend for everyone. Postgres for the record, Storage for photos,
   `pg_cron` for reminders, and the MCP endpoint as an Edge Function that
   checks a per-person API key itself. (Supabase's own MCP guide doesn't
   cover auth yet, but a bearer-key check is plain code, and it's what Muse
   prefers anyway.) Trade-off: self-hosters would run their own Supabase,
   which works but is heavier than a single container.
3. ~~Reuse Build #1's backend?~~ Decided 2026-10-23: start clean (see §5).
4. ~~What do partners see?~~ Decided 2026-10-23: streak, current book and
   check-in photos. Notes stay private unless shared.
5. Names for the core and the CLI.
