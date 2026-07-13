# Using Margin — a plain guide

Margin is your journaling companion. Your memory repo *is* the app: open a
coding agent inside that folder and you're talking to Margin. Everything it
remembers about you lives there as readable files, and nothing becomes memory
without your yes.

## Which agents does this work with?

Margin is **not** built for Claude Code specifically — it's built on plain
markdown, git, and the cross-vendor `AGENTS.md` convention. Any capable coding
agent that reads `AGENTS.md` (Claude Code, Codex, Cursor's CLI, and the like)
can be Margin: the schema file tells the agent what this repo is, where the
skills and the spec live, and how to behave. Claude Code additionally picks up
the skills natively from `.claude/skills/`; other agents reach the same skill
files through the pointers in `AGENTS.md`. What you need on the machine: git,
your agent of choice, and (optional but recommended) the GitHub CLI `gh` —
without it Margin still works fully, it just delivers its review letters
locally instead of as GitHub pull requests.

## First-time setup (you don't have a Margin yet)

If you're starting from the documentation repo (`margin-v2-docs`), onboarding
is a conversation, not an install script:

1. Clone the docs repo, and create an empty folder wherever you want your
   memory to live (it will become a private git repo — it's your life story,
   keep it private).
2. Open your coding agent in that empty folder and say:
   > "Read `MARGIN_V2_SPEC.md` and `SCAFFOLD_SPEC.md` from ../margin-v2-docs
   > and bootstrap a new Margin repo for me here (Path A). I'm a new user."
3. The agent scaffolds the repo and then **interviews you** — a warm
   conversation, not a form: what to call you, your timezone, why you want to
   journal, what's off-limits, how you want it to speak to you, and your
   safety preference. Answer in plain words; skip anything you like.
4. Minutes later you get your **first letter** — "here's what I heard about
   you" — and review it item by item (see *review* below). Merging that letter
   is the moment your Margin starts knowing you. You never touch git at any
   point.

Already journaled in an older Margin (v1)? Say "migrate my v1 repo (Path B)"
instead of step 2 — your old entries come across verbatim and get a fresh,
better consolidation.

## Daily starting (you already have a Margin)

The one-word way (recommended — set up once, see below):

```
margin
```

Or by hand:

```
cd <your-margin-repo>
claude        # or codex, or whichever agent you use
```

then say anything — "hi" is enough. (Coding agents wait for your first
message; they can't speak unprompted. The `margin` alias sends that first
"hi" for you, so the home screen appears immediately.)

Either way, Margin opens with the **home screen**: a personal hello, one
thread picked up from your recent life — offered as an invitation, never a
to-do reminder — anything waiting for you (a letter to review, entries to
consolidate), and the menu of things you can do. You never need exact
commands — plain words work: "let's journal", "I want to tell you about
something from my childhood", "just want to talk".

**One-time alias setup** — add this line to your `~/.zshrc` (adjust the path,
and swap `claude "hi"` for your agent's equivalent if you use a different one):

```
alias margin='cd /Users/prashantkhatri/Documents/margin/margin_v2_test && claude "hi"'
```

## The six things you can do

**chat** — just talk. About your day, a decision, anything. *Chats are private
by default*: Margin keeps the transcript (so you can pick a conversation back
up) but will NOT fold it into long-term memory unless you say "remember this
one" — or it asks you, once, when a conversation touched something real, and
you agree.

**journal** — you write, Margin files it. Type freely, or point it at a draft:
"file the entry at ~/notes/tuesday.md" — text, images, whatever. Your words
are kept verbatim, forever. Afterwards it offers *once* to help refine
(typos / structure / a deepening question) and drops it if you say no.

**guided** — Margin leads. It asks about your day, one question at a time,
using what it knows to ask better questions. At the end it writes the entry
*for* you — in your voice, from your words — and shows it to you. It only
counts once you say "yes, that's what I said."

**share** — for the past, not today. "I want to tell you about my Kota years"
/ "let me tell you about Tipsy." Margin pulls up what it already knows so it
can ask the questions a stranger couldn't, and the result becomes a life-share
entry, confirmed by you, dated to *when it happened*.

**consolidate** — the memory pass. Margin reads everything new since last
time and writes what it noticed into its memory pages — but as a *proposal*,
never directly. It ends with a letter to you. Run it when the home screen
says a few entries have piled up, or whenever you like.

**review** — reading that letter. Numbered items, least-certain first. Answer
in plain words, item by item or all at once:

- `1 yes` — that's me *(Margin may now say it plainly in conversation)*
- `2 no` — drop it *(it won't be re-pitched unless something genuinely new appears — and then it must say so)*
- `3 yes, but it was my aunt` — your correction wins, always, no argument
- `4 later` — set it aside; it may come back naturally
- or just talk: "the sister part is wrong, she's older than me…"

Only after this review does anything become memory. **This is the whole
trust model in one sentence: Margin proposes, you decide.**

## The rhythm that works

1. Live your life; drop in whenever — a two-line journal entry, a guided
   session, a chat. Abandoning a session mid-way is fine; nothing is lost, and
   "continue" picks any conversation back up where it trailed off.
2. Every few days (or when nudged), say **"consolidate"**.
3. Next time you're in, read the letter and answer it. Merge done — memory
   updated, exactly as you approved it.

## Good to know

- **It's all just files.** Open this folder in Obsidian and wander the graph
  of your own life. Every claim on every page links to the exact words you
  said, so you can always check *why* Margin believes something.
- **Confidence shows.** Things you've confirmed, Margin states plainly.
  Things it's inferred, it hedges ("I might be off, but…"). It's built to say
  "I don't know that about you yet" instead of making things up.
- **Sensitive things move slowly.** Anything touching who you are, your
  history, or tender ground always requires your explicit confirmation — and
  Margin never applies clinical labels to you, full stop.
- **You never touch git.** Margin does all of it. If you ever see a git word,
  something's wrong — tell it to run **doctor**, the health check that
  verifies every link, page, and promise in the repo (worth running once a
  month or after anything crashes).
- **Deferring is respected.** Say "another day" about any topic and Margin
  notes the door without pushing on it.

## Quick reference

| You say | What happens |
|---|---|
| *(nothing — just open it)* | Home screen |
| "just want to talk" | Chat (private unless you say "remember this") |
| "let's journal" / "file this: …" | Your entry, verbatim, kept |
| "guide me through today" | Guided reflection → entry in your voice |
| "let me tell you about …" | Past-recall session, enriched by memory |
| "consolidate" / "do a memory pass" | Reads new entries → writes you a letter |
| "review" / "show me the letter" | Item-by-item review → memory updates |
| "continue" | Pick a previous session back up |
| "doctor" | Repo health check |
| "what do you remember about me?" | An honest, evidence-backed answer |
