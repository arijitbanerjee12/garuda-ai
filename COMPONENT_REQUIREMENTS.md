# Component-Level Requirements (Detailed PO Document)

This document breaks down the high-level Product Requirements (PRD) into granular, component-level specifications for the development team.

## 1. Test Definition & Configuration Component
**Objective:** Provide a standardized, version-controllable way for users to define their test suites.
- **Format:** Users will define test cases, rubrics, and agent personas using **YAML or JSON** configuration files.
- **RAG Data Handling:** For RAG-specific evaluation, the framework must support ingesting expected context/ground truth via **Excel (.xlsx/.csv)** files. This caters to scenarios where the target app does not natively return its retrieved context.
- **Flexible Test Execution:** Built natively on `pytest`, users can write standard Python tests to leverage all existing `pytest` features, plugins, and fixtures. It also fully supports `pytest-bdd` for users who prefer writing tests in Gherkin syntax (Given/When/Then).

## 2. LLM Evaluator Engine (The "Judge")
**Objective:** The core brain of the framework responsible for grading target responses and roleplaying personas.
- **Primary Provider:** Initial implementation will default to **Groq** for high-speed inference.
- **Extensibility:** The engine must be provider-agnostic, supporting Cloud APIs (OpenAI, Anthropic) and Local Models (Ollama, vLLM) interchangeably via a unified interface.
- **Evaluation Logic:** The framework will adopt **DeepEval's structure** for RAG evaluation metrics (e.g., Contextual Precision, Contextual Recall, Faithfulness, Answer Relevance).
- **Determinism Controls (Flakiness Handling):** Users can explicitly configure agent generation parameters (like `temperature`). Additionally, users can opt into a **"Consensus Mode"** (e.g., run the evaluation 3 times and take the majority vote) at runtime to ensure CI stability.
- **Rate Limiting & Cost Control:** The framework will include built-in rate limit handling (e.g., exponential backoff) configurable by the user, preventing test suite crashes from HTTP 429 errors.

## 3. Target Application Interface (Adapter Layer)
**Objective:** How Garuda AI connects to the systems it is testing.
- **User-Defined Wrappers (BYO-Adapter):** Because target API schemas and streaming protocols (SSE, WebSockets) vary wildly, Garuda AI will *not* attempt to parse them natively. Instead, the framework will define an Interface/Base Class.
  - The Framework generates the prompt (input).
  - The **User** writes a custom Python wrapper (HTTP or SDK) to hit their specific target app and aggregate any streaming responses.
  - The Framework consumes the final aggregated string/JSON output for evaluation.
- **State & Context Maintenance:** The Evaluator testing agent will maintain conversation state (using Session IDs or Message IDs) internally and pass these to the user's wrapper, ensuring context isolation across parallel test runs.

## 4. Reporting & Execution Component
**Objective:** Generate actionable reports and run independently or within CI/CD.
- **Standalone Execution:** CI/CD integration is strictly optional. The framework functions perfectly as a local command-line testing tool.
- **Test Runner:** The framework utilizes `pytest` under the hood.
- **Allure Integration:** Test runs generate **Allure Reports** to provide rich, visual dashboards.
- **Cucumber JSON:** Execution outputs **Cucumber JSON** artifacts (via `pytest-bdd`) to integrate with enterprise test management tools.

## 5. Component Architecture Summary
To support the above, the codebase will be structured into the following distinct modules:
- `garuda/config`: YAML/JSON parsers, Excel data loaders, and Rate Limit configurations.
- `garuda/evaluators`: Groq-powered Judge agents, Consensus voting logic, and DeepEval metrics.
- `garuda/adapters`: Base classes and interfaces for users to implement their target app wrappers.
- `garuda/runners`: `pytest-bdd` wrappers, Allure hooks, and Cucumber JSON generation.
