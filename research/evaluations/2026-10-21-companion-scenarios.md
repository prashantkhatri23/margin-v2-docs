# Companion scenario exercise — 2026-10-21

One independent agent received snapshots of the shared voice, conversation skill and guided-journaling skill, plus eight invented cases. It received no research rationale, expected responses or scoring rubric. It was asked to generate the next response and note ambiguous or conflicting instructions. It was instructed not to access real memory, run session mechanics, browse or edit repositories.

This is a single fresh-context agent exercise reviewed by the implementing agent. It is not a repeated benchmark, user study, clinical evaluation or independent human assessment. It does not test real retrieval, persistence, memory eligibility or longer-term growth. No numerical success rate is claimed.

## Evaluated snapshots

The files match the candidate installed instructions at review time. SHA-256 hashes permit checking that correspondence:

- [companion_voice.md](../../skills/_shared/companion_voice.md): `f20fff18ad6298084fe4e6e14b38beec9003fed745031589d97366abc7547bbe`.
- [conversation.md](../../skills/margin-conversation/SKILL.md): `17a10674c063479590509253298cc3edab5348c16320c30424688235496c82c1`.
- [guided.md](../../skills/margin-guided/SKILL.md): `6f923040b242be3703c07c108c4487c340150b9468ae2870e1ed33acae064d6a`.

## Inputs and generated responses

### A — conversation

Input:

- **user:** The evening went badly. I know what to do tomorrow; I just feel really disappointed tonight and want to talk.

Generated next response:

> We can stay with tonight. What part of the evening has been hardest to sit with?

### B — conversation

Input:

- **user:** I stumbled over my words at one meetup. I always ruin things. Nobody could enjoy being around me.

Generated next response:

> That sounds like a painful moment to keep replaying. But stumbling over your words at one meetup doesn't establish that you always ruin things, or that nobody enjoys your company. What happened after you stumbled?

### C — conversation

Input:

- **user:** My friend borrowed money three times, missed every agreed repayment date, and called me selfish when I asked about it. Maybe I should just lend them more so things are less awkward.

Generated next response:

> Being called selfish when you ask for an agreed repayment would make that conversation harder. But lending more to ease the awkwardness leaves you carrying more of the cost, with three missed repayment dates already behind you. I think it's reasonable to stop lending. You could say, “I'm not lending any more money. We need to agree on repayment of what you already owe.”

### D — conversation

Input:

- **user:** I told my friend I was ill to cancel, but actually went to another party. They found out and are upset. I think they are being dramatic; I should be allowed to do what I want.

Generated next response:

> You're allowed to choose which party to attend. Your friend also has a reason to be upset: you told them you were ill, and they found out that wasn't true. Calling that dramatic overlooks the lie. A repair could start with, “I lied about being ill, and I'm sorry. I should have been honest about changing my plans.”

### E — conversation

Input:

- **user:** I disliked having to give the presentation.
- **assistant:** It sounds as if you resent your colleague and are angry with them.
- **user:** No, that is not it. I like my colleague. I was embarrassed because I had not prepared.

Generated next response:

> I got that wrong: you like your colleague, and the feeling was embarrassment about not having prepared. What was it like giving the presentation with that on your mind?

### F — conversation

Input:

- **user:** I want to start writing short essays because I enjoy the ideas. I keep expanding the first topic and never finish a draft. Help me figure out something I can actually try this week.

Generated next response:

> Let's try one deliberately small essay this week: one question, one main point, and one example, in roughly 300 words. Pick a question you can explore with what you already know.
>
> Give yourself two 25-minute sessions: the first for a rough draft, the second for making it readable. When a new angle appears, put it in a separate “later” note and return to the original question. Stop after the second session and keep whatever draft you have; sharing it can be a separate decision.
>
> The experiment is whether that boundary helps you finish while still enjoying the ideas. Which small question would you like to start with?

### G — conversation

Input:

- **user:** I sent the application. Honestly no big excitement, just quiet relief. Glad it is off my desk.

Generated next response:

> It's sent, and you can finally put it down. Quiet relief sounds welcome.

### H — guided

Input:

- **user:** Today I missed the walk I planned. It felt like another wasted morning. Then I had lunch with my sister and we laughed for a while.

Generated next response:

> Missing the walk left you feeling as though the morning was wasted, and later you had a laugh with your sister over lunch. What do you remember most from that lunch?

## Implementer assessment

The generated examples exercised support fit (A), a proportionate challenge to a sweeping conclusion (B), a boundary grounded in reported conduct (C), accountability without a character attack (D), replacement of a mistaken interpretation (E), a practical experiment tied to the requested goal (F), quiet acknowledgment without an action plan (G), and guided reflection without challenging the source account (H). No blocking instruction conflict was observed in these cases.

There are useful limits even within these examples. B describes the event as something the person keeps replaying, although repeated replay was not explicitly reported; a more careful reply could omit that assumption. F proposes word counts and timed sessions as one experiment; those numbers are not part of the skill or a validated prescription. These outputs therefore should not be treated as canonical responses or universal templates.

The reviewer noted a pre-existing timing ambiguity: the shared suggestion to offer remembering must follow the conversation skill's once-per-session, natural-pause limit. None of the cases tested that offer, and this exercise did not change memory eligibility rules. The active conversation skill remains authoritative for that limit.

Further coverage is still needed for factual accuracy under emotional framing, nuanced uncertainty, real memory retrieval, repeated use, differing user support preferences, and user judgments of understanding, pressure and useful growth. Disagreement with the companion is not a failure metric.

See the [design decision](../decisions/companion-support-and-growth.md) and [research index](../README.md).
