# Technical Context: CoAuth

## 1. Technology Stack

*   **Base Framework:** `aider-ai/aider` (forked)
*   **Primary Language:** Python 3.11+
*   **Cognitive Framework:** `stanford-futuredata/dspy-ai`
*   **Version Control Integration:** `GitPython`
*   **Diffing Engine:** Python's built-in `difflib`
*   **Vector Database (for Memory System):** To be selected, with `LanceDB` or `ChromaDB` as primary candidates.
*   **LLM Endpoints:** Configurable to support any `dspy`-compatible API (e.g., OpenAI, Anthropic, local models).

## 2. Environment

*   **Operating System:** The tool is developed and tested in a Unix-like environment (Linux, macOS).
*   **Dependencies:** All Python dependencies are managed via `requirements.txt` and `pip`.
*   **Execution:** CoAuth is a command-line application.

## 3. Mocking and Testing Policy

*   **External Services:** True external services, such as the LLM APIs (OpenAI, etc.), should be mocked during integration tests to ensure speed, determinism, and cost control.
*   **Internal Modules:** Internal modules should **not** be mocked. The architecture relies on the interaction between the Shell, Cognitive Core, and Tooling Layer. Tests should validate these interactions directly. The `DSPy` framework has its own testing and evaluation methodologies that should be leveraged for the Cognitive Core.
*   **Behavior-First Development:** New features should be developed with a "behavior-first" approach. Define the desired user interaction and outcome first, potentially as a test case, before implementing the underlying logic.
