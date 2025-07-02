# System Patterns: CoAuth

## 1. Core Architecture: Shell-Core-Tools

The system is designed around a clear separation of concerns, embodied by three main components:

*   **The Shell (Aider Fork):** Manages the user interface, file system, and version control. It is the "view" and "controller" of the application. It translates user input into commands and presents results for approval.
*   **The Cognitive Core (DSPy):** Handles all semantic and creative tasks. It is the "brain" of the operation, interpreting intent and orchestrating LLMs to generate or analyze text. It is stateless from the Shell's perspective; it receives a request and returns a result.

This pattern ensures that creative, probabilistic work is isolated from deterministic, structural work, and both are managed by a user-facing shell that ensures authorial control.

## 2. Memory System: Grounded Creativity

All creative generation is grounded in a well-defined memory system to ensure consistency and relevance.

*   **Long-Term Memory (World Bible & Style Guide):** This is the system's "source of truth" for the narrative's content and style. The Cognitive Core **must** consult this memory when performing relevant tasks (e.g., drafting, consistency checking). The `World Bible` is implemented as a collection of user-managed Markdown files in a `/world_bible/` directory, which are vectorized for efficient semantic search. The `Style Guide` is a specific file used for prompt optimization.
*   **Short-Term Memory (Conversation History):** This provides immediate context for follow-up commands and clarifications.

This pattern prevents the AI from "hallucinating" details that contradict the established canon and helps align its output with the author's voice.

## 3. Interaction Pattern: Diff and Approve

The cardinal rule of CoAuth is that the **author is always in control**. No change is ever made to a file without explicit user approval.

1.  The Cognitive Core or Tooling Layer generates a proposed change (e.g., a new paragraph, a revised scene, a structural modification).
2.  The Shell receives this proposed change.
3.  The Shell computes a diff between the original content and the proposed change using `difflib`.
4.  The diff is presented to the user in a clear, easy-to-read format.
5.  The user must explicitly approve the change (e.g., by typing `y`).
6.  Only upon approval does the Shell write the change to the file and create a git commit.

This pattern builds trust and ensures that the AI acts as a co-pilot, not an autopilot.
