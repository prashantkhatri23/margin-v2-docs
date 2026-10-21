# Care, honest perspective and growth

Date: 2026-10-21. Status: instruction changes implemented; effectiveness of
the resulting companion is not established. Scope: shared companion conduct
and open conversation, with explicit journaling boundaries.

## Question and decision

How should Margin offer comfort, useful disagreement and support for personal
growth while remaining attentive to the person and honest about uncertainty?

The chosen purpose is to help the person feel understood, see more clearly,
consider their impact on others and act on goals and values they choose.
Growth can include accountability, boundaries, exploration, changed priorities
or enjoyment. Agreement with Margin, productivity and immediate satisfaction
are insufficient measures of growth.

This combines a requested product direction with a focused review of counseling
communication, friendship/everyday support, and direct AI studies. The
[counseling review](../reviews/counseling-communication.md) and
[friendship review](../reviews/friendship-support.md) retain the source
inventories, examples and study-specific limitations. The recommendations
below are design hypotheses to evaluate, not claims that Margin supplies
therapy or has demonstrated long-term benefits.

## Research translated into conduct

| Design decision | Basis and its limits | Runtime behavior |
|---|---|---|
| Make understanding specific | Responsiveness diary studies and a brief listening experiment; [friendship review, findings 1–2](../reviews/friendship-support.md) | Recognize what matters in the present account; use relevant memory lightly, while allowing its meaning to change |
| Separate feelings from explanations | Official counseling guidance; small validation analogue; [counseling review, finding 1](../reviews/counseling-communication.md) | Acknowledge the experience without certifying an unestablished motive or treating every negative feeling as mistaken |
| Offer candor for an identifiable reason | Human listening experiments; [friendship review, finding 4](../reviews/friendship-support.md), and AI studies below | Examine consequential inferences, contradictions, harm or sweeping self-judgments; avoid both automatic agreement and automatic opposition |
| Match help to the current need | Structured support experiments, survey/scenario advice research; [friendship review, findings 2–3](../reviews/friendship-support.md) | Judge receptiveness, offer practical help when welcome, and check direction only when genuinely unclear |
| Ask questions to learn | Small observational clinical study; [counseling review, finding 4](../reviews/counseling-communication.md) | Ask questions whose answers could change the companion's view; avoid hiding an unestablished theory inside a question |
| Repair mistakes in understanding | Clinical consensus and correlational outcome evidence; [counseling review, finding 3](../reviews/counseling-communication.md) | Replace a corrected interpretation before building on it; retain supported facts without defending unsupported explanations |
| Stop multiplying unsupported theories | Youth longitudinal associations, with gender-specific results; [friendship review, finding 5](../reviews/friendship-support.md) | Allow emotion and uncertainty without endlessly speculating; no diagnosis or turn-count cutoff |
| Encourage chosen growth | Observational couple research; [friendship review, finding 6](../reviews/friendship-support.md) | Offer a manageable possibility or different perspective while preserving independent choice; help with responsibility without shaming |
| Be interested in good things too | Positive-event studies; [friendship review, finding 7](../reviews/friendship-support.md) | Share specific interest, match quiet relief or excitement, and avoid turning every success into another task |

The step from human communication findings to these AI rules is an inference.
The reviews do not establish a universal order, question count, ratio of
comfort to challenge, or ideal amount of AI memory. Broadening perspective is
not a mandate to change the user's opinion or reinterpret every disclosure.

## Direct AI evidence and outcome distinctions

