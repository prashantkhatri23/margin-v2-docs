outline a single, highly specialized memory-sharing skill along with an architectural course-correction and a companion variant.

Here is the parsed breakdown of how this specific skill is designed to function within the Margin CLI app.

---

## Skill Overview: Past-Recall Journaling

**Commands:** `/adding_event`, `/memory`, or `/share_a_part_of_life`

Unlike standard daily journaling, this skill is specifically built for **past-recall-based journaling**. It is triggered when a user wants to log a specific life event, memory, or distinct "part of life" rather than recap their entire day.

---

## Step-by-Step Execution Flow

### 1. Greeting & Flexible Input

* **Agent Trigger:** The agent greets the user and asks what specific memory or event they want to share.
* **User Flexibility:** The user's response doesn't have to be perfect. It can be a highly detailed paragraph or just a rough outline of the event they want to preserve.

### 2. Deep Contextual Conversation

* The agent carries forward the conversation with a high degree of **goal-orientation and subtlety**.
* **Retrieval Mechanic:** Unlike daily logging, this skill **does not need to look at recent raw entries**. Instead, it pulls directly from the deeper **memory file system structure** to offer personalized context.
* **The Bottom Line:** The goal of this interaction is to leverage existing memories to help the user reflect, ask personalized questions, and inject relevant context to **"enrich the memory recall."**

### 3. Local Checkpoints (Intermediate Saving)

* **At its own discretion**, the agent will automatically extract and store bullet-point highlights of the discussion at intermediate points.
* These bullet points act as **local checkpoints**, ensuring that if a terminal session drops or a user gets distracted, the progress of the memory recall isn't entirely lost.

### 4. Global Drafting & Memory Consolidation

* Once the conversation winds down, the agent compiles a **global draft** of the final entry.
* It presents this draft to the user to confirm if anything needs to be modified.
* Upon confirmation, the app **saves the entry and explicitly marks it for memory consolidation** into the user's permanent long-term storage.

---

## The Architectural Addendum 
There is an important structural realization noted in the top-right box that applies retroactively to *all* the guided/reflection skills designed so far:

* **The Rule:** When a session starts, it begins with basic "short-term personalized context."
* **Dynamic Access:** However, once the agent successfully moves the user into deep reflection or guided journaling, the LLM is dynamically granted explicit permission to access the **specialized components of the user's core memory structure files**. This allows the prompts to become progressively more tailored and context-relevant based on how the user responds in real time.

---

## The Companion Variant: `/conversation`

A standalone `/conversation` skill would borrow this exact same foundational framework, but with a different user intent:

1. **Broader, Open-Ended Goals:** While the core memory skill is highly specific (capturing an event), the `/conversation` skill is much broader. It still utilizes the memory file structure for personalization, but the ultimate objective is simply to act as an **AI Companion**—which is a core pillar of what the Margin app is intended to be.
2. **Turn-Tracking Only:** It continuously tracks raw user and agent turns for future memory consolidation, but it modifies how intermediate highlights are treated to keep the conversational flow more casual and companion-like.
