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
- **Extensibility:** The engine must be provider-agnostic, supporting Cloud APIs (OpenAI, Anthropic) and Local Models (Ollama, vLLM) interchangeably via a unified interface (e.g., LiteLLM or LangChain).
- **Evaluation Logic:** The framework will adopt **DeepEval's structure** for RAG evaluation metrics (e.g., Contextual Precision, Contextual Recall, Faithfulness, Answer Relevance).

## 3. Target Application Interface (Adapter Layer)
**Objective:** How Garuda AI connects to the systems it is testing.
- **Hybrid Connectivity:** The framework must support two modes of interaction with the target application:
  1. **HTTP/REST API Adapter:** For testing deployed services (e.g., sending JSON payloads to a `/chat` endpoint).
  2. **Direct Python SDK Adapter:** For testing Python functions/classes directly in memory without network overhead (useful for unit/integration testing in CI).

## 4. Reporting & CI/CD Component
**Objective:** Generate actionable, enterprise-grade reports that plug directly into CI/CD pipelines.
- **Test Runner:** The framework will utilize `pytest` under the hood.
- **Allure Integration:** Test runs must generate **Allure Reports** to provide rich, visual dashboards for stakeholders.
- **Cucumber JSON:** Execution must output **Cucumber JSON** artifacts (via `pytest-bdd`) to integrate with broader enterprise test management tools.
- **Pipeline Native:** Exit codes must cleanly fail pipelines (Exit Code 1) if safety guardrails or core functional tests fail.

## 5. Component Architecture Summary
To support the above, the codebase will be structured into the following distinct modules:
- `garuda/config`: YAML/JSON parsers and Excel data loaders.
- `garuda/evaluators`: Groq-powered Judge agents and DeepEval-style metric implementations.
- `garuda/adapters`: HTTP and Python SDK clients for target communication.
- `garuda/runners`: `pytest-bdd` wrappers and Allure/Cucumber reporting hooks.
