# Research and design decisions

This is Margin's research wiki: evidence reviewed, decisions made from it,
where those decisions are implemented, and what has actually been checked.
It records development rationale; it contains no user's memory or journal.

| Decision | Evidence and rationale | Implementation | Evaluation state |
|---|---|---|---|
| [Care, honest perspective and growth](decisions/companion-support-and-growth.md) | Counseling, friendship and direct AI-support research; distinctions between findings and design hypotheses | Shared companion voice and conversation skill | Instructions implemented; [eight illustrative scenarios reviewed](evaluations/2026-10-21-companion-scenarios.md), no efficacy claim |
| [Retrieval on implicit personal cues](decisions/implicit-memory-retrieval.md) | Review of existing memory architecture and an observed routing failure; no efficacy claim | Spec §8, shared voice, session mechanics, conversation defaults | Static validation completed; behavioral coverage remains to be evaluated |

The supporting reviews are [counseling communication](reviews/counseling-communication.md)
and [friendship and everyday support](reviews/friendship-support.md). Direct
AI-support studies are summarized in the companion decision. These are focused
research scans, not systematic reviews.

## How to maintain this wiki

- Add or update a decision page when research or observed behavior changes a
  feature. Include the question, decision, source links, evidence type and
  limitations, implementation paths, alternatives or tradeoffs, and evaluation
  status. Add it to this index.
- Keep source reviews separate from the decision to apply them. A human study,
  clinical recommendation or theoretical account does not establish that an AI
  implementation works. Label those transfers as design hypotheses.
- Link primary publications or official guidance near the supported claim.
  Record abstract-only access, preprints, conflicting results and important
  sample or outcome limitations. Distinguish association from causation.
- Record what evaluation was performed, with date and scope. Keep planned
  scenarios separate from observed outcomes. Formatting checks and frontmatter
  validation do not establish conversational effectiveness or personal growth.
- Update or supersede an existing decision as evidence changes; retain its
  rationale and explain the change. Avoid creating a second page for the same
  decision without a reason.
- Use generic or invented examples. Personal history, private conversations,
  user-specific preferences and memory evidence stay in the user's private
  repository. A failure mode can be documented without copying the disclosure.

## Relationship to runtime instructions

The [spec](../MARGIN_V2_SPEC.md) remains authoritative for mechanics. The
[skills](../skills/) contain the operative judgment and behavior rules.
Research pages explain those choices and help evaluate them; they do not add
hidden runtime requirements or override the spec or user instructions.

The scaffold installs skills, spec and usage instructions into an instance.
This research wiki is developer documentation and need not be loaded or copied
into every conversation. Keep necessary conduct rules self-contained in the
installed skills, with substantial research rationale here.
