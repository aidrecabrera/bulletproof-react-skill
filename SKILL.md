---
name: bulletproof-react-architecture
description: Apply bulletproof-react conventions when writing, organizing, or reviewing React or TypeScript application code. Covers feature-based folder structure, API layer patterns, state management categories, testing strategy, error handling, authentication and authorization, and performance optimization. Use whenever the user creates a new React feature, sets up a project structure, decides where a piece of code or state should live, writes data-fetching or auth code, or asks for a review of React code against a scalable architecture.
---

# Bulletproof React architecture

Source: [bulletproof-react](https://github.com/alan2207/bulletproof-react) by Alan Alickovic, MIT licensed. 35k+ stars, actively maintained. This skill is a working reference distilled from that repo, not a copy of it. See [NOTICE.md](NOTICE.md) for the license and adaptation note.

## Vocabulary

Use these terms exactly. Don't substitute "module," "service," or "component" where the repo means "feature."

- **Feature**: a self-contained folder under `src/features/`, holding everything specific to one piece of product functionality: its own `api/`, `components/`, `hooks/`, `stores/`, `types/`.
- **Shared code**: anything in the top-level `components/`, `hooks/`, `lib/`, `types/`, `utils/`. Usable from any feature or from the app layer. Never depends on a feature.
- **App layer**: `src/app/`, meaning routes, root component, providers, router config. The only layer allowed to compose multiple features together.

## Rules of thumb

**No cross-feature imports.** `features/comments` importing from `features/discussions` is always a bug, not a shortcut. If two features need the same logic, promote it to shared code.

**Code flows one direction: shared → features → app.** Never the reverse. If you find yourself importing from `app/` into a feature, or from one feature into another, that's the rule breaking, not an exception to it.

**Match the state to its category before reaching for a store.** Component, application, server cache, form, and URL state are five different problems with five different right answers. Mixing them into one `useState` object is the most common cause of unrelated re-renders.

## Before you apply this

This structure earns its cost on an app with real, growing feature surface area, several people touching the codebase, or a plan to still be maintained in a year. It does not earn its cost on a five-file prototype, a one-off script, or a component being built to answer a design question and then thrown away.

Check scale before reaching for structure:

- **One feature, a handful of files?** Skip `src/features/` entirely. A flat `src/` with a few files in it is not a violation of this skill, it's the correct application of it. Creating the full feature-folder skeleton (`api/`, `components/`, `hooks/`, `stores/`, `types/`) for a feature that has one component and one fetch call is the failure mode this skill exists to prevent, not the goal.
- **Already have a working pattern in this codebase?** Match it instead of introducing bulletproof-react's pattern alongside it. Two competing conventions in one repo cost more than either convention's imperfections.
- **Unsure whether a rule applies at this size?** Ask what breaks without it. If nothing breaks, skip it and say so in one line rather than applying it preemptively.

None of the reference files below are a checklist to run top to bottom on every task. Each one solves a specific problem. Read the one that matches the problem in front of you; skip the rest.

## When this skill doesn't apply

Skip this skill entirely for a task that isn't structural: fixing a bug in existing code, writing a single function, styling a component, or anything where the question isn't "where does this go" or "how should this be organized." Reaching for feature-folder conventions on a request to fix a typo is the same mistake as reaching for a design pattern to solve a one-line problem.

This skill is also weaker on unfamiliar or unusually small models. It assumes the model reading it will apply judgment about scale rather than mechanically creating every folder listed in project-structure.md because the folder is listed. If you notice a pattern from this skill being applied somewhere it clearly doesn't fit, that's a signal to stop and ask, not a signal to keep going because the skill said so.

## These resources may help

Each reference file below is short enough to load in full. Read the one relevant to the current task, not all of them.

- **[reference/project-structure.md](reference/project-structure.md)**: folder layout for the app and for a feature, the ESLint rules that enforce the import direction above. Start here for any new feature, or any question about where a file belongs. This is the most-read file in the skill because "where does this go" is the most common question it answers.

- **[reference/api-layer.md](reference/api-layer.md)**: how to structure a request declaration: types, fetcher, hook. Read this when writing or reviewing any data-fetching code, not just when setting up a new feature.

- **[reference/state-management.md](reference/state-management.md)**: the five state categories from the rule of thumb above, with the library recommendation for each. Read this before adding a `useState`, a context, or a store, since the category determines the right tool, and picking the tool first usually means picking it wrong.

- **[reference/testing.md](reference/testing.md)**: test types and where to weight effort, plus the specific tooling (Vitest, Testing Library, Playwright, MSW). Read this when setting up a test suite or deciding what kind of test a change actually needs.

- **[reference/error-handling.md](reference/error-handling.md)**: API error interception, error boundary placement, production error tracking. Short file, read it in full rather than skimming.

- **[reference/security.md](reference/security.md)**: authentication token storage, authorization models (RBAC and PBAC). Read this in full, not selectively. This is the one file in the skill where skipping a section is more likely to introduce a real vulnerability than to save time, since the storage and sanitization guidance depend on each other.

- **[reference/performance.md](reference/performance.md)**: code splitting, re-render causes, the `children` pattern, image and prefetching optimizations. Read this when something is measurably slow, not preemptively. Most of what's in this file is a fix for a specific symptom, not a checklist to apply blind.

- **[reference/overengineering-check.md](reference/overengineering-check.md)**: a narrow review pass that only looks for this skill's own conventions applied somewhere they don't earn their cost, empty feature subfolders, stores for state that never leaves one component, ESLint rules copied onto a two-feature app. Read this when reviewing a codebase for unnecessary structure, or as a self-check right after you've generated a feature scaffold. It doesn't check correctness or security, only whether the structure itself is pulling its weight.
