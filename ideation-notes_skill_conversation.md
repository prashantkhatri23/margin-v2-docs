These notes focus on the low-level workspace architecture for the `/conversation` command, along with a clever strategy for handling agent initialization without blowing out the context window with raw data.

Here is the parsed structural breakdown of the workspace layout and the newly proposed context-bootstrap layers:

---

## 📂 The `/conversation` Folder Structure

For every new interaction session, the app initializes a dedicated conversation folder based on a standardized template. The notes outline five core files to manage this workspace state:

* **Raw Turns File (`.json` or similar):** Stores sequential raw conversation turns. The format needs to allow lightweight, incremental updates after every single turn and fast retrieval of specific past turns.
* **Metadata File:** Tracks high-level execution data such as `creation_date`, `updated_date`, and total `number_of_turns`.
* **Summary File (`.md`):** Keeps a running, up-to-date narrative summary of the active conversation.
* **Highlights File:** Regularly captures and appends key bullet points or milestones mid-session.
* **Memory Citations Log:** A dedicated file to track and log exactly which historical memory files the agent referenced/cited during the chat.

---

## 🧠 Agent Initialization & Bootstrapping

To make the agent feel deeply personalized right out of the gate—without passing massive amounts of raw history into the prompt context—the initialization pipeline loads a layered snapshot of the user:

### 1. Core Profile Reading

The agent reads the user's **identity-related memories in full**, combined with a high-level **memory overview file**. This establishes a foundational baseline of who the user is.

### 2. The Memory Structure Map *(The "Segue" Concept)*

The notes introduce a new file concept explicitly designed to familiarize the LLM with the organization of the long-term storage directory.

* **Must Contain:** A clean, tree-like layout of the active memory file system.
* **Purpose:** Allows the model to know exactly where certain types of information live so it can promptly and accurately query or access deep file paths later as the conversation naturally shifts.

---

## ⏳ The Temporal Local Memory Layer

The final section of the notes addresses a gap in the current memory design: bootstrapping the agent with the user's *immediate* recent life context before a session or guided journaling task begins.

Instead of shoving a massive backlog of raw text into the prompt, the system injects a targeted **Temporal Local Memory** footprint containing:

* Bullet-point highlights from the most recent events and daily entries.
* Detailed summaries of the last few sequential journal entries.
* Crucial, immediate "need-to-know" details that are brand new and haven't yet been officially processed or consolidated into the permanent long-term identity files.

