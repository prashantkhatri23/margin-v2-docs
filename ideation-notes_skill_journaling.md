The core philosophy here is to lower the barrier to habit-building by allowing the user to simply invoke a `margin` alias command in their terminal, with the system saving interactions turn-by-turn to prevent data loss.

---

## Core Architecture & Shared Mechanics

Regardless of the journaling mode, the backend relies on specific real-time state tracking and memory mechanics:

* **Turn-by-Turn Persistence:** The system stores and tracks every agent (LLM) and user turn immediately after it happens, rather than waiting until the end of the session.
* **Real-time Syncing:** Turns are continuously written to raw files during the session.
* **Memory Consolidation:** At the end of a session, files are flagged/marked so the system knows to look at them during the next "memory consolidation" phase (moving short-term interactions into long-term context).

---

## Skill 1: Guided Journaling (`[->] / guided journaling`)


This mode is designed to be highly interactive, where the agent takes the lead to reduce user friction (it's "easier to follow the lead than just share").

### 1. Session Initialization & Context Loading

* The agent starts a fresh session by assembling context from multiple streams: **visuals + recent memory + raw entries**.
* **The "Highlight" Check:** The session begins with a greeting and asks the user if there is a specific highlight from their day they want to focus on. If yes, the agent uses that highlight alongside general memory context to guide the conversation.

### 2. Flow & Conversation

* The agent guides the user through the entry by prompting them to share details about their day.
* To make the conversation feel organic and personalized, the agent injects context from *very* recent entries, past conversations, or specific "recent life domains."

### 3. Post-Processing & Entry Generation

* **Before Analysis:** Once the chat concludes, the agent takes the raw transcript and attempts to generate a cohesive "journal equivalent entry."
* **Faithfulness & Confirmation:** This generated entry must stay strictly faithful to what the user wrote and felt. The agent then presents it to the user to confirm/approve the final entry.

---

## Skill 2: Traditional Journaling (`[->] / journaling`)


This mode follows a traditional journaling route where the user wants to write freely or point the CLI to an existing draft file (image, audio, text etc), using the AI primarily as an editor and sounding board.

### 1. The Execution Flow

The whiteboard outlines a distinct step-by-step pipeline for this skill:

```
[New Session] 
      │
      ▼
[Agent Greetings] ──► (Options: Free-write or direct agent to an existing entry file)
      │
      ▼
[User Provides Entry]
      │
      ▼
[Agent Fills Initial Draft File] ──► (Automatically marks it for memory consolidation)
      │
      ▼
[Refinement Prompt] ──► "Do you need help refining this?"

```

### 2. Refinement Pathways

If the user wants to refine their draft, the agent can offer assistance across several layers:

* **Trivial fixes:** Correcting grammar, syntax, or fixing typos.
* **Non-trivial structural help:** Improving organization and narrative flow.
* **Personalized / Memory-Based Prompts:** If the agent needs to pull in **"Short-Term Personalized Context,"** it draws from:
* Recent raw entries (past conversations or journal logs).
* The active short-term memory component.



### 3. Finalization Outcomes

Once the entry is completed, the system branches into finishing tasks:

* Ensuring everything is safely saved and explicitly marked for long-term memory consolidation.
* Injecting a final bit of reflection text into the file to finalize the journal entry.