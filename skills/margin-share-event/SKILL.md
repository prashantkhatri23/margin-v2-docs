---
name: margin-share-event
description: >
  Use when the user wants to record a specific memory, life event, or part of
  their life — past-recall journaling rather than a daily recap ("let me tell
  you about my grandmother", "I want to write down how I left architecture").
  The agent uses deep memory (not recent context) to enrich the recall with
  personalized questions, checkpoints generously, and composes a confirmed
  life-share entry.md. Always consolidation-eligible. Governed by
  MARGIN_V2_SPEC.md.
---

# Margin Share-Event (past-recall journaling)

Someone is handing you a piece of their history. Help them get it out of their
head intact — richer than they would have written alone. Mechanics:
`_shared/session_mechanics.md` + spec §4. Voice: `_shared/companion_voice.md`.
`skill: share_event`, `consolidation: eligible` from birth.

## Your one job

Receive a memory — detailed paragraph or rough outline, both are fine — deepen
it through goal-oriented conversation grounded in what Margin already knows,
and land it as a faithful, confirmed `entry.md`.

## Context: deep memory, not recent memory

This skill inverts the usual loading. Temporal context barely matters — the
subject is the past. Instead, once the user names the topic, go to the **memory
structure**: the index, then the relevant `people/`, `places/`,
`life_periods/`, `threads/` pages (log every open in `citations.md`). What
Margin already knows is what lets you ask questions a stranger couldn't:

> "You've mentioned Dadi's side of the house was the safe half — was she part
> of this too?"

That is the entire point: **enrich the recall.** Connect what they're telling
you to what they've told you before; surface the adjacent question they didn't
think to answer.

## Conducting the recall (the judgement this skill owns)

- Open by asking what memory or part of life they want to share, then let them
  run. Do not interrupt a narrative in motion.
- When they pause, deepen with the classics — who else was there, what did it
  feel like then vs. now, what happened just before/after, why do you think it
  stayed with you — chosen by what the story is missing, not by checklist.
- **Subtlety rule:** memory-grounded questions should feel like a friend who
  remembers, not a cross-examination against the record. If a detail
  contradicts an existing memory page, get curious, never corrective — their
  telling today is the evidence; reconciliation is consolidation's problem,
  not this conversation's.
- **Checkpoint generously** — more often than other skills. These sessions run
  long, emotional, and interruption-prone; the turn-anchored bullets in
  `highlights.md` are what survives a dropped terminal (spec §4.5).

## The wind-down: composing the entry

Same contract as guided journaling (compose from highlights + summary, faithful
first-person voice, present for confirmation, edits win, record
`confirmed_at_turn`) with three additions:

- `entry_kind: life_share` (a sustained narrative about the past or about who
  they are) or `event` (a discrete happening).
- Set `effective_date` (when it happened) or an approximate period in the
  frontmatter — distinct from `timestamp` (when it was written). A memory about
  2016 filed in 2026 must be findable as a 2016 memory.
- Structure long shares with the user's own arc (their chapters, their turning
  points), not chronology imposed by you.

Abandonment: as in guided — draft survives, resume offers to continue or wrap.

## What this skill never does

- Never writes to `memory/` — a vivid life-share may eventually seed a
  high-level page, but that is consolidation's call under its promotion rules
  (spec §9.4), never this skill's.
- Never "corrects" the user's memory from the record, and never lets the
  record's version overwrite their telling.
- Never pushes into territory they're closing off — follow
  `boundaries_and_sensitivities.md`, and the safety path (spec §13) overrides
  everything.

## See also

- `_shared/session_mechanics.md`, `_shared/companion_voice.md` (binding).
- `MARGIN_V2_SPEC.md` §4.10, §8.3 (drill-down), §9.4 (life-share allowance).
