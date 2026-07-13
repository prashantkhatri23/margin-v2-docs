# Margin v2 — Foundation Specification

**Audience:** an autonomous coding/orchestration agent (e.g. Claude Code) that will
(a) scaffold a per-user memory repository, (b) run the interactive session skills,
and (c) run the recurring consolidation + review loop against it.

**Status:** v2 / POC. Self-hosted ("BYO Claude Code + GitHub"). One private Git repo
per user. CLI-first: the terminal *is* the product surface for this stage; a GUI
comes later and will render the same repo artifacts.

**Relationship to v1:** this document **supersedes** `MARGIN_AGENT_SPEC.md` (v1).
The v1 architecture is the foundation and most of its reasoning stands; v2 changes
the interaction model (session-first), the source container (uniform session
folders), the memory taxonomy (pages born on demand), and adds the temporal layer,
the in-CLI review flow, and LLM-wiki/OKF alignment. Where this spec is silent, v1's
reasoning applies. Where they conflict, this spec wins.

**This document is the source of truth for mechanics.** Where it gives an exact
rule (ID format, write order, watermark semantics), follow it literally — do not
re-derive or "improve" it; downstream skills depend on these exact contracts.
Judgement (voice, classification, what deserves promotion) lives in the skill
files, not here.

---

## 0. What Margin is, in one paragraph

Margin is a journaling companion that lives in the terminal. A user talks to it —
free-writing, guided journaling, sharing memories from their past, or just
conversing — and every interaction is preserved verbatim as an immutable session.
Separately, Margin maintains a small interlinked wiki of "memory pages" that
synthesise who the user is over time. Memory pages are never invented from nothing
and never silently changed: every non-trivial claim cites the exact words it came
from, and every change to derived memory is proposed to the user as a warm,
reviewable letter (a Pull Request under the hood) that the user reviews *in the
CLI* before it becomes durable.

---

## 1. Non-negotiable invariants

These hold in every code path. If an instruction elsewhere appears to conflict
with one of these, the invariant wins and the agent must stop and surface the
conflict.

1. **Source is immutable.** Session turns, confirmed entries, and attachments are
   append-only evidence. Once written, their content is never edited by the agent.
   Each session file has exactly one designated writer and designated mutable
   fields (§4.9); everything else is frozen.
2. **No derived claim without evidence.** Every non-trivial statement in a memory
   page carries `evidence_refs` pointing at stable source anchors (§5). When in
   doubt, cite.
3. **Derived memory changes go through a PR.** The agent never commits a change
   under `memory/` or `system/` directly to the default branch. Those land only
   via a reviewed Pull Request (§10). Source filing, session files, and generated
   artifacts (index, log, temporal context) commit directly — they are safe
   because they only add evidence or are regenerable.
4. **The user is the only reviewer.** This is their life. The PR is a consent and
   accuracy gate, reviewed and merged from inside the CLI (§10); the user never
   visits GitHub and never types Git.
5. **Identity moves slowly.** A single session never rewrites `memory/identity/`
   or `memory/life_periods/`. Identity-touching changes require repeated evidence
   across ≥2 sources or explicit user confirmation (§9.4).
6. **Confidence is always explicit.** Every memory page declares `confidence`
   (`low|medium|high`) and `user_confirmed` (`true|false`). Low-confidence
   inference is never presented as fact — in pages, PRs, or conversation.
7. **Idempotent processing.** Re-running consolidation over already-processed
   turns must not create duplicates. The mechanism is the per-session
   consolidation watermark (§4.8) plus the update-first procedure (§6.3).
8. **No diagnosis, no hidden profiling.** No clinical labels or psychological
   classifications, ever. Patterns are described in the user's own framing.
9. **Assistant words are not evidence about the user.** Only the user's own turns,
   their confirmed entries, and their attachments are evidence. The agent's turns
   — including its own past reflections and praise — are never citable as facts
   about the user (§5.2).
10. **Structure grows deliberately.** Memory pages are created only through the
    update-first decision procedure (§6.3). A `create` without a written
    justification is a defect.
11. **Everything user-facing is a rendered repo artifact.** The home screen, the
    review letter, the "what do you know about me" answer — all are renders of
    files (or PR bodies) that live in or hang off the repo. Nothing user-facing
    depends on state outside it. This is what lets a future GUI replace the CLI
    without redesign.
12. **Safety overrides reflection.** Acute-risk signals trigger the safety path
    (§13); consolidation of that session pauses. Support first, memory never.

---

## 2. Design principles (not rules, but binding direction)

- **Repo as truth.** Any fresh agent session, given only the repo + skills, can
  resume all work. Harness-native session resumption (§4.7) is an optimization,
  never the mechanism of record.
- **The CLI is an app.** Terminal output is designed like a UI: a stable home
  screen (§11), quiet defaults, one nudge at a time, actions always named. Not a
  log stream.
- **v1 constrained the structure and left the judgment free; v2 frees the
  structure and constrains the judgment.** No pre-baked pages; instead, a fixed
  decision procedure, per-kind page contracts, and lint.
- **Warmth is a contract, not decoration.** PR letters, nudges, and the companion
  voice are product surfaces held to product quality. "Update
  threads/rest_guilt.md" is a defect; "I think I'm noticing something about how
  you rest" is the standard.
- **Interoperate by convergence.** The repo is a valid LLM-wiki/OKF bundle (§3.2)
  so open-source tooling (Obsidian, renderers, lint tools) works on it — but
  Margin's own contracts always take precedence over ecosystem conventions.

---

## 3. Repository layout

### 3.1 The tree

The bootstrap creates exactly this. Directories that start empty carry `.gitkeep`.
**No memory pages are pre-created** — every page under `memory/` is born from the
user's life via consolidation (§6.3).

```text
AGENTS.md                  # the schema file: repo purpose, page contracts, workflows (§3.2)
CLAUDE.md                  # symlink → AGENTS.md (or one-line @AGENTS.md import)
index.md                   # GENERATED memory index, OKF shape (§8.2); never hand-edited
log.md                     # append-only run log, newest-first, ## YYYY-MM-DD headings (§3.2)

.margin/
  spec_version.txt         # "2.0.0" — agent refuses to run on mismatch (§17)
  config.yml               # user settings (§3.3)
  counters.json            # monotonic ID counters (§4.4); cache — auto-rebuilds if lost
  ui_state.yml             # GENERATED: last nudge / greeting-hook dates (§9.1, §11)
  open_proposal.yml        # present on main ONLY while a proposal is open (§9.7, §10.1)
  proposal.yml             # exists ON THE INSIGHT BRANCH only: the proposal manifest (§9.2)
  lock                     # untracked advisory lockfile (§4.13)
  work/                    # .gitignore'd — consolidation scratch workspaces (§9.2)

system/                    # PINNED. Always loaded. Tiny & stable. Scaffold leaves this
                           #   EMPTY (.gitkeep); the five pages below are created by the
                           #   FIRST consolidation PR from onboarding evidence (§14).
  companion_contract.md
  user_profile.md
  communication_preferences.md
  boundaries_and_sensitivities.md
  current_chapter.md       # the ONLY current-chapter page (closed chapters → memory/life_periods/)

sessions/                  # SOURCE. One folder per session, all skills. Direct commits.
  <session_id>/
    turns.jsonl            # append-only raw turns (§4.2)
    meta.yml               # session-owned metadata (§4.3)
    summary.md             # running narrative summary (§4.5)
    highlights.md          # checkpointed bullets, turn-anchored (§4.5)
    citations.md           # memory files consulted during the session (§4.6)
    consolidation_state.yml# consolidation-owned watermark (§4.8) — created by first run
    entry.md               # journaling kinds only: the journal-equivalent entry (§4.10)
    attachments/           # files the user shared, copied in (§4.11)

memory/                    # DERIVED. PR-gated. Pages born on demand, one level deep.
  identity/                # who they are: values, self-concept, orientation-of-life pages
  people/                  # one page per human
  places/                  # one page per meaningful place
  threads/                 # one page per named cross-cutting pattern
  events/                  # one page per discrete event / milestone
  domains/                 # one page per broad life area (work, family, …)
  life_periods/            # one page per CLOSED chapter of life

working_memory/            # Auto-committed. Low ceremony.
  temporal_context.md      # GENERATED recency layer (§7)
  candidates/              # single-source hunches awaiting corroboration
  pending_confirmations/   # held until explicit user "yes"
  rejected/                # anti-nag record of declined proposals (§9.6)

archive/                   # Superseded/merged pages. Never delete; move here (via PR).
```

