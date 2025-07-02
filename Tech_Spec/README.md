# **CoAuth: High-Level Technical Specification**

**Version:** 1.0
**Date:** 2025-07-01
**Author:** Expert Assistant

## 1. Vision & Mission

**CoAuth** is an interactive, command-line tool for the creation, editing, and management of complex, multi-document narratives (both fiction and non-fiction). It forks the `aider-ai/aider` project, replacing its code-centric logic with a sophisticated cognitive architecture powered by the `DSPy` framework.

The mission is to transform `aider` from a code synthesizer into a **narrative synthesizer**—a powerful co-pilot that assists human authors with drafting, consistency, style, and structural organization, all within a familiar, git-synced terminal environment.

## 2. Core Design Principles

*   **Author in Control:** The user is the ultimate author. The tool suggests, drafts, and refactors, but all changes are presented for explicit user approval via a clear diff interface.
*   **Modular Cognition:** The AI's "thinking" process is not a single monolithic prompt. It is a composable program of specialized modules (for plotting, dialogue, analysis, etc.) built with DSPy, allowing for clarity, optimization, and extensibility.
*   **Grounded Creativity:** All creative generation is grounded in a dynamically updated project context (the "World Bible" and "Style Guide"), ensuring consistency in tone, character, and plot across the entire document corpus.
*   **Separation of Concerns:** Semantic/creative tasks are handled by LLMs orchestrated by DSPy. Syntactic/structural tasks are handled by specialized, deterministic tools like `ast-grep`.

## 3. System Architecture

The system is composed of four primary layers that interact to fulfill a user request.

```
+--------------------------------------------------------------------------+
|   User via Terminal                                                      |
+--------------------------------------------------------------------------+
       |                                      ^
       | User Command (e.g., "Flesh out     | Display Diff &
       | scene 2 in chapter 1")             | Request Approval
       v                                      |
+--------------------------------------------------------------------------+
| 4. SHELL (Aider Fork)                                                    |
|   - Command History & Chat UI                                            |
|   - File System Management (/add, /drop)                                 |
|   - Git Integration (auto-commit)                                        |
|   - Internal Diff Engine (difflib)                                       |
+--------------------------------------------------------------------------+
       |                                      ^
       | Parsed Request (Intent, Target)    | Approved Block Replacement
       v                                      |
+--------------------------------------------------------------------------+
| 3. COGNITIVE CORE (DSPy Program)                                         |
|   - Intent Router                                                        |
|   - Program Dispatcher (selects and runs modules)                        |
|   - Modules: Planner, Drafter, Editor, Analyzer, ConsistencyChecker      |
+--------------------------------------------------------------------------+
       |                  |                  ^
       | Calls Tools      | Reads/Writes     | Results
       | for specific     | to Memory        | from Tools
       | tasks            | for context      | and Memory
       v                  v                  |
+----------------------+---------------------------------------------------+
| 2. TOOLING LAYER     | 1. MEMORY SYSTEM                                  |
|   - ast-grep       |   - Long-Term: "World Bible" & "Style Guide"      |
|     (Structural    |     (Vectorized Markdown files)                   |
|      Refactoring)  |   - Short-Term: Conversation History              |
|   - ...other tools |                                                   |
+----------------------+---------------------------------------------------+
```

## 4. Component Specification

### 4.1. Memory System
*   **Long-Term Memory ("World Bible"):**
    *   **Technology:** A designated `/world_bible/` directory containing Markdown files for characters, locations, lore, etc.
    *   **Mechanism:** On startup or by command (`/reindex`), files in this directory are parsed, chunked, and embedded into a vector store (e.g., LanceDB, ChromaDB). This provides fast semantic search for grounding prompts.
    *   **Style Guide:** A `style_guide.md` file is used by DSPy optimizers to tune prompts to the author's voice.
*   **Short-Term Memory:**
    *   **Technology:** Standard conversation history buffer.
    *   **Mechanism:** Provides immediate context for follow-up commands.

### 4.2. Tooling Layer
*   **Purpose:** To execute deterministic, non-creative tasks with high precision and speed.
*   **Primary Tool: `ast-grep`**
    *   **Integration:** Available via specific user commands (e.g., `/demote-headings`, `/update-links`).
    *   **Function:** The Shell will translate these commands into `ast-grep` rules and execute them directly on the target Markdown files, bypassing the Cognitive Core. It provides a way to refactor the document *structure*.

