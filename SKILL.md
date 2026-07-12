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

This structure earns its cost on an app with growing feature surface area, several people touching the codebase, or a plan to still be maintained in a year. It does not earn its cost on a five-file prototype, a one-off script, or a throwaway component.

Check scale before reaching for structure:

- **One feature, a handful of files?** Skip `src/features/` entirely. A flat `src/` is not a violation here, it's the correct call. Building the full feature-folder skeleton for one component and one fetch call is the failure mode this skill exists to prevent, not the goal.
- **Already have a working pattern in this codebase?** Match it instead of introducing bulletproof-react's pattern alongside it. Two competing conventions cost more than either convention's imperfections.
- **Unsure whether a rule applies at this size?** Ask what breaks without it. If nothing breaks, skip it and say so in one line.

The reference files below are not a checklist to run top to bottom. Each solves one problem. Read the one that matches the problem in front of you, skip the rest.

## When this skill doesn't apply

Skip it entirely for non-structural work: fixing a bug, writing a single function, styling a component, anything where the question isn't "where does this go" or "how should this be organized."

This skill also assumes the model applies judgment about scale rather than mechanically building every folder because the folder is listed. If a pattern from this skill is being applied somewhere it clearly doesn't fit, stop and ask rather than continuing because the skill said so.

## These resources may help

Each file is short enough to load in full. Read the one relevant to the current task.

- **[reference/project-structure.md](reference/project-structure.md)**: app and feature folder layout, the ESLint rules enforcing the import direction above. Start here for a new feature or any "where does this go" question. Most-read file in the skill.

- **[reference/api-layer.md](reference/api-layer.md)**: request declaration structure: types, fetcher, hook. Read when writing or reviewing data-fetching code.

- **[reference/state-management.md](reference/state-management.md)**: the five state categories, with a library recommendation for each. Read before adding a `useState`, context, or store. Picking the tool before the category usually means picking wrong.

- **[reference/testing.md](reference/testing.md)**: test types, where to weight effort, tooling (Vitest, Testing Library, Playwright, MSW).

- **[reference/error-handling.md](reference/error-handling.md)**: API error interception, error boundary placement, production error tracking. Short, read in full.

- **[reference/security.md](reference/security.md)**: token storage, authorization models (RBAC, PBAC). Read in full, not selectively. Storage and sanitization guidance depend on each other; skipping a section here risks a real vulnerability, not just wasted time.

- **[reference/performance.md](reference/performance.md)**: code splitting, re-render causes, the `children` pattern, image and prefetch optimizations. Read when something is measurably slow, not preemptively.

- **[reference/overengineering-check.md](reference/overengineering-check.md)**: a narrow review pass for this skill's own conventions applied where they don't earn their cost, empty feature subfolders, stores for state that never leaves one component, ESLint rules on a two-feature app. Read when reviewing a codebase for unnecessary structure, or as a self-check after generating a feature scaffold. Doesn't check correctness or security.
