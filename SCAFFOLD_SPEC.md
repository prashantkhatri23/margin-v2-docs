# Margin v2 — Scaffold & Onboarding Spec (one-time bootstrap)

This is the spec for the **one-time** per-user setup: creating the repo and
conducting the onboarding session. It is NOT a skill (skills are for recurring
behaviour; this runs once). It elaborates `MARGIN_V2_SPEC.md` §14 — on any
apparent conflict, the main spec wins.

Two entry paths, sharing all scaffold steps:

- **Path A — fresh user (the common case):** scaffold → onboarding interview →
  first consolidation → first review.
- **Path B — migration from v1:** scaffold → convert v1 sources (spec §15) →
  short settings check-in → first consolidation → first review. No interview:
  the migrated onboarding session already *is* the interview evidence.

## Absolute requirement

The user types **zero Git commands** and sees **zero Git concepts** ("commit",
"branch", "merge", "PR" as a technical term). They answer natural-language
questions and review a letter. If any step would require the user to touch
Git, that step is wrong.

## 1. Pre-flight

- Verify Git + an authenticated GitHub identity (`gh auth status`), BYO model.
- Create a **private** repo (private is mandatory — this is someone's life
  story). If GitHub is unreachable, proceed local-only and say so plainly; the
  local-proposal fallback (spec §10.1) keeps everything working.
- Onboarded-ness is gated on **interview completion**, not scaffold existence
  (spec §14.1): a completed onboarding session + a timezone in `config.yml` →
  already onboarded, switch to normal operation. Scaffold present but interview
  unfinished (a crashed bootstrap) → **resume the interview session** where it
  left off; never re-scaffold, never wedge.

## 2. Scaffold

Create the exact spec §3.1 tree:

- `AGENTS.md` from `templates/AGENTS.md`; `CLAUDE.md` as a symlink (or a
  one-line `@AGENTS.md` import file if symlinks are unavailable).
- Empty `index.md` (header + `okf_version: "0.1"` note) and `log.md` (header
  only).
- `.margin/`: `spec_version.txt` = `2.0.0`; `config.yml` from the spec §3.3
  defaults with empty user fields; `counters.json` = `{"s": {}}`; empty
  `ui_state.yml`.
- `.gitignore` containing `.margin/work/` and `.margin/lock`.
- `.gitkeep` in every otherwise-empty directory (`system/`, all seven `memory/`
  kind folders, `working_memory/` subfolders, `archive/`, `sessions/`).
- **No memory pages, no system pages yet.** `system/` starts empty — its five
  pages are *born from the onboarding evidence* via the first PR, not from
  stub templates. (Deliberate v1 change: v1 seeded stubs because empty pinned
  files degraded runs; v2's first consolidation runs minutes later, so stubs
  would only add noise the user never asked for.)