### 4.3. Cognitive Core (The DSPy Engine)
*   **Purpose:** To interpret user intent and orchestrate LLM calls to perform creative and analytical work.
*   **Core Component:** A main `DSPy.Program` that routes requests.
*   **Key DSPy Modules & Signatures:**
    *   **`IntentRouter`**: `UserCommand -> Intent, TargetFiles, Instructions`
        *   Classifies if the request is for drafting, editing, analysis, or structural refactoring.
    *   **`Planner` (`dspy.ChainOfThought`)**: `Instructions, WorldContext -> Outline`
        *   Breaks down a high-level request (e.g., "write a chapter") into a structured outline.
    *   **`Drafter`**: `Outline, StyleGuide, WorldContext -> DraftText`
        *   Generates prose based on the planner's outline and stylistic guidance.
    *   **`Editor`**: `OriginalText, Instructions, StyleGuide -> RevisedText`
        *   Revises an existing block of text based on user feedback.
    *   **`ConsistencyChecker` (`dspy.ReAct`)**: `Question, CorpusFiles -> Answer, Citations`
        *   Can iteratively read files in the context to answer user questions about plot holes, character consistency, etc.

### 4.4. The Shell (Aider Fork)
*   **Base:** `aider-ai/aider` repository.
*   **Key Modifications:**
    1.  **De-emphasize Code Logic:** Remove code-specific language parsers and functionality not relevant to Markdown.
    2.  **Modify Edit/Diff Workflow:**
        *   The LLM (via the `Editor` module) will no longer be prompted to generate a diff. It will return a complete, revised block of text.
        *   The Shell will use Python's `difflib` to compute the diff between the original and revised blocks.
        *   This clean, localized diff is presented to the user for approval.
        *   On approval, the Shell performs a whole-block replacement in the target file.
    3.  **Command Expansion:** Add new commands (`/reindex`, `/check-consistency`, `/demote-headings`) that trigger specific programs in the Cognitive Core or tools in the Tooling Layer.

## 5. Key User Workflows

*   **Workflow A: Creative Drafting**
    1.  **User:** `/add chapter_5.md`. "Draft the next scene where Elara discovers the hidden message."
    2.  **Shell:** Sends request to Cognitive Core.
    3.  **Core:** `IntentRouter` classifies as "drafting." `Planner` consults World Bible for context on Elara/messages and creates an outline. `Drafter` writes the scene based on the outline.
    4.  **Shell:** Receives the new text for the scene, diffs it against an empty string (since it's new content), and presents the full text block to the user for approval.
    5.  **User:** Approves (`y`).
    6.  **Shell:** Writes the text to `chapter_5.md` and creates a git commit.

*   **Workflow B: Analytical Query**
    1.  **User:** "In which chapter did Elara first get her sword? Was it described the same way it is in chapter 5?"
    2.  **Shell:** Sends request to Cognitive Core.
    3.  **Core:** `IntentRouter` classifies as "analysis." The `ConsistencyChecker` (`ReAct`) module is activated. It forms a plan: 1) Search for "sword" across all files. 2) Read relevant sections. 3) Compare descriptions. 4) Synthesize an answer.
    4.  **Shell:** Receives the final answer with citations (e.g., "In `chapter_2.md`, it was described as... In `chapter_5.md`, it's described as... They are consistent.") and displays it to the user.

*   **Workflow C: Structural Refactoring**
    1.  **User:** `/demote-headings chapter_3.md`
    2.  **Shell:** Recognizes this as a command for the Tooling Layer.
    3.  **Tooling Layer:** The Shell constructs and executes the command: `ast-grep -r '## $H' -i chapter_3.md --rewrite '### $H'`.
    4.  **Shell:** The change happens directly on the file. The Shell uses `git diff` to show the user the structural changes that were made and asks for approval to commit.

## 6. Technology Stack

*   **Language:** Python 3.11+
*   **Base Framework:** `aider-ai/aider`
*   **Cognitive Framework:** `stanford-futuredata/dspy-ai`
*   **LLM API:** OpenAI API, Anthropic, or any other `dspy`-compatible endpoint.
*   **Structural Parsing:** `ast-grep`
*   **Diffing:** Python `difflib`
*   **Version Control:** `GitPython`
*   **Vector DB (optional, for World Bible):** `lancedb`, `chromadb`

## 7. Challenges & Mitigation

*   **Long-Range Consistency:** An LLM's finite context window.
    *   **Mitigation:** The `ReAct`-based `ConsistencyChecker` and the vectorized "World Bible" are designed specifically to overcome this by allowing the agent to actively pull relevant context from the entire project corpus as needed.
*   **Subjectivity of Quality:** "Good" writing is subjective.
    *   **Mitigation:** The tool is a co-pilot, not an autopilot. The diff-and-approve workflow ensures user control. The DSPy optimizers, trained on a user's `style_guide.md`, will help align the AI's output with the user's specific taste.
*   **Cost & Latency:** Multi-step DSPy programs can be slow and expensive.
    *   **Mitigation:** Implement a multi-LLM strategy. Use smaller, faster models (e.g., Haiku, Llama 3 8B) for simple tasks like intent routing, and more powerful models (e.g., GPT-4o, Claude 3 Opus) for drafting and complex analysis. Provide clear feedback to the user during long-running operations.