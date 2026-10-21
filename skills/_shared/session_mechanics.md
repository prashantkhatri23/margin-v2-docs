# Shared rubric: session mechanics

Referenced by every session-creating skill (`margin-conversation`, `margin-journal`,
`margin-guided`, `margin-share-event`, `margin-review`). This is the operational
loop; the authoritative contracts are `MARGIN_V2_SPEC.md` §4 — on any apparent
disagreement, the spec wins.

## Opening a session

1. **Tidy-up + recovery first:** commit any dirty session files left over from
   a previous run — refreshing their `meta.yml` counters and validating the
   `turns.jsonl` tail first — then run the spec §4.13 crash recovery (spec
   §4.12). All your commits go to main; verify `HEAD == main` before every
   commit.
2. Allocate the session ID **under the lockfile** (spec §4.4/§4.13; rebuild
   `counters.json` first if it fails to parse). Create `sessions/<id>/` with
   `meta.yml` (spec §4.3), setting `skill:` and the skill's default
   `consolidation:` flag: `conversation → not_eligible`; everything else →
   `eligible`.
3. Load `index.md` as the map of available memory, plus pinned `system/`
   pages (spec §8.1). For conversational skills, also load
   `temporal_context.md` — applying the lazy-refresh check first (spec §7.3):
   small gap → load as-is plus the `summary.md` of each newer session; large
   gap → incremental rebuild, then load. The index makes deeper pages
   discoverable; it does not require loading them all.

## During the session

- **Persist every turn immediately** to `turns.jsonl` — the user's turn on
  receipt, yours once produced. Turn numbers are permanent anchors; never
  renumber (spec §4.2).
- **Checkpoints** at natural beats, roughly every 8–15 turns: append
  turn-anchored bullets to `highlights.md`, refresh `summary.md` (never let it
  lag more than 15 turns; update its `<!-- through tN -->` marker), keep
  `meta.yml: description` current, then commit the session folder
  (`chore(session): <id> through tN`). The index regeneration rides this commit
  (spec §8.2).
- **Reassess memory relevance** when the topic or its personal significance
  changes, using `_shared/companion_voice.md` and spec §8.3. Personal drafting,
  preferences, and implicit callbacks can warrant deeper retrieval.
- **Log every memory file you open** in `citations.md` with the turn that
  prompted it (spec §4.6). Navigate through the index and relevant page links;
  if they offer no grounding, do not go wandering — not knowing is an honest
  answer.
- **Attachments:** when the user shares a file, copy it to
  `attachments/att_<n>_<name>`, record the id on the sharing turn, and write a
  one-line description of its contents (spec §4.11). Never depend on the
  original path again.
- **Safety scan continuously.** Acute-risk signals trigger the safety path (spec
  §13) immediately: warmth first, resources only if opted in, set
  `consolidation: safety_held`, stop extraction (no more highlights/candidates
  from this session), keep persisting turns verbatim.

## Ending and resuming

- Sessions do not end; they trail off. Never demand a goodbye. Do a final
  checkpoint + commit whenever the interaction plausibly pauses.
- On resume (spec §4.7): try the native fast path if `harness_session_id` is
  usable on this machine; otherwise rebuild from `summary.md` + `highlights.md`
  + the last ~20 turns + every file in `citations.md`. Both paths append new
  turns with the next unused number.
- Record the current harness session id in `meta.yml: harness_session_id` when
  available.

## Hard boundaries (all session skills)

- You write **only** inside your own session folder (plus generated artifacts
  that ride your commits: `index.md`, temporal refresh when the lazy check
  demands it), and only on main. You never touch `memory/`, `system/`, other
  sessions' folders, or `consolidation_state.yml` — those belong to
  consolidation and review. The single exception (spec §4.9): on the user's
  explicit recorded consent, you may flip *another* session's
  `meta.yml: consolidation` field — "remember Tuesday's chat", releasing
  `safety_held` — citing the consenting turn, touching nothing else.
- You never flip `consolidation:` to `eligible` silently. Only the user's
  explicit agreement flips it (spec §4.8).
- Your own words in the transcript are not evidence about the user and never
  will be (spec §5.2). Write them faithfully anyway — they are context.
- If `.margin/spec_version.txt` is unreadable or mismatches the spec version
  you were built against (spec §17), stop and ask.
