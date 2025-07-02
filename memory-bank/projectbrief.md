# Project Brief: CoAuth

## 1. Mission

To transform the `aider-ai/aider` codebase from a tool focused on code synthesis into **CoAuth**, a sophisticated **narrative synthesizer**. CoAuth will serve as a powerful co-pilot for human authors, assisting with drafting, ensuring consistency, maintaining style, and managing structural organization for complex narratives.

## 2. Scope

The project involves forking the `aider-ai/aider` repository and replacing its code-centric logic with a new cognitive architecture. This architecture will be powered by the `DSPy` framework to handle the nuances of creative writing.

The tool will remain an interactive, command-line application that integrates with git for version control.

## 3. Core Deliverables

*   A modified version of the `aider` shell, stripped of code-specific features and adapted for narrative content (primarily Markdown).
*   A "Cognitive Core" built with `DSPy` that orchestrates LLM-driven narrative tasks like drafting, editing, and analysis.
*   A "Memory System" that grounds the AI's creative output in user-defined context (a "World Bible" and "Style Guide").
*   A "Tooling Layer" for deterministic structural manipulation of documents.
*   A clear, user-controlled workflow where all AI-generated changes are presented as diffs for explicit approval before being committed.
