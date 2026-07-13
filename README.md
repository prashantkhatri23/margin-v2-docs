# Margin v2 — Handoff Package

This package is what you give to a coding agent (e.g. Claude Code). It contains
everything needed to build and run Margin v2: the binding mechanics, the
judgement skills, the bootstrap procedure, and the templates.

## What's in here

```
MARGIN_V2_SPEC.md      ← THE source of truth for mechanics. Supersedes v1's spec.
SCAFFOLD_SPEC.md       ← one-time bootstrap: scaffold + the onboarding interview
                          (Path A: fresh user · Path B: migrate a v1 repo)
skills/
  _shared/
    session_mechanics.md   ← operational loop, binding on all session skills
    companion_voice.md     ← memory use, register, failure modes — binding on
                             every skill that speaks from memory
  margin-conversation/     ← companion chat (not consolidated by default)
  margin-journal/          ← user-written entries; file pointers; refinement on request
  margin-guided/           ← agent-led daily reflection → confirmed entry.md
  margin-share-event/      ← past-recall journaling, enriched from deep memory
  margin-consolidate/      ← session deltas → memory wiki, on a branch → one PR letter
  margin-review/           ← the in-CLI consent gate: per-item review, then merge
  margin-doctor/           ← lint: evidence integrity, wiki health, sanity checks
templates/
  AGENTS.md              ← schema file copied into every user repo at scaffold
  memory_page.md         ← wiki page shape
  pr_body.md             ← the letter shape
ideation-notes_*.md      ← historical v2 whiteboard notes; superseded by the spec
```

Read order for a builder: `MARGIN_V2_SPEC.md` §0–§2 → §3–§5 (containers &
evidence) → §6–§10 (wiki, temporal, consolidation, review) → `SCAFFOLD_SPEC.md`
→ the skills.

## What v2 is, relative to v1

The v1 foundations survive intact: immutable source, evidence-cited derived
memory, the candidate layer, promotion tiers, anti-nag, safety path, and — most
load-bearing of all — **the human-reviewed PR gate** (spec §16; proposal may be
automated, merge never). What changed:

- **Session-first.** Every interaction (chat, journal, guided, past-recall,
  review) is one uniform session folder with turn-by-turn persistence, no
  explicit endings, and watermark-based consolidation. v1's `journal/` /
  `conversations/` split and its three submission paths are gone.
- **Conversations count — by consent.** All sessions persist; only flagged ones
  consolidate. `/conversation` is off by default and flips only by the user's
  word.
- **The wiki replaces the taxonomy.** No pre-created pages; seven kind folders,
  pages born on demand under the update-first procedure, aliases for dedup,
  inline links between pages as the memory graph.
- **A temporal layer** (`temporal_context.md`: now / active strands / open
  loops) so a fresh session knows the user's week, not just their identity.
- **Review lives in the CLI.** The PR letter is attention-ordered and numbered;
  the user answers item by item in the terminal; the agent does all git.
- **Ecosystem-aligned.** The repo is a valid LLM-wiki/OKF bundle (`type:`
  frontmatter, `index.md`/`log.md`, root `AGENTS.md`, relative markdown links)
  — it opens as an Obsidian vault and renders on GitHub as-is.

## How to kick this off with a coding agent

1. **Critique before code.** "Read MARGIN_V2_SPEC.md and SCAFFOLD_SPEC.md.
   Before building anything, tell me where they are internally inconsistent,
   underspecified, or would break. Don't build yet." An agent that finds
   nothing is not reading carefully.
2. **Bootstrap a fresh test user** (SCAFFOLD_SPEC Path A). Play the user in the
   interview. Watch: exact tree, zero Git surfaced, first letter warm and
   reviewed in-CLI.
3. **Run the loop.** One `/journal`, one `/guided` (abandon it mid-way, then
   resume), one `/conversation` (accept the eligibility ask), then consolidate
   and review with at least one correction and one rejection. Check the spec
   §18 definition-of-done items as you go.
4. **Migrate the v1 test repo** (SCAFFOLD_SPEC Path B) and compare the fresh
   consolidation against the old v1 derived pages — the pipeline should now do
   better; the old pages are the benchmark, not an input.
5. **Adversarial pass.** Re-run consolidation twice (no duplicates, no
   re-proposals); a session that contradicts a high-confidence page (archive +
   gentle letter, not overwrite); an acute-risk entry (held, no PR); run
   `margin-doctor` on a deliberately corrupted copy.

## The design risks to keep watching

1. **Consolidation is run-to-run inconsistent.** Inherent to LLM synthesis. The
   defenses are the PR gate, evidence refs, tiers, and the candidate layer —
   keep proposals reviewable; resist auto-merge forever (spec §16).
2. **Update-first discipline is the new failure surface.** The freedom of
   born-on-demand pages is paid for by the §6.3 procedure. Watch early runs
   for fragmentation (duplicate subjects) and let `margin-doctor` findings
   drive prompt fixes.
3. **Setup friction still dominates early feedback.** If testers spend their
   goodwill on `gh auth`, you learn nothing about the memory experience — keep
   bootstrap near-one-command.
