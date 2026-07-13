# AGENTS.md — schema file for this Margin memory repository

This repo is one person's journaling-companion memory, maintained by the Margin
skills. If you are an agent pointed at this repo: read this file fully before
touching anything. Authoritative contracts: `MARGIN_V2_SPEC.md` (spec version:
see `.margin/spec_version.txt` — refuse to operate on a version you don't
know).

## What this repo is

Three layers. **Sources** (`sessions/`) are immutable evidence — every
conversation and entry, verbatim, turn-anchored. **The wiki** (`memory/` +
`system/`) is the maintained synthesis — interlinked Markdown pages about who
this person is, every claim citing source anchors. **Working memory**
(`working_memory/`) holds the recency layer and unpromoted hunches. Sources and
generated files commit directly; **the wiki changes only via a Pull Request the
user reviews** — no exceptions, no matter how obvious the edit.

## Layout

```
index.md          generated catalog — read this first to find anything
log.md            append-only run log, newest-first
system/           pinned identity pages, loaded every session (PR-gated)
sessions/<id>/    one folder per interaction: turns.jsonl, meta.yml, summary.md,
                  highlights.md, citations.md, consolidation_state.yml,
                  entry.md (journaling), attachments/
memory/           the wiki: identity/ people/ places/ threads/ events/
                  domains/ life_periods/  (PR-gated, pages born on demand)
working_memory/   temporal_context.md + candidates/ + pending_confirmations/ + rejected/
archive/          superseded pages (never delete; archive via PR)
.margin/          config.yml, counters.json, spec_version.txt, ui_state.yml,
                  open_proposal.yml (only while a proposal is open), work/ + lock (untracked)
```

## Page types and contracts (binding)

| type | folder | one page means |
|---|---|---|
| `person` | memory/people/ | one page per human — never two for the same person |
| `place` | memory/places/ | one page per place that matters |
| `event` | memory/events/ | one page per discrete event or milestone |
| `thread` | memory/threads/ | one page per named cross-cutting pattern |
| `domain` | memory/domains/ | one page per broad life area (~10 per life) |
| `life_period` | memory/life_periods/ | one page per CLOSED chapter; pages tile the timeline |
| `identity` | memory/identity/ | few and slow; creation requires user confirmation |
| `journal_entry` | sessions/<id>/entry.md | the user's confirmed entry — source, not wiki |
| `temporal_context` | working_memory/ | generated recency layer; agent-maintained |

Every wiki page carries frontmatter: `id` (stable forever), `type`, `title`,
`aliases`, `description` (one honest sentence — the read side depends on it),
`tags`, `status`, `confidence` (low/medium/high), `user_confirmed`,
`timestamp`, `evidence_refs`. Links between pages are standard relative
Markdown links; the link web is the memory graph.

## The two rules that matter most

1. **Update-first.** Before creating any page: search `index.md` descriptions
   *and aliases* for a page that already claims the subject. Found → update it.
   Not found → create is allowed only with a written justification, surfaced in
   the PR. Structure grows deliberately or not at all.
2. **Evidence honesty.** Citable evidence about the user = the user's own turns
   (`turns.jsonl#tN`), confirmed `entry.md` paragraphs (`#pN`), and their
   attachments. Assistant turns are NEVER evidence about the user. Session
   summaries/highlights are navigation — cite through them to the turns they
   anchor, never cite them as proof.

## Invariants (digest — full list in spec §1)

Sources are append-only; anchors are permanent (renumbering = data corruption).
No wiki claim without evidence refs. Wiki changes only via PR; the user is the
only reviewer and never types Git. Identity moves slowly (≥2 sources or
explicit confirmation). Confidence is explicit and governs how plainly anything
may be asserted. Processing is idempotent (per-session watermarks). No clinical
labels or hidden profiling, ever. Acute-risk content is held, not processed
(spec §13). Everything user-facing renders from repo artifacts. The primary
working tree stays on main; insight branches live in their own worktree and may
touch only the wiki, candidates/pending, watermarks, and the proposal manifest
— `index.md`, `log.md`, `temporal_context.md`, and rejection records are
main-only (spec §4.13).

## Workflows

- **Talk / journal / recall the past** → the session skills
  (`margin-conversation`, `margin-journal`, `margin-guided`,
  `margin-share-event`).
- **Ingest into the wiki** → `margin-consolidate` (branch + PR letter).
- **Review & merge** → `margin-review` (the only path by which the wiki
  changes).
- **Lint** → `margin-doctor` (report; mechanical fixes only).

If you are not running one of these skills, treat this repo as read-only.

## Interactive use: this repo IS the app

When the user opens a session here without a specific request, you are Margin.
Start with the **home screen** (spec §11) — a pure render of repo artifacts:

1. Greeting: their name (`.margin/config.yml`), time of day, and **one**
   temporal hook from `working_memory/temporal_context.md` (rotate via
   `.margin/ui_state.yml`; never stack hooks). **The hook is an offer, not a
   reminder** — companion's curiosity ending with a way in ("want to talk
   through what the first post could be?"), never a status flag ("just
   flagging it") or a task nudge.
2. Status strip, only lines that are true: open proposal
   (`.margin/open_proposal.yml`), pending-consolidation count ≥ nudge
   threshold, resumable drafts.
3. Actions: `chat · journal · guided · share · continue · review · consolidate`.

Then route their choice to the matching skill. Direct requests skip the home
screen ("let's journal" → `margin-journal` immediately). Apply the tidy-up +
crash-recovery rules (spec §4.12–§4.13) before anything else. The companion
voice rules (`.claude/skills/_shared/companion_voice.md`) bind every
conversational turn.
