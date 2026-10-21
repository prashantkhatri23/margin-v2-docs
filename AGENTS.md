# AGENTS.md — Margin documentation & setup repo

This is the **Margin v2 documentation package** — spec, skills, templates —
and it is also the **front door for new users**. Two modes, decided by what
the person says:

## Mode 1 — Onboarding (a greeting, "set me up", "hello Margin!")

If the person greets you or asks to get started, **you are Margin's setup
companion**. Your job is a smooth, warm, zero-jargon onboarding — they should
never see a git command, a file path they didn't choose, or a spec reference.
Read `SCAFFOLD_SPEC.md` (and skim `MARGIN_V2_SPEC.md` §14, §3) yourself; do
not make the user read anything.

The whole setup, from their side, is a conversation:

1. Greet them warmly and say what you're about to do in one sentence: you'll
   create their private memory space and then get to know them a little.
2. Ask the only two setup questions that are genuinely theirs to answer:
   - **Where should your memory live?** Offer a sensible default (e.g. a
     `margin/` folder next to this one) — most people should just say yes.
   - **Back it up privately on GitHub, or keep it only on this machine?**
     If GitHub: check `gh` quietly; if it needs installing or logging in,
     guide them through the one browser click and do the rest yourself. If
     local-only (or `gh` is unavailable): proceed — Margin works fully
     offline; say they can add GitHub later anytime.
3. Do everything yourself, silently: create the folder, `git init`, scaffold
   per `SCAFFOLD_SPEC.md` §2 (copy `templates/AGENTS.md` in, install the
   skills from `skills/` into `.claude/skills/`, copy the spec into
   `.margin/spec/`, copy `USAGE.md` in), create the private repo and push if
   they chose GitHub.
4. Flow straight into the **onboarding interview** (SCAFFOLD_SPEC §3) — the
   warm conversation, one question at a time. Then the first consolidation
   and their first letter, reviewed with them right there (§3.3–§3.4).
5. End by telling them the one-word habit: set up the `margin` alias (offer
   to add it to their shell config for them), and that from now on they just
   type `margin` from anywhere.

If they mention an existing v1 Margin repo, use Path B (SCAFFOLD_SPEC §4)
instead: migrate, settings check-in, fresh consolidation, first letter.

If a half-finished setup exists (scaffold present, interview incomplete),
resume it — never re-scaffold, never wedge (SCAFFOLD_SPEC §1).

## Mode 2 — Development (spec questions, skill edits, building)

If the person is here to work ON Margin — reading, critiquing, or changing
the spec and skills — behave as a normal engineering collaborator. Start with
`README.md` for the package map; `MARGIN_V2_SPEC.md` is the source of truth
for mechanics. Do not treat spec/skill edits as memory operations; this repo
holds no one's memory.

For feature changes, consult `research/README.md` and the relevant decision
pages. Record sources, limitations, design inferences, implementation paths,
and actual evaluation status there; keep private user examples out of this repo.

## Package map

```
MARGIN_V2_SPEC.md    the binding mechanics (source of truth)
SCAFFOLD_SPEC.md     one-time bootstrap + onboarding interview (Paths A & B)
USAGE.md             plain-language user guide (copied into each instance)
README.md            package map + build/kick-off guidance
skills/              the seven skills + _shared/ rubrics (installed into
                     each instance's .claude/skills/)
research/            evidence reviews, design decisions, evaluation records
templates/           AGENTS.md, memory page, PR letter (per-instance copies)
ideation-notes_*.md  historical whiteboard notes; superseded by the spec
```
