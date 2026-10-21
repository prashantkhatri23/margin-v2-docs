# Retrieval on implicit personal cues

Date: 2026-10-21. Status: implemented in documentation commit
[`f779ce7`](https://github.com/prashantkhatri23/margin-v2-docs/commit/f779ce7).
Evidence type: architecture inspection and an observed conversational routing
failure. This decision is an engineering judgment, not an empirical claim
about psychological benefit or the best retrieval architecture.

## Question and decision

How should a companion find relevant deeper memory when a personal request
does not explicitly ask it to remember?

Use the existing hierarchy: load the index map, pinned pages and appropriate
recent context; select deeper pages using descriptions and aliases; follow
relevant links and evidence refs as needed. Reassess when the topic or personal
significance changes. Personal drafting, preferences, choices and implicit
callbacks can justify retrieval without a recall command.

The motivating failure was a personal drafting exchange treated as generic
wording assistance. Relevant memory was discoverable, but targeted pages were
not consulted until the user explicitly questioned the lack of recognition.
The failure is documented at the level of routing behavior; private content
and identifying details are intentionally omitted.

## Why this change

The [read-side spec](../../MARGIN_V2_SPEC.md) already provided pinned, temporal,
indexed and cold tiers. The index supplies titles, descriptions, aliases and
confidence; links in pages form the graph. A title alone can hide that an
apparently broad page contains the interest or value needed for a reply.

The earlier shared trigger emphasized avoiding irrelevant retrieval, and its
examples did not clearly cover personal drafting or a question that becomes
personal partway through. Startup mechanics named pinned and recent context
without explicitly naming the index. The correction makes that existing map
and the positive retrieval cues operationally explicit.

This diagnosis is based on the observed sequence of reads and replies. It does
not prove a unique internal model cause, or that wording alone eliminates the
failure. The companion still has to apply the instruction in context.

## Implementation

- [Spec §8](../../MARGIN_V2_SPEC.md): explicit index loading, relevant graph
  traversal, and table entries for personal drafting and implicit callbacks.
- [Session mechanics](../../skills/_shared/session_mechanics.md): load the index
  and reconsider relevance when the conversation changes.
- [Companion voice](../../skills/_shared/companion_voice.md): select by
  descriptions and aliases, use sufficient loaded context, and retrieve when
  knowing the person improves the answer.
- [Conversation defaults](../../skills/margin-conversation/SKILL.md): explicit
  index map alongside pinned and temporal context.

## Tradeoffs and evaluation

No new graph store, embedding service, personality summary or blind directory
scan was introduced. Stop once the answer is grounded. General factual and
technical questions need no additional personal retrieval. Evidence and
confidence rules continue to govern what retrieved material supports.

Original validation covered skill frontmatter, whitespace, matching copies and
successful publication. It did not establish retrieval reliability. Behavioral
scenarios should distinguish a generic factual query, a personal drafting
request, an implicit callback after a topic change, and a request already
grounded in loaded context. Check both relevant retrieval and unnecessary
personalization. See also [care, perspective and growth](companion-support-and-growth.md).