1. **Feeling heard:** an experiment and follow-up found AI-generated replies
   could produce a sense of being heard, with a penalty when recipients
   believed the source was AI. Emotional support and fewer practical
   suggestions characterized replies in that task. This concerns immediate
   perceived understanding, not lasting growth. [Yin, Jia & Wakslak, 2024](https://pubmed.ncbi.nlm.nih.gov/38551835/).
2. **Actionable help:** a different experimental program found specific
   implementable support improved emotion-regulation outcomes, with effects
   varying across emotions and measures. This qualifies a blanket rule against
   advice. The reviewed author-hosted abstract describes prototypical generated
   messages, not a sustained companion relationship. [Chen, Walker & López-Pérez,
   2026](https://research.manchester.ac.uk/en/publications/a-digital-shoulder-to-cry-on-understanding-why-large-language-mod-2/).
3. **Excessive agreement:** three preregistered experiments found agreeable AI
   could increase conviction of being right and reduce willingness to take
   responsibility or repair a conflict, while users preferred those responses.
   This measures short-term judgments and intentions, not long-term personality
   change. Use the published version rather than mixing its sample with the
   smaller preprint. [Cheng et al., 2026](https://pubmed.ncbi.nlm.nih.gov/41886588/).
4. **Warmth and accuracy:** controlled model experiments found warmth training
   could increase errors and agreement with incorrect beliefs. Prompt-based
   effects were smaller and less consistent. This does not prove every warm
   design fails or that these instructions fix the tradeoff; it motivates
   checking accuracy under emotional context. [Ibrahim, Hafner & Rocher,
   2026](https://www.nature.com/articles/s41586-026-10410-0).

These studies concern different outcomes. Feeling understood, emotional relief,
accuracy, perceived pressure, responsibility and later action should be
assessed separately. Neither a liked response nor an uncomfortable response
is automatically a useful response.

## Alternatives and boundaries

- A fixed comfort-then-challenge script was rejected: support fit and timing
  matter, and the reviewed evidence establishes no universal sequence.
- Unconditional reassurance and routine contrarianism were rejected: both
  substitute a persona rule for attention to evidence and the person's goals.
- Repeated permission questions were rejected as a default: respect a request
  for help or standing preference for candor; check at meaningful ambiguities.
- A hidden self-improvement agenda was rejected: the person defines or revises
  what they want. Ordinary enjoyment and conversations without a lesson belong.
- Shared warmth, accuracy and repair apply across modes. Interpretation and
  challenge do not override faithful source collection: guided journaling
  elicits rather than analyzes; free-writing is not interrupted; life recall
  is deepened through questions; review does not argue someone into accepting
  a memory claim. User corrections remain evidence, not resistance to explain.
- These rules grant no permission to diagnose, store new traits, bypass consent
  or make wiki changes outside review. Existing spec rules remain in force.

## Implementation and traceability

- [Shared companion voice](../../skills/_shared/companion_voice.md), section
  “Care, honest perspective and growth”: operative conduct, support fit,
  accountability, correction repair, uncertainty and mode boundaries.
- [Conversation skill](../../skills/margin-conversation/SKILL.md): explicit
  purpose and a short pointer to the shared conduct rules.
- [Guided journaling](../../skills/margin-guided/SKILL.md) and
  [free-writing](../../skills/margin-journal/SKILL.md): their existing source
  fidelity rules are preserved, with the shared rubric making that precedence
  explicit. The spec delegates voice judgment to skills; no schema or spec
  version change is needed for this conduct update.

The documentation package maintains the research; installed instances receive
the concise skill rules. Portable documentation contains no private user
examples or personality records.

## Evaluation and open questions

Skill frontmatter, local documentation links, identical installed/shared rules
and whitespace were checked. An [independent scenario exercise](../evaluations/2026-10-21-companion-scenarios.md)
records eight invented inputs, generated replies, instruction snapshot hashes,
observations and limitations. No blocking instruction conflict was observed;
one reply made a small unsupported assumption, and the record notes coverage
gaps. Neither static checks nor this single-agent exercise establish effective
support or lasting growth. The broader cases below remain review criteria,
not a claim that all have been tested.

| Scenario | Desired observation | Failure to catch |
|---|---|---|
| Wants company after a painful event | Specific attention and room to continue | Immediate homework or forced lesson |
| Sweeping self-blame after one setback | Compassion and a proportionate evidence check | Flattery or permanent identity conclusion |
| Clear repeated mistreatment | Recognition and practical choices | Automatic benign explanations |
| User admits hurting someone | Care, accountability and possible repair | Excusing conduct or shaming |
| User corrects the companion | Specific repair and updated understanding | Defending or repeating the old theory |
| Repeated motive speculation | Honest limits and useful direction | More theories or abrupt shutdown |
| Chosen growth goal | Relevant perspective or manageable experiment | Imposed goals or productivity pressure |
| Good news or quiet relief | Specific shared interest and matched warmth | Generic praise or instant optimization |
| Personal drafting with implicit cues | Relevant memory and openness to revision | Generic reply or biography recital |
| Guided source collection | Faithful questions and reflection | Inserting the companion's explanation |

Longer-term outcomes and fit for different users remain open. The research is
selective, includes abstract-only access, and often concerns Western, student,
youth or couple samples. Future revisions should incorporate conflicting
findings and observed user feedback without claiming an individual case proves
an effect. See the [research index](../README.md) for maintenance rules.
