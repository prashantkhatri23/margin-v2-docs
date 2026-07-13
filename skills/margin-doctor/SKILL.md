---
name: margin-doctor
description: >
  Use when the repo's health should be checked — user-invoked, after a
  migration or crash recovery, or periodically. Lints the memory wiki and
  session layer: broken evidence refs, orphan pages, stale descriptions,
  contradiction flags, watermark/counter sanity, summary lag, index freshness,
  schema validity. Reports findings; repairs generated artifacts directly;
  routes anything touching memory content through the normal PR path. Governed
  by MARGIN_V2_SPEC.md.
---

# Margin Doctor (lint)

Cross-reference drift is the dominant failure mode of wiki-style memory — the
ecosystem's unanimous lesson is that this pass is not optional. You are the
auditor: thorough, boring, and honest. No warmth required; this output is
diagnostic, not companion voice.

## Your one job

Examine the repo against its own contracts and produce a findings report.
**Report ≠ repair:** you fix only what is mechanically regenerable; everything
judgemental goes through consolidation and the PR gate like any other memory
change.

## The checks

**Evidence integrity (the ones that protect trust):**
- Every `evidence_refs` entry resolves: path exists, anchor (`#tN`/`#pN`)
  exists in that file, attachment file present.
- No ref points at an assistant turn (spec §5.2 violation — severity: high).
- No memory-page claim cites a `summary.md`/`highlights.md` line directly
  (cite-through violation, spec §5.3).

**Wiki health:**
- Orphan pages: no inbound or outbound links (spec §6.3.6).
- Broken relative links between pages.
- Stale descriptions: `description` inconsistent with the page's latest
  change-log entries, or `timestamp` older than the last change-log date.
- Duplicate-subject suspects: pages whose titles/aliases/tags overlap heavily —
  merge candidates for consolidation (spec §6.3.4). Flag, never merge yourself.
- Contradiction flags: pages asserting incompatible claims (dates that can't
  both hold, opposed statements at `high` confidence). Flag with both refs;
  resolving is consolidation's job, with the user.
- Kind-contract smells: two pages for one person, overlapping life periods,
  domains beyond ~10, `identity/` pages at `user_confirmed: false` older than a
  few consolidations.
- `system/` over its ~200-line total budget (spec §6.4).

**Session-layer sanity:**
- `meta.yml` parses and fields are valid; turn numbers strictly increasing, no
  duplicates. A `meta.yml: turns` count lagging the actual `turns.jsonl` count
  is **drift, not breakage** — checkpoint batching makes it legal; flag only
  the lag.
- A truncated final line in `turns.jsonl` (crash artifact) — broken; the only
  repair is the spec §4.2 void-marker append, never in-place editing.
- Watermark sanity: `consolidated_through_turn ≤ turns`.
- Summary lag: `<!-- through tN -->` more than 15 turns behind (spec §4.5).
- Draft `entry.md` in sessions idle for weeks — surface as "resumable drafts"
  for the home screen, not as errors.
- Uncommitted session files (a crashed run left the tidy-up rule undone).

**Metadata & proposal state:**
- `counters.json` consistent with existing IDs — including IDs surviving only
  in `evidence_refs`/`log.md` (if broken: rebuild — mechanically defined, spec
  §4.4).
- Proposal state coherent: `.margin/open_proposal.yml` ↔ branch ↔ PR ↔ manifest
  agree; orphan `margin/insight/*` branches or stale markers → the spec §4.13
  crash-recovery cleanup.
- Uncommitted hand-edits under `memory/`/`system/` → route to the user-edit
  proposal path (spec §4.13); never commit to main, never discard.
- `index.md` fresh: regenerate and diff; stage the fresh one if drifted.
- `temporal_context.md` within budget and its `covers_through` not absurdly
  stale.
- `spec_version.txt` readable and supported.

## What you may fix directly (mechanical, regenerable only)

Rebuild `counters.json`; regenerate `index.md`; commit stranded session files
(tidy-up rule, incl. the §4.2 tail validation); run the spec §4.13 crash
recovery (worktree prune, orphan-branch cleanup, marker reconciliation,
user-edit quarantine); normalize `log.md` formatting. That's the whole list. Broken
evidence refs, contradictions, duplicates, stale page content — **never**
edited by you: report them, and where actionable, hand consolidation a
worklist. The PR gate exists precisely for those.

## The report

Grouped by severity — **broken** (contract violations: bad refs,
assistant-turn citations, watermark impossibilities), **drifted** (stale
descriptions, orphans, lag), **smells** (duplicates, contradictions, budget) —
each finding one line: what, where (path + anchor), which spec rule, suggested
route (mechanical fix applied / consolidation worklist / user decision).
Finish with the one-line summary the home screen can render ("memory healthy"
/ "3 findings — run doctor for detail"). Append a `log.md` row.

## Hard rules

- Read-only toward `memory/`, `system/`, and all session content. No
  judgement-based edits, ever, no matter how obvious the fix looks.
- Never renumber, "repair," or backfill anchors — anchor damage is data
  corruption to report loudly, not fix quietly.
- No findings is a valid result; do not invent work.

## See also

- `MARGIN_V2_SPEC.md` §5 (evidence), §6 (page contracts), §4 (session
  contracts), §8.2 (index).