- Scaffold commit: `chore(bootstrap): scaffold margin repo (spec 2.0.0)`. (The
  onboarding session follows in subsequent commits — spec §14.5's "initial
  commit" refers to this whole bootstrap sequence, of which this is the first.)

## 3. The onboarding interview (Path A — defined fully)

A normal session in every mechanical respect (`_shared/session_mechanics.md`
applies: turns persisted immediately, checkpoints, running summary):
`skill: onboarding`, `consolidation: eligible`. Its ID is allocated with the
**system timezone** (no user timezone exists yet — spec §4.4), noted in
`meta.yml`. It is the one session with truly zero context — no temporal file,
no index, no pinned pages exist yet.
Do not pretend otherwise; "I don't know you yet — that's what this
conversation is for" is the honest, warm opening frame.

### 3.1 The interview's three jobs

1. **Practical setup** → `.margin/config.yml` (settings are not memory; write
   them directly as answers land).
2. **Seed evidence for the five `system/` pages** → consolidated minutes later
   into the first PR.
3. **Teach the two rituals** — entries/conversations in, a review letter back —
   in one or two plain sentences each, woven in, never a lecture.

### 3.2 Coverage checklist (judgement in the conduct, not the list)

Warm conversation, one question at a time, order adapted to how they answer —
never a form. By the end, aim to have touched:

| Area | Feeds |
|---|---|
| What to call them; timezone; locale | `config.yml` + `user_profile` |
| Why journaling, why now; what they want Margin to *be* for them | `companion_contract` |
| A little of who they are and what this stretch of life is about | `user_profile`, `current_chapter` |
| Preferred tone; listening vs. speaking up proactively | `communication_preferences` |
| What's off-limits or to be handled gently | `boundaries_and_sensitivities` |
| Safety-resource surfacing opt-in (**default off**; ask plainly, no pressure) | `config.yml` |
| Nudge appetite (`nudge_after`) and schedule interest (Mode B — only configure with informed consent, spec §9.1) | `config.yml` |

Depth follows the user. If they open up into real autobiography, receive it
generously — it is first-class life-share evidence and the §9.4 allowance
applies. If they're terse, take what's given and stop: **system pages may
start thin; they grow through use.** An onboarding that extracts is worse than
one that under-collects. Anything not covered simply isn't known yet — the
companion voice already knows how to say that.

Safety path (spec §13) applies from turn one, before any memory exists.

### 3.3 First consolidation — immediately, user present

Run `margin-consolidate` on the onboarding session, with two deliberate
differences from a routine run:

- **It creates the five `system/` pages** (normally §9.4 gates `system/`
  behind confirmation — here the confirmation *is* the review the user is
  about to do, minutes after saying the words). Each page: evidence refs into
  the onboarding turns, honest confidence, `user_confirmed: false` until
  review affirms.
- **Be thorough, stay conservative.** Thorough = don't leave clearly-stated
  onboarding material unprocessed; it also seeds any first memory pages the
  evidence genuinely supports (a named person, a current struggle), the
  initial `temporal_context.md`, and the index. Conservative = tiers still
  bind; single-mention hunches go to `working_memory/candidates/`, not pages.
  Deferring almost everything and producing a thin first letter is a *valid*
  outcome for a terse interview.

### 3.4 First review — the taught ritual

Chain straight into `margin-review` while they're present. Preface once, in
plain words: "Before anything becomes part of how I remember you, you get it
as a letter like this — you say yes, no, or correct me. That's always how it
works." Then run the normal §10.3 flow. Merging their first letter minutes
after their first conversation is the moment the product's promise becomes
concrete — do not defer it to tomorrow.

## 4. Path B — migration variant

1. Scaffold (§2) identically.
2. Convert every v1 source per spec §15 (turn/paragraph anchors preserved,
   `migrated_from:` recorded, onboarding session marked `skill: onboarding`).
3. **Settings check-in, not an interview:** carry what the v1 repo knows
   (display name, timezone, safety opt-in from v1 `config.yml`); ask only the
   genuinely new v2 preferences (nudge appetite, schedule) plus confirm
   timezone. Two minutes, not twenty.
4. First consolidation over the migrated sessions (§3.3 rules apply — it seeds
   `system/` from the migrated onboarding evidence exactly as a fresh
   interview would; respect the per-run cap if there are many sessions).
5. First review (§3.4), same ritual. Log the migration in `log.md`.

## Done when

- The §3.1 tree exists; `.margin/` metadata valid; `AGENTS.md`/`CLAUDE.md` in
  place; initial commit clean.
- The onboarding (or migrated) session folder is complete and committed:
  turns, meta, summary, highlights.
- `config.yml` holds the user's actual answers, including explicit safety and
  nudge decisions.
- The first PR was reviewed **in the CLI** — merged, corrected, or declined —
  and was written in the letter voice. (A legitimately-empty first run with
  everything deferred to `working_memory/` is also a valid completion; do not
  block on a PR that honestly wasn't warranted.)
- `system/` pages that exist carry evidence refs into the onboarding turns;
  `temporal_context.md` and `index.md` exist and are current.
- The user never saw a Git concept and typed no Git.