Pinned facts to internalise:

- `sessions/` = **evidence**. Append-only, uniform container for every kind of
  interaction. There is no separate `journal/` or `conversations/` tree — a
  free-write journal entry is simply a short session whose product is `entry.md`.
- `memory/` = **the wiki**: dense living pages with evidence lists, change logs,
  and links to each other. Rewritten in place, but only via PR.
- `working_memory/` = **the holding pen** plus the temporal layer. Auto-committed;
  deliberately low ceremony so hunches don't prematurely harden into identity.

### 3.2 LLM-wiki / OKF alignment

Margin repos are valid Open Knowledge Format (OKF v0.1) bundles and follow the
LLM-wiki layering (immutable sources → maintained wiki → schema file). Concretely:

- **Frontmatter.** Every memory page, system page, and `entry.md` carries YAML
  frontmatter with at minimum: `type` (OKF-required), `title`, `description`,
  `tags`, `timestamp` (ISO 8601, last substantive update). Margin's own fields
  (`id`, `aliases`, `confidence`, `user_confirmed`, `evidence_refs`, `status`)
  ride alongside as legal custom extensions — OKF consumers must preserve unknown
  fields.
- **Reserved files.** `index.md` is the generated catalog (§8.2), root entries in
  OKF list shape (`* [Title](path) - description`), with `okf_version: "0.1"`
  noted at its head. `log.md` is the append-only run log: newest-first,
  `## YYYY-MM-DD` date headings, one short line per event (session committed,
  consolidation run proposed/merged/rejected, lint run, counter rebuild,
  migration).
- **The schema file.** `AGENTS.md` at the repo root documents: what this repo is,
  the folder map, every page `type` and its one-line contract (§6.1), the
  update-first procedure (§6.3), evidence rules (§5), and a pointer to this spec
  + `spec_version`. It is what makes *any* schema-file-aware coding agent able to
  maintain the repo. `CLAUDE.md` is a symlink (or a one-line `@AGENTS.md`
  import) so Claude Code picks it up natively.
- **Links.** Standard **relative Markdown links** are canonical:
  `[Dadi](../people/dadi.md)`. No `[[wikilinks]]` (mechanical, lossless rewrite
  if ever needed for Obsidian-native tooling). Links render on GitHub, satisfy
  OKF, and open cleanly in Obsidian as-is.
- **Citations.** Evidence anchors (§5.1) are Margin's canonical provenance.
  Memory pages additionally end with a human-readable `## Evidence` section
  listing the refs as links. Mirroring anchors as `path:line-range` for ecosystem
  lint tools is a documented post-POC option, valid because sources are
  append-only.

### 3.3 `.margin/config.yml`

```yaml
display_name: ""
timezone: ""                 # drives all ID dates (§4.4)
locale: ""
safety_resource_surfacing: false   # opt-in only (§13)
index:
  source_window_days: 120    # rolling session window in index.md (§8.2)
temporal:
  staleness_days: 3          # rebuild trigger (§7.3)
  staleness_sources: 3       # rebuild trigger (§7.3)
  size_budget_lines: 120     # hard budget for temporal_context.md (§7.2)
consolidation:
  nudge_after: 5             # home-screen/session-start nudge threshold (§9.1)
  schedule: null             # opt-in cron expression (Mode B, §9.1); null = manual
  max_sessions_per_run: 12   # catch-up cap (§9.7)
```

---

## 4. The session container (the uniform source layer)

Every interaction with Margin — a chat, a guided journaling session, a life-share,
a free-written entry, onboarding, even a review sitting — is one **session
folder** under `sessions/`. One container type; `meta.yml: skill` says which skill
produced it. This is the v2 replacement for v1's `journal/` + `conversations/`
split and its three submission paths.

### 4.1 Life of a session

1. User invokes a skill (from the home screen or directly). The agent allocates a
   session ID (§4.4), creates the folder, writes `meta.yml`.
2. Every turn is persisted to `turns.jsonl` **immediately** — the user's turn on
   receipt, the agent's turn once produced. The working tree is the real-time
   crash-safety layer.
3. At checkpoints (§4.5) the agent refreshes `summary.md`, appends to
   `highlights.md`, logs consulted memory in `citations.md`, updates `meta.yml`
   counters, and **commits the session folder** (batched — not per turn; §4.12).
4. Sessions **do not end**. There is no close command and no timeout with
   semantics. A session simply stops receiving turns; if the user returns to it
   (§4.7), new turns append with the next unused turn numbers. Anchor numbering
   is permanent — renumbering is data corruption.

### 4.2 `turns.jsonl` — the raw turns file

Append-only. One JSON object per line:

```json
{"t": 14, "role": "user", "ts": "2026-07-13T21:04:00+05:30", "text": "…", "attachments": ["att_1"]}
{"t": 15, "role": "assistant", "ts": "2026-07-13T21:04:31+05:30", "text": "…"}
```

- `t` is the turn number: monotonically increasing from 1, never reused, never
  renumbered — it is the anchor (`#t14`).
- `role ∈ {user, assistant, system_note}`. `system_note` is for rare agent-written
  markers (e.g. "entry confirmed at this point", safety hold) — never for content.
- `attachments` lists attachment IDs introduced by this turn (§4.11).
- Existing lines are never edited. Corrections happen by the user saying so in a
  later turn — which is itself evidence.
- **Append atomically; recover from truncation.** Each append is one write of a
  complete line ending in a newline. On opening a session file, validate the
  tail: a truncated final line (crash mid-write) is never edited or removed —
  append a newline, then a `system_note` turn marking the previous line void,
  and continue from the last parseable `t`. The tidy-up rule (§4.12) performs
  this validation before committing anything.

### 4.3 `meta.yml` — session-owned metadata

Written **only** by the session's own skill (single-writer rule, §4.9):

```yaml
id: s_2026_07_13_001
skill: conversation        # conversation | journal | guided | share_event | onboarding | review
title: ""                  # optional; user-supplied or agent-proposed, human phrasing
description: ""            # one plain sentence, kept current as the session evolves (§8.2 signpost)
created_at: 2026-07-13T21:02:11+05:30
updated_at: 2026-07-13T21:40:02+05:30
turns: 42
consolidation: not_eligible   # not_eligible | eligible | safety_held  (§4.8)
harness_session_id: ""     # optional; native-resume fast path (§4.7)
entry_status: null         # journaling kinds: null | draft | confirmed (§4.10)
```

### 4.4 IDs (exact — do not vary)

`s_<YYYY>_<MM>_<DD>_<NNN>`

- Single prefix `s` for all sessions regardless of skill (the skill lives in
  `meta.yml` and the index).
- Date = the session's creation date in the user's timezone (`config.yml`), not
  UTC. If no timezone is configured yet (the onboarding session itself), use the
  system timezone and note it in `meta.yml`. `NNN` = zero-padded per-day
  sequence from `001`.
- Counters live in `.margin/counters.json` (`{"s": {"2026-07-13": 2}}`).
  Allocation = read → increment → write → use, performed **under the lockfile**
  (§4.13) so two concurrent terminals cannot allocate the same ID. Counters
  never decrease and are never reused.
- **Recovery:** `counters.json` is a cache; the repo is ground truth. If
  missing/corrupt, rebuild by scanning existing session IDs **and** every
  session ID mentioned in `evidence_refs` and `log.md` (a user may have deleted
  a folder; its ID must still never be reused), setting each `(prefix, date)` to
  the max `NNN` found. Run this automatically before any allocation if the file
  fails to parse; note it in `log.md`.

### 4.5 `summary.md` and `highlights.md` — the running artifacts

Because sessions have no end, there is no "write the description at session end"
moment (v1's rule is dead). Instead:

- **`highlights.md`** — appended at the agent's discretion at natural checkpoints
  (a topic completing, an emotional beat, roughly every 8–15 turns). Each bullet
  carries its turn range: `- [t8–t14] Paper rejection came up; relief more than
  sadness.` These are the local checkpoints that survive a dropped terminal.
- **`summary.md`** — a running narrative of the session so far, refreshed at the
  same checkpoints. Ends with a coverage marker: `<!-- through t42 -->`. Hard
  rule: the summary must never lag the transcript by more than 15 turns.
- **`meta.yml: description`** — one sentence, kept current alongside the summary.

These are agent-written and *not* user-confirmed — see §5.3 for what they may and
may not be used for.

### 4.6 `citations.md` — the memory-consult log

Whenever the session's skill opens a memory page, system page, or old session to
personalise the conversation, it logs one line:
`- t12: memory/people/dadi.md — checked current health status`.
Two consumers: resumption (§4.7 — the agent re-knows what it had looked up) and
consolidation (which learns what context the conversation was already grounded
in). Session-owned, append-only.

### 4.7 Resuming a session

From the home screen ("continue"), the user picks a recent session (listed from
`meta.yml` + `description`). Two paths:

1. **Native fast path (optimization).** If `harness_session_id` is set and that
   harness session still exists on this machine, resume it natively — full
   context, zero rebuild.
2. **File rebuild (the guaranteed path).** Load `summary.md` + `highlights.md` +
   the last ~20 turns verbatim from `turns.jsonl` + every file listed in
   `citations.md`. This must always work; correctness never depends on path 1.

New turns append with the next unused `t`. Both paths converge on identical
on-disk behaviour.

### 4.8 Consolidation eligibility and the watermark

Two fields, two owners:

- **`meta.yml: consolidation`** (owner: the session skill).
  - `journal`, `guided`, `share_event`, `onboarding`, `review` sessions are
    created `eligible` — capturing memory is their point.
  - `conversation` sessions are created `not_eligible`. The flag flips to
    `eligible` only by user request ("remember this one") or by the agent
    *asking* — "this felt like it touched something real; want me to keep it for
    reflection?" — and the user agreeing. Never silently. The flip may also
    happen later, from a *different* session ("remember Tuesday's chat"): the
    acting skill touches exactly that one field, citing the user's consenting
    turn — the §4.9 exception.
  - `safety_held` is terminal until the safety path releases it (§13).
- **`consolidation_state.yml`** (owner: consolidation, §4.9):
  `consolidated_through_turn` plus `entry_consolidated: true|false`. Created by
  the first consolidation run that includes the session.

**Selection rule (replaces v1's `processing_state`):** a session needs
consolidation iff `consolidation: eligible` **and** (`turns >
consolidated_through_turn`, **or** `entry_status: confirmed` while
`entry_consolidated` is absent/false). Missing state file = watermark 0. A
reopened session that gathers new turns past its watermark automatically becomes
eligible again for exactly the delta; a confirmed entry is read by exactly one
run, which sets `entry_consolidated: true` alongside the watermark bump. This is
computable purely from repo state — no session memory, fully idempotent. (The
entry clause is also what makes migrated v1 journal entries — sessions with a
confirmed `entry.md` and zero turns — selectable; §15.)

### 4.9 The single-writer rule

Concurrent writers must never touch the same file. Two levels:

**Inside a session folder:** the session's own skill owns `turns.jsonl`,
`meta.yml`, `summary.md`, `highlights.md`, `citations.md`, `attachments/`;
consolidation owns `consolidation_state.yml` and nothing else. The watermark
bump rides the PR branch (§9.2 step 4) and can never collide with live-session
writes.

**At the repo root:** `index.md`, `log.md`, `temporal_context.md`,
`working_memory/rejected/`, and `.margin/` state files are **main-only** — no
insight branch ever carries a change to them (branch scope, §4.13).
Consolidation and review write them directly on main via the primary working
tree. This is what keeps a proposal mergeable after sitting open for days while
sessions continue on main.

**The one narrow exception to single-writer:** flipping another session's
`meta.yml: consolidation` field — "remember Tuesday's chat" (§4.8), or releasing
`safety_held` (§13) — may be done by whatever skill the user is currently
talking to, on the user's explicit recorded consent, citing the consenting turn,
touching that one field and nothing else. It is the only cross-writer touch in
the system.

### 4.10 `entry.md` — the journal-equivalent entry

Journaling-kind sessions (`journal`, `guided`, `share_event`) produce a coherent
entry as their product:

- For `/journal` free-writes, `entry.md` is the user's own text, verbatim, and is
  born `confirmed`; `confirmed_at_turn` = the turn that delivered the final text.
- For `/guided` and `/share_event`, the agent composes it at the natural
  wind-down — **from `highlights.md` + `summary.md`**, verifying against the
  transcript only where unsure (never a full re-read of a long session) — then
  shows it to the user. It must stay strictly faithful to what the user said and
  felt; the user's voice, not the agent's. On explicit user approval it becomes
  `confirmed`; the approving turn number is recorded.

```yaml
---
type: journal_entry
entry_kind: daily          # daily | life_share | event | free
title: ""
description: ""
tags: []
timestamp: 2026-07-13
effective_date: null       # when the content HAPPENED, if about the past — a date
                           # or approximate period ("2016–2018", "childhood").
                           # Distinct from timestamp (when written). life_share and
                           # event entries set it; a 2016 memory filed in 2026 must
                           # be findable as a 2016 memory.
status: confirmed          # draft | confirmed
confirmed_at_turn: 44      # null while draft
---
<!-- p1 -->
First paragraph… (permanent paragraph anchors, exactly as v1: assigned in order,
never renumbered; appended paragraphs take the next unused number.)
```

Abandonment is normal: if the user vanishes before confirming, `entry.md` stays
`draft` (or absent). On resume the agent offers it; consolidation treats a
non-confirmed entry as nonexistent and falls back to the transcript (§5.3).

A `confirmed` entry is **closed**: nothing is appended to it. Later material
from a reopened session lives in the new turns (and, if it deserves an entry, a
new session's entry) — the "next unused paragraph number" rule exists for drafts
being revised before confirmation, not for growing confirmed entries.

### 4.11 `attachments/` — files shared in a session

When the user shares a file (a path, an image, an existing draft), the agent
**copies it into** `sessions/<id>/attachments/` as `att_<n>_<original_name>`
(stable id `att_<n>`, allocated per session from 1). The sharing turn records the
id, and the agent writes a one-line content description into that turn's `text`
or an adjacent `system_note` — so future consolidation knows what `att_2` was
without opening it, but can open it. Attachments are citable evidence
(`sessions/<id>/attachments/att_2_photo.jpg`). Originals outside the repo are
never relied upon after copy-in. (Large-binary/LFS handling: documented post-POC
seam, §17.)

### 4.12 Commit discipline for sessions

Writes to the working tree are immediate (crash safety); **commits are batched at
checkpoints** (§4.5) and on skill hand-offs, message
`chore(session): s_2026_07_13_001 through t42`. Session commits happen **on main
only** — the skill verifies `HEAD == main` before committing (§4.13).

Every Margin invocation begins with the **tidy-up rule**: commit any dirty
*session* files left by a previous run — after refreshing that session's
`meta.yml` counters and validating the `turns.jsonl` tail (§4.2) — then run
crash recovery (§4.13). Dirty files under `memory/` or `system/` are never
tidy-committed (Invariant 3); they take the user-edit path (§4.13). The index
refresh rides whichever main-side commit triggered it (§8.2).

### 4.13 Git mechanics (worktrees, sync, locking, recovery)

The mechanics that keep "the user never sees Git" true under concurrency and
crashes. Binding.

**Two working trees, fixed roles.** The repo's primary working tree stays
checked out on the default branch (`main`) at all times — sessions, generated
artifacts, and tidy-up commits all happen here, and session skills verify
`HEAD == main` before every commit (if it isn't, stop and surface it; never
commit source anywhere else). Insight branches are **never checked out in the
primary tree**: consolidation creates them in a dedicated `git worktree` at an
OS-appropriate scratch location (path recorded in `.margin/open_proposal.yml`);
all branch-side edits — consolidation's and review's — happen in that worktree.
It is removed (`git worktree remove` + prune) when the proposal resolves.

**Branch scope.** An insight branch may contain changes ONLY under: `memory/`,
`system/`, `working_memory/candidates/`, `working_memory/pending_confirmations/`,
`sessions/*/consolidation_state.yml`, and `.margin/proposal.yml` (the manifest,
§9.2) — plus `.margin/proposal_letter.md` in offline mode (§10.1). Everything
else — `index.md`, `log.md`, `temporal_context.md`, `working_memory/rejected/`,
`.margin/` state, all other session files — is main-only (§4.9).

**Sync protocol (when a remote exists).**
- After each main-side commit batch: `git push`, best-effort. A failed push
  (offline) is tolerated and retried at the next invocation — local state is
  always the working truth.
- Before `gh pr create`: push `main`, then the branch.
- Merging (review only): push the branch; **verify the remote branch head equals
  the local one**; `gh pr merge --merge`; then immediately `git pull --ff-only`
  on main. Only after the pull do the post-merge main-side commits happen (index
  refresh §8.2, `log.md` row, `.margin/open_proposal.yml` removal, worktree
  cleanup). Any failure at any step → stop *before* the merge, tell the user
  plainly, retry next invocation. Nothing is lost, because nothing merged.

**Locking.** One advisory lockfile, `.margin/lock` (untracked), held during
(a) ID allocation (§4.4) and (b) the whole consolidate and review pipelines. A
second invocation finding it waits briefly, then tells the user who holds it. A
stale lock (no live process, older than ~10 minutes) may be broken, noted in
`log.md`.

**Crash recovery — runs at every invocation, right after tidy-up (§4.12):**
- `git worktree prune`. Delete any `margin/insight/*` branch that has neither an
  `.margin/open_proposal.yml` entry on main nor an open PR — it is a crashed,
  never-proposed run; its watermarks never landed, so nothing is lost. Note it
  in `log.md`.
- If `.margin/open_proposal.yml` exists but its branch (or `pr:` reference) is
  gone, remove the marker and log the inconsistency.
- Dirty (uncommitted) changes under `memory/` or `system/` in the primary tree —
  e.g. the user hand-edited a page in Obsidian, an advertised workflow — are
  never committed to main and never discarded: move them onto a
  `margin/user-edit/<date>` branch as a **user-authored proposal** the review
  flow renders ("you edited these pages by hand — keep it?"). The user wrote it,
  so the review gate is satisfiable; hand-edits enter memory through the same
  single door as everything else.

---

## 5. Evidence rules

### 5.1 Anchors and refs

Stable anchors, citable from memory pages:

- Turn: `sessions/s_2026_07_13_001/turns.jsonl#t14`
- Entry paragraph: `sessions/s_2026_07_13_001/entry.md#p3`
- Attachment: `sessions/s_2026_07_13_001/attachments/att_1_draft.md`

Anchor stability is what makes memory auditable; treat renumbering as data
corruption.

### 5.2 Who counts as evidence (Invariant 9, operationalised)

Evidence about the user is: **the user's own turns**, **confirmed `entry.md`
paragraphs**, and **attachments the user shared**. The assistant's turns are never
evidence about the user — citing them would launder the model's own speculation
into fact. Corollary: when the agent offers a reflection and the user's next turn
confirms it ("yeah, that's exactly it"), the *user's confirming turn* is citable —
cite the pair (assistant turn for context, user turn as the evidence).

### 5.3 The three artifact classes (what may be cited)

| Class | Examples | Citable? |
|---|---|---|
| **Ground truth** | user turns, confirmed `entry.md`, attachments | Yes — this is what `evidence_refs` point at |
| **User-confirmed derivation** | `entry.md` after confirmation | Yes — first-class, equal to hand-written text |
| **Agent-written, unconfirmed** | `summary.md`, `highlights.md`, `meta.yml: description`, draft `entry.md` | **Cite-through only.** Usable for navigation and session-level pointers ("came up in s_…"), but any claim-level evidence must resolve to the turn refs those bullets carry. A summary is never itself proof of anything. |

The cite-through rule exists to block a specific failure: the model summarises
with a slight spin, then later cites its own spin as if the user said it.

---

## 6. The memory layer (the wiki)

### 6.1 Kinds and page contracts

Seven kinds, seven one-level folders under `memory/`, **zero pre-created pages**.
Each kind has a one-line contract for what "one page" means — these live verbatim
in `AGENTS.md` and bind the update-first procedure:

| Kind | Contract |
|---|---|
| `people/` | One page per human. Never two pages for the same person. |
| `places/` | One page per place that matters. |
| `events/` | One page per discrete event or milestone. |
| `threads/` | One page per *named pattern* that cuts across life areas. Before creating, ask: new pattern, or new evidence for an existing one? |
| `domains/` | One page per broad life area (work, family, health…). A life has roughly ten; an eleventh should feel suspicious. |
| `life_periods/` | One page per **closed** chapter. Pages tile the timeline; no overlaps. The current chapter lives in `system/current_chapter.md` only. |
| `identity/` | Few and slow. Values, self-concept, orientation-of-life pages. Creation is rare and always confirmation-gated (§9.4). |

Folder = broad kind. All nuance lives in frontmatter (`type`, `tags`) — a tag
list can say "pattern *and* struggle *and* initiation"; a folder path forces one
possibly-wrong answer. Deep taxonomies are v1 legacy; do not recreate them.

### 6.2 Page anatomy

```yaml
---
id: thread_start_vs_continue        # stable forever, independent of display name
type: thread                        # = the kind; OKF-required
title: "Start vs. continue"
aliases: [start vs continue, initiation friction, cold-start problem]
description: "Hard to start things cold, easy to continue once in motion."
tags: [pattern, initiation, motivation]
status: active                      # active | archived
confidence: high
user_confirmed: true
timestamp: 2026-07-13
evidence_refs:
  - sessions/s_2026_05_18_001/turns.jsonl#t18
  - sessions/s_2026_05_18_002/turns.jsonl#t2
---
Body: the living synthesis, in the user's framing. **Related memory pages are
linked inline** — [gym and body image](body_image.md), [calling his
mother](../people/mom.md) — the link web IS the memory graph and a page with
zero links to related pages is a lint finding (§12).

## Open questions
(what would raise confidence; genuinely open, not rhetorical)

## Evidence
(the refs above, as readable links)

## Change log
- 2026-07-13: Created from s_2026_07_12_001. (PR #4)
```

Requirements carried from v1, still binding: an honest one-sentence `description`
(the read side depends on it; stale after an update = defect); append-only
`evidence_refs`; dated change-log lines; supersede by archiving, never deleting.

### 6.3 Page lifecycle: the update-first decision procedure (Invariant 10)

For every distinct piece of new information, consolidation runs this fixed
sequence — it is never an open creative question:

1. **Search `index.md` first** — descriptions and `aliases` — for any existing
   page that *claims this subject*. Aliases exist precisely so "Dadi" and "my
   grandmother" resolve to one page.
2. **If a page claims the subject → `update`. Always.** Growing an existing page
   is the default action. New information about a known person goes on their
   page, even if it's a new facet.
3. **`create` only if no page claims the subject**, and the action envelope's
   `reason` (§9.5) must contain a written justification: "no existing page covers
   X because…". Creations are headline items in the PR (§10.2) — the structure
   cannot grow without the user watching it grow.
4. **Merging** (when lint or consolidation finds two pages that turned out to be
   one subject): a new combined page is proposed via PR; both originals move to
   `archive/` on the same branch. Their stable `id`s are recorded in the merged
   page's frontmatter (`merged_from:`) so old evidence trails stay followable.
5. **Archival:** a page whose subject is genuinely closed (a life period ends, a
   thread resolves) gets `status: archived` and moves to `archive/` via PR. Never
   deleted.
6. **Interlinking duty:** whenever a page is written or updated, link it to the
   related pages it mentions, and add back-links where clearly warranted. Orphan
   pages (no in- or out-links) are lint findings.

### 6.4 `system/` — the pinned layer

Five pages, seeded at onboarding (§14), loaded every session, kept tiny. They are
PR-gated like `memory/` (they are the most identity-dense pages of all). Budget:
if `system/` collectively exceeds ~200 lines, consolidation should be moving
detail out to memory pages and linking instead.

`current_chapter.md` lifecycle: it describes *now*. When consolidation concludes
(with user confirmation — this is identity-touching by definition) that a chapter
has closed, the same PR writes the closing synthesis as a new
`memory/life_periods/<name>.md` and resets `system/current_chapter.md` for what's
beginning. One page's death is the other's birth; they never coexist as two
"current" descriptions.

---

## 7. Temporal context (the recency layer)

`working_memory/temporal_context.md` — generated, auto-committed. The layer v1
lacked: a fresh session knew who you *are* but not your *week*.

**The design test for what belongs in it (binding):** *what would a good friend
who last saw you a week ago need, to pick the conversation back up naturally?*

### 7.1 Frontmatter (staleness is computable)

```yaml
---
type: temporal_context
built_at: 2026-07-13
covers_through: 2026-07-11      # creation date of the newest source folded in
last_included: s_2026_07_11_002 # newest session/entry included
---
```

### 7.2 Structure: three bands with different semantics

- **Now (last ~7 days)** — dated, detailed highlights per session/entry. Calendar
  time is genuine here: "what happened this week" is a real question.
- **Active strands** — the heart of the file. One bullet per *ongoing situation*,
  updated **in place**, always stating current state, never accumulating history:
  `- Paper resubmission — advisor feedback received, revising. (last touched Jul 9)`.
  A new session that mentions a strand refreshes its bullet and its date.
- **Open loops** — unresolved commitments and questions ("was going to call
  Dadi"). Age-agnostic; removed only when closed.

**Decay by demotion, not cliff:** at each rebuild, Now-items age into strand
updates; a strand whose `last touched` passes ~3–4 weeks and isn't an open loop
drops off — safe by construction, because anything alive that long has been
through consolidation into long-term memory. Re-mention re-promotes. Hard size
budget (`config.yml`, default 120 lines): if over, stalest strands compress or
drop first.

Temporal context is **not pruned when consolidation absorbs its contents** — it
is a recency cache, long-term memory is a synthesis; they overlap by design and
owe each other nothing.

### 7.3 Refresh: lazy, with a cheap gap-filler

At session start the agent compares repo state to `covers_through`:

- **Gap small** (≤ `staleness_days` *and* ≤ `staleness_sources` newer sources):
  do **not** rebuild. Load the file as-is **plus the `summary.md` of each newer
  session** — the running summaries make freshness free.
- **Gap large:** rebuild **incrementally** — previous temporal file + the new
  sessions' summaries/highlights in, new version out. Never a from-scratch scan
  of raw turns.
- Additionally, **every consolidation run refreshes it at the end for free** (it
  already read all the new material) — committed on main, never on the branch
  (§4.13).
- The gap is measured against sessions' **`updated_at`**, not just creation
  dates — a months-old session reopened yesterday counts as new material.
- If the file is missing entirely (deleted by hand), a one-time from-scratch
  seed is permitted: build it from the `summary.md` and recent highlights of the
  last ~10 sessions, and note the rebuild in `log.md`.

The file is user-readable and user-editable; rebuilds read the previous version,
so user edits flow forward naturally. (Mostly it's the agent's to maintain.)

---

## 8. The read side

### 8.1 Four load tiers

- **Pinned.** `system/` — every session, always.
- **Temporal.** `temporal_context.md` (+ gap-filler summaries, §7.3) — every
  session start for conversational skills.
- **Indexed.** Every memory page and session is *discoverable* via `index.md`
  (path + description + confidence) but loaded only deliberately.
- **Cold.** Everything else. Not loaded, not scanned.

Hard rules carried from v1: **up carries judgment, down carries evidence** (never
present an inference you couldn't drill to source for); **never blind-load** a
directory — if the index doesn't point at a file, it isn't relevant to this
question. Every deliberate open during a session is logged in `citations.md`
(§4.6).

### 8.2 `index.md` (generated, never hand-edited)

Sections: System · Memory pages (grouped by kind) · Working memory · Pending
consolidation (all sessions needing it, §4.8, regardless of age) · Recent
sessions (rolling `source_window_days` window). Entries in OKF shape —
`* [Title](path) - description` — with confidence/confirmed appended to the
description sentence for memory pages, and skill/eligibility for sessions.

Regeneration: on main, bundled **into the commit that triggered the change** (a
session checkpoint, a tidy-up), never its own commit; if byte-identical, nothing
staged. Two sanctioned exceptions, each a standalone main commit: immediately
after a proposal merges (`chore: refresh index after <proposal>` — a server-side
merge commit cannot carry a regeneration; §4.13) and when `margin-doctor` finds
drift. The index is **never committed on an insight branch** (§4.13). Older
sessions fall out of the window but remain reachable by path and via
`evidence_refs` — the index is a signpost, not an inventory.

### 8.3 Drill-down by question type (binding for conversational skills)

| Question | Start at | Drill to (only if needed) |
|---|---|---|
| Who am I / how have I changed | pinned + relevant `identity/`, `domains/`, `threads/` pages | their `evidence_refs` → source anchors |
| What's going on lately | temporal context (+ gap summaries) | the sessions it points at |
| Specific fact/date/person/event | `events/`, `people/`, the session via index | widen the date range, then related sources |
| "What do you remember about me?" | `index.md` + pinned | whatever the user asks about next |

Confidence governs register everywhere: `high` + `user_confirmed` may be stated
plainly; anything else is hedged in natural language. If the index points
nowhere, the honest answer is "I don't think I know that about you yet" — never
confabulate.

---

## 9. Consolidation

Skill: `margin-consolidate`. Turns eligible session turns into proposed memory
changes on a branch, ending in exactly one PR. Never merges itself.

### 9.1 Triggers: manual, nudged, or scheduled

- **Manual (default).** "Consolidate now" / the home-screen action.
- **Nudged.** When pending sessions ≥ `consolidation.nudge_after`, the home
  screen shows the count and the agent may ask *once* at a session's natural end
  — never mid-conversation, never twice in a day. The last-nudge date lives in
  `.margin/ui_state.yml` (generated, committed) so "once a day" holds across
  invocations.
- **Scheduled (opt-in, Mode B).** A real cron entry, recorded in `config.yml` and
  created on the user's machine with their informed consent (the agent states
  exactly what it changed, that unattended runs consume quota, and how to turn it
  off). A scheduled run follows the identical path ending in a PR — the schedule
  decides *when a proposal is prepared*, never *whether memory changes* (§16).
  Finding nothing pending, it exits silently; no empty PRs.

### 9.2 The pipeline (per run)

1. **Lock, worktree, workspace.** Take the lock (§4.13). Create the insight
   branch in its dedicated worktree (§4.13). Create `.margin/work/<run-id>/`
   (gitignored): the list of selected sessions (≤ cap, oldest first) with their
   deltas (`watermark+1 … turns`, plus any unconsolidated confirmed entry), a
   snapshot of the current `memory/` tree, an empty `checklist.md` and
   `update_log.md`.
2. **Scan → checklist.** Read the delta turns (+ confirmed entries +
   attachments) of every selected session. Write `checklist.md`: one item per
   distinct piece of new information, each with its evidence anchors and a
   proposed tier (§9.4).
3. **Enrichment loop.** Item by item: run the update-first procedure (§6.3) —
   search index/aliases → pull the *target pages only* (read-for-writing; no tree
   wandering) → apply the edit on the branch **as that item's own commit** (the
   manifest references it; reverting an item = reverting its commit,
   newest-first when several share a page) → append one line to
   `update_log.md` → tick the item. Every page edit follows §6.2 discipline
   (evidence appended, description refreshed, change-log line, links added).
4. **Watermarks ride the branch.** Write each selected session's
   `consolidation_state.yml` on the branch: `consolidated_through_turn = <last
   turn read>`, and `entry_consolidated: true` if its confirmed entry was read
   (§4.9). Merge lands memory changes + watermarks atomically; close/reject
   discards both, so sessions correctly remain pending.
5. **The manifest.** Write `.margin/proposal.yml` on the branch: run metadata
   plus one record per numbered item — `{n, action, target_path, evidence_refs,
   summary, requires_user_confirmation, commit, verdict: pending}` — where
   `commit` is the item's own branch commit (step 3), the handle review uses to
   revert it. The letter is the
   human surface; the manifest is the machine contract review depends on
   (surgical per-item removal, rejection signatures, resumable sittings). It is
   generated from the envelopes, never re-derived, and must agree with the
   letter item-for-item.
6. **One PR** (§10), body composed from `update_log.md` + the envelopes, pushed
   and opened per the sync protocol (§4.13). Then, **on main**: write
   `.margin/open_proposal.yml` (branch, date, PR number or `local`, item count,
   worktree path, and the letter's title for the home screen), append the
   `log.md` row (`proposed`), and refresh
   `temporal_context.md` (free — you just read everything). Delete the scratch
   workspace (the manifest, not the workspace, is what review needs); keep the
   worktree until the proposal resolves. Release the lock.

### 9.3 Idempotency

Selection is watermark-based (§4.8), so re-runs skip consolidated turns
automatically. Before any `create`, the §6.3 search makes an equivalent existing
page an `update`. A fresh agent session resumes all of this from repo state alone.

### 9.4 Promotion tiers (carried from v1, binding)

| Tier | Signal | Destination |
|---|---|---|
| **High** | Explicit user statements: preferences, boundaries, named relationships, stated facts | Memory page (via PR) at `confidence: high` |
| **Medium** | Pattern across ≥2 sources over time | Memory page at `confidence: medium`, framed as tentative noticing |
| **Low** | Single-source interpretation, mood inference, agent hypothesis | `working_memory/candidates/` only. May appear in a PR as "a hunch I'm holding lightly"; must not touch identity |
| **Confirmation required** | Anything touching `identity/`, `life_periods/`, `system/`, sensitivities, or anything that could read as health/psych profiling | `working_memory/pending_confirmations/`; promoted only on explicit user "yes" |

A vivid life-share may seed a high-level page in one go, but with `## Open
questions`, non-`high` confidence, and `user_confirmed: false` — narrative ≠
verified fact. **`user_confirmed` is never `true` on a page's first PR** unless
the user explicitly affirmed that exact item during review (§10.3), in which case
the affirming turn is cited.

### 9.5 The action envelope (unchanged from v1 in shape)

```json
{
  "action": "create | update | merge | archive | defer | ask_user | no_change",
  "target_path": "memory/threads/start_vs_continue.md",
  "summary": "…",
  "evidence_refs": ["sessions/s_2026_07_12_001/turns.jsonl#t9"],
  "confidence": "low | medium | high",
  "requires_user_confirmation": false,
  "reason": "Plain language; MANDATORY and specific for create/merge/archive."
}
```

`requires_user_confirmation` is forced `true` for any target under
`memory/identity/`, `memory/life_periods/`, `system/`, or safety-adjacent
content.

### 9.6 Anti-nag (carried from v1, hardened)

Rejected proposals are recorded in `working_memory/rejected/` with
`evidence_signature = sha1(target_path + "\n" + "\n".join(sorted(evidence_refs)))`.
Never re-propose a claim with an already-rejected signature; revisit only when
new evidence changes the signature, and then acknowledge the earlier "no" in the
PR.

Two additions that make this hold under real conditions: rejection records are
written by review **directly to main at the moment of rejection** (§4.13 branch
scope) — they must survive a wholly-discarded branch. And any new proposal
targeting a `target_path` that carries *any* prior rejection record must
explicitly acknowledge the earlier "no" in the letter, even when new evidence
legitimately changes the signature — the signature is the exact-duplicate
tripwire, not a license to re-pitch.

### 9.7 Caps and hygiene

- **Catch-up cap:** ≤ `max_sessions_per_run` (default 12) sessions per run,
  oldest first; if more remain, say so plainly and offer another pass.
- **One open proposal at a time.** The gate is the presence of
  `.margin/open_proposal.yml` on main (§9.2 step 6) — repo state, so it works
  offline; PR status via `gh` is a cross-check, never the source of truth. A
  second consolidation while one is open is refused warmly ("I'm still waiting
  to hear what you think about the last thing — let's settle that first").

---

## 10. The proposal and `/review` (the heart of the UX)

### 10.1 Mechanics

One proposal per run. Branch: `margin/insight/<YYYY-MM-DD>-<slug>`, built in its
worktree (§4.13), opened as a PR via the user's authenticated `gh` after pushing
per the sync protocol. `.margin/open_proposal.yml` on main is the canonical "a
proposal is open" marker (§9.7); PR state is a cross-check, never the source of
truth.

**Offline fallback (fully specified):** if GitHub is unreachable, the run
commits the letter itself to the branch as `.margin/proposal_letter.md`
(alongside the manifest) and sets `pr: local` in the marker. Review renders that
file instead of the PR body; "merge" = the agent merges the branch into main
locally after the sitting (the user approves in the CLI exactly as always),
deletes the branch, removes the marker; `log.md` rows cite the branch name where
a PR number would go. When connectivity returns, the next invocation simply
pushes main — nothing to reconcile, because the marker, not GitHub, was always
the gate. The *contract* (a reviewable, titled, evidence-linked, per-item
proposal) is what matters, not GitHub.

### 10.2 The PR is a letter, ordered by attention

Title: a human sentence ("I think I'm noticing something about how you rest"),
never a filename. Body sections, **in this order**:

1. **"Please look closely"** — items the agent is least sure about, anything
   identity-touching, and every **new page creation** (with its §6.3
   justification), each framed honestly: "I might be wrong here."
2. **"Quick confirmations"** — high-confidence restatements of things the user
   plainly said, skimmable.

Every item is **numbered** and self-contained: what was noticed, in plain
companion voice; the evidence, linked to anchors so the user can re-read their
own words; how confident and what would raise it; a pressure-free invitation to
say no. Never imply the user should approve. The PR body is user-facing product
copy — the CLI renders it directly, and a future GUI will render the same body.

### 10.3 The `/review` flow (primary skill)

Consolidation *produces*; review *consumes*. Review is invoked from the home
screen whenever an insight PR is open (or chained immediately after a manual
consolidation when the user is present). It:

1. Renders the letter in the terminal, item by item.
2. Collects per-item verdicts — "1 yes · 2 no · 3 yes, but it was my aunt" —
   recording each in the branch's `.margin/proposal.yml` manifest with the
   verdict turn. Five verdicts exist: **yes** (explicit affirmation), **proceed**
   (consent to keep the change without affirming it), **correction**, **no**,
   and **later** (withdraw the item from this proposal *without* an anti-nag
   record — it may legitimately return when future evidence adds to it).
3. Applies each outcome **on the branch** (in the worktree): affirmed items may
   set `user_confirmed: true` citing the affirming review turn; corrected items
   are revised (the correction is the user's own words — first-class evidence);
   rejected and deferred items are reverted via their manifest-recorded commits
   (§9.2 step 5), and rejections are recorded per §9.6 **directly on main**.
4. Partial sittings are normal: the manifest records which items have verdicts;
   a later sitting resumes from the pending ones. All items resolved → merge per
   the sync protocol (§4.13), then on main: remove `.margin/open_proposal.yml`,
   update `log.md`, refresh the index (§8.2), clean up the worktree. Closing
   everything = reject the run; sources stay pending except explicitly rejected
   claims (whose records already live on main).

The user's self-report always outranks inference. Never argue.

### 10.4 Review sessions are sessions

A review sitting is recorded as a session folder (`skill: review`, `eligible` by
default) — its transcript is where confirmations and corrections live as citable
turns. This keeps the write path single and auditable: even the act of reviewing
memory produces evidence through the same door as everything else.

One guard on its later consolidation: **verdict turns are not independently
promotable.** The user's "1 yes / 2 no" answers had their entire effect during
the sitting (confirmations applied, rejections recorded); consolidation must
never mine them as fresh evidence for the same claims — that would launder a
review verdict into corroboration, and would let a rejected claim return citing
the very conversation where it was declined. What *is* candidate evidence from a
review session: corrections in the user's own words, and anything new they
spontaneously shared along the way.

---

## 11. The home screen (`margin` bare)

A pure render of repo artifacts — temporal context, `index.md`, `log.md`, PR
state, pending-consolidation counts. It computes nothing new. Three zones:

1. **Greeting** — name, time of day, and **one** temporal hook (a question or
   callback from `temporal_context.md`; rotate, never stack — the last hook used
   is recorded in `.margin/ui_state.yml`). **The hook is an offer, not a
   reminder:** it must read as a companion's curiosity or care and end with a
   way in. ✅ "The blog's been sitting there ready — want to talk through what
   the first post could be?" ✗ "The blog site is built and waiting for its
   first post — just flagging it." A hook that reads as a status flag, task
   nudge, or progress report is a voice defect.
2. **Status strip** — only lines that are true: an open proposal (rendered from
   `.margin/open_proposal.yml`: title + item count), pending-consolidation count
   when ≥ nudge threshold, resumable drafts. No empty-state noise.
3. **Actions** — `chat · journal · guided · share · continue · review ·
   consolidate`, one line each, plain words.

Layout is stable across runs (it's a home screen, not a log). Skills are equally
invocable directly (`margin chat`, `margin journal …`) without passing through it.

---

## 12. Skills inventory

**Primary (built in v2):**

| Skill | Role | Sessions it creates |
|---|---|---|
| `margin-conversation` | open companion chat; the default voice | `conversation` (not_eligible by default) |
| `margin-journal` | free-write or point at a draft/file | `journal` (eligible; entry.md born confirmed) |
| `margin-guided` | agent-led daily reflection | `guided` (eligible; entry.md composed + confirmed) |
| `margin-share-event` | past-recall: a memory, event, or part of life | `share_event` (eligible; entry.md composed + confirmed) |
| `margin-consolidate` | §9 pipeline → one PR | none (works on a branch) |
| `margin-review` | §10.3 in-CLI review & merge | `review` (eligible) |
| `margin-doctor` | lint: broken/orphan links, stale descriptions, contradiction flags, watermark/counter sanity, summary lag | none (report; fixes via PR where derived) |

**Shared rubric (not a skill):** the companion-voice rules v1's recall skill
carried — derived-first for identity questions, confidence governs register,
speculation labelled as the agent's own, narrow trigger (general questions get
normal answers with zero memory traversal), never confabulate — bind **every**
conversational skill above. They live in a shared rubric file the skills
reference.

**Onboarding** is a script + interview (§14), not a skill — it runs once.

**Growing skills (post-POC, explicitly not now):** weekly/monthly summaries,
pattern-insight reports, proactive nudging beyond the home-screen line,
off-the-record chat (a `conversation` variant that writes no files), data
export/erasure flows.

---

## 13. Safety path (overrides everything)

If a session surfaces acute risk (self-harm, harm to others, crisis, abuse): the
agent (1) stays in the conversation with warmth — support first; (2) surfaces
resources only if `safety_resource_surfacing: true`; (3) preserves the session
verbatim like any other (it is the user's words; never deleted) and sets
`meta.yml: consolidation: safety_held`; (4) produces no highlights, no candidate
extraction, no insight PR from that session. `safety_held` lifts only if the user
later revisits the topic in a clearly non-acute way and consents to reflection —
mechanically: the skill they are then talking to flips that session's flag,
citing the consenting turn (the §4.9 exception). No clinical labels, ever
(Invariant 8).

---

## 14. Onboarding (one-time bootstrap)

This section is the contract; the step-by-step builder procedure — including
the full definition of the onboarding interview — lives in `SCAFFOLD_SPEC.md`.
The user types zero Git and sees zero Git concepts, ever. Steps:

1. **Pre-flight.** Git + authenticated `gh`; create the **private** repo.
   Onboarded-ness is gated on **interview completion**, not scaffold existence:
   if a completed onboarding session exists (and `config.yml` carries a
   timezone), abort to normal operation; if the scaffold exists but onboarding
   never finished (a crashed bootstrap), **resume the interview** instead of
   wedging.
2. **Scaffold** the §3.1 tree: `AGENTS.md` (+ `CLAUDE.md` link) from template,
   empty `index.md`/`log.md`, `.margin/` metadata, `.gitignore` (`.margin/work/`
   and `.margin/lock`),
   `.gitkeep`s. **No memory pages, no seeded system content yet.**
3. **Interview** — a warm conversation, not a form, recorded as the first session
   (`skill: onboarding`, `eligible`). Covers: what to call them, timezone, why
   journal, what's off-limits, preferred tone, safety-resource opt-in, and the
   v2 additions: consolidation-nudge appetite and whether they want a schedule.
4. **First consolidation, immediately, with the user present** — thorough, because
   it seeds everything at once: the five `system/` pages, any first memory pages
   the interview supports (life-share allowance, §9.4), the initial
   `temporal_context.md`, and the index. It flows straight into `/review` —
   teaching the review ritual on day one is the point. Conservative deferral to
   `working_memory/` with no PR is also a valid outcome.
5. **Initial commit** = scaffold + onboarding session; system/memory seeding
   arrives via that first PR like every other derived change.

---

## 15. Migration from v1

Binding decisions: **raw sources only; v1 derived pages are discarded** (they
were built by a weaker pipeline and would contaminate an honest test of v2's
consolidation). The v1 repo is kept untouched as an external archive/benchmark —
nothing from `identity/…/reflections/` is copied in.

Procedure, per v1 repo:

1. Scaffold a fresh v2 repo (§14 steps 1–2).
2. **Convert each v1 source file to a v2 session folder.** v1 conversations
   (`cs_*`): each `<!-- tN -->` turn becomes a `turns.jsonl` line **preserving
   its turn number**; `meta.yml` gets `skill: onboarding` (for the onboarding
   session) or `conversation` with `consolidation: eligible`; the v1
   `description` seeds `meta.yml: description` and a first `summary.md`. v1
   journal entries (`je_*`), if any: body → `entry.md` verbatim with `<!-- pN -->`
   anchors preserved and `status: confirmed` (the user wrote it);
   `skill: journal`. IDs are allocated in v2 format **using the original v1
   creation dates** (so the timeline, index window, and temporal context stay
   truthful), and `created_at` carries the v1 date; a `migrated_from:` line in
   `meta.yml` records the v1 ID so old references remain traceable.
   **Migration is idempotent:** before converting any v1 source, check whether
   its ID already appears in some session's `migrated_from` — skip if so, so a
   crashed migration is safely re-run and no statement is ever duplicated into
   fake corroboration. Migrated journal entries have zero turns; the entry
   clause of the selection rule (§4.8) is what makes them consolidatable.
3. Run v2 consolidation over the lot (respecting the per-run cap), review, merge.
4. Log the migration in `log.md`.

Anchor semantics (`tN`, `pN`) are deliberately identical across versions, so
evidence references survive mechanically.

---

## 16. Why human-reviewed (carried forward, still load-bearing)

Recorded so nobody "fixes" it later. Fully-automatic memory is right for
productivity agents and wrong for an intimate companion: (1) this data is
identity, not telemetry — a silently wrong belief about who someone is is a harm,
not a wasted token; (2) LLM consolidation is run-to-run inconsistent — automatic
regeneration makes a person's sense of being understood flicker; the human merge
pins it; (3) silent rewrite is the exact failure Invariant 5 exists to prevent.

The bright line is **proposal vs. merge**: any trigger (manual, nudge, cron) may
decide when a *proposal* is prepared; only the human merge decides whether memory
*changes*. A contributor who auto-merges "to save the user a click" is
reintroducing the forbidden failure and must be pointed here. What v2 *does*
improve is review ergonomics: attention-ordered letters, numbered items, partial
acceptance in the CLI (§10) — make the gate pleasant, never remove it.

Governance constraints carried from v1 unchanged: purpose limitation (memory
exists for the user's reflection only), minimisation (sensitive detail stays in
source unless a page needs it), correction outranks inference, user-controlled
export/deletion, no hidden profiling — everything Margin believes is inspectable
Markdown plus a PR the user reviewed.

---

## 17. Versioning, POC calibration, and deferred boundaries

`spec_version.txt` = `2.0.0`. On mismatch: migrate or refuse; never mix contracts.

**POC calibration (build the simplest thing that satisfies the contract):** this
stage is a handful of users for a few weeks. Contracts here are exact and
binding; the following are documented seams, deliberately **not** built:

- **Atom layer** between source and derived (individual deduped facts). The seam
  is `evidence_refs` — never bypass it; atoms can later slot in without rewrites.
- **Index sharding** beyond the rolling window; **evidence-ref partitioning** for
  pages with hundreds of refs (append-only refs make this a later-add; never
  lossily compress refs now).
- **Attachment scale** (git-lfs for large binaries).
- **Line-range citation mirroring** for ecosystem lint tools (§3.2).
- **True multi-device / concurrent-writer support.** Single machine, single user
  assumed; the single-writer rule (§4.9) is the seam.
- **Off-the-record mode** and the growing skills (§12).

Friction remains the enemy of the feedback: if a choice trades elegance for less
setup friction, take the friction reduction.

---

## 18. Definition of done (v2 build)

1. Onboarding produces the §3.1 tree, a filed onboarding session, a
   first insight PR reviewed **in the CLI**, seeded `system/`, initial temporal
   context and index — zero Git typed or seen by the user.
2. Each primary skill produces a correct session folder: turns persisted
   immediately, checkpointed summary/highlights (≤15-turn lag), citations
   logged, batched commits, index bundled into the triggering commit.
3. A `/guided` session composes `entry.md` from highlights+summary, gets it
   confirmed, and the confirmation turn is recorded. An abandoned session leaves
   a resumable draft and loses nothing.
4. A `/conversation` session is not consolidated by default; flipping the flag
   (user-initiated or agent-asked-user-agreed) makes exactly its unconsolidated
   delta selectable. Watermarks advance only via merged PRs; a closed PR leaves
   sessions pending.
5. Consolidation runs the §9.2 pipeline: checklist, update-first procedure
   honoured (no create without justification; aliases checked), one
   attention-ordered PR, caps and one-open-PR respected, anti-nag signature
   respected across runs.
6. `/review` renders the letter, takes per-item responses including corrections,
   applies them on the branch, merges via `gh`, and the review sitting itself
   exists as an eligible session.
7. Temporal context: correct three-band structure, lazy refresh honoring the
   staleness thresholds, gap-filling from session summaries, decay by demotion,
   size budget enforced.
8. Evidence discipline holds under audit: every memory-page claim drills to a
   user turn / confirmed entry / attachment; no assistant turn is ever cited as
   evidence about the user; summaries are only ever cited through their turn refs.
9. Re-running any skill or the consolidator over already-processed state creates
   no duplicates and no re-proposals of rejected claims. A fresh agent session
   with zero context resumes everything from the repo.
10. An acute-risk test session is preserved, `safety_held`, produces no
    extraction and no PR.
11. `margin-doctor` reports broken links, orphan pages, stale descriptions, and
    summary lag on a deliberately corrupted test repo.
12. The repo opens cleanly as an Obsidian vault and renders on GitHub (links
    resolve); `index.md`/`log.md`/`AGENTS.md` conform to §3.2.
13. Migration of the v1 test repo yields sessions with preserved turn anchors
    and original dates, no imported derived pages, a successful fresh
    consolidation over them (including zero-turn entry sessions via the §4.8
    entry clause), and re-running the migration converts nothing twice.
14. Concurrency and sync hold: two simultaneous terminals cannot collide on ID
    allocation (lock); a proposal merge round-trips push → verify → merge →
    pull, leaving local main current; an orphaned insight branch and a
    hand-edited memory page are both detected and handled per §4.13 at the next
    invocation; a review sitting interrupted halfway resumes from the manifest.

When all fourteen hold, v2 is functional and ready for real use.
