<!-- GENERATED FILE. Do not edit directly. -->
<!-- Source: SKILL.md and reference/*.md in this skill's directory. -->
<!-- Regenerate with: python scripts/build_agents_md.py -->

# Bulletproof React architecture

Conventions adapted from [bulletproof-react](https://github.com/alan2207/bulletproof-react), MIT licensed.

# Project structure

## Top-level layout

Most code lives under `src/`:

```
src/
├── app/            # routes, root app component, providers, router config
├── assets/         # static files: images, fonts
├── components/     # shared components used across the whole app
├── config/         # global config, exported env variables
├── features/       # feature-based modules, most code lives here
├── hooks/          # shared hooks
├── lib/            # reusable libraries, preconfigured for the app
├── stores/         # global state stores
├── testing/        # test utilities and mocks
├── types/          # shared types
└── utils/          # shared utility functions
```

## Feature folder layout

Each feature gets its own folder under `src/features/`:

```
src/features/awesome-feature/
├── api/            # API request declarations and hooks for this feature
├── assets/         # static files scoped to this feature
├── components/     # components scoped to this feature
├── hooks/          # hooks scoped to this feature
├── stores/         # state stores scoped to this feature
├── types/          # types used within this feature
└── utils/          # utility functions for this feature
```

Only create the subfolders a feature actually needs. A small feature might just have `api/` and `components/`.

If a lot of API calls are shared across features, it's fine to keep a dedicated top-level `api/` folder outside of `features/` instead of duplicating fetchers per feature.

## Import rules

**No cross-feature imports.** `features/discussions` should never import from `features/comments`. If two features need the same logic, pull it into a shared module (`components/`, `hooks/`, `lib/`, `utils/`) or compose the features together at the app layer, not by reaching into each other.

**Imports flow one direction: shared → features → app.** Shared code can be used anywhere. Features can use shared code but not other features. The app layer can use features and shared code. Nothing shared should ever import from a feature or from `app/`.

Enforce both rules with ESLint's `import/no-restricted-paths`:

```js
'import/no-restricted-paths': [
  'error',
  {
    zones: [
      // no cross-feature imports
      {
        target: './src/features/auth',
        from: './src/features',
        except: ['./auth'],
      },
      {
        target: './src/features/comments',
        from: './src/features',
        except: ['./comments'],
      },
      // repeat per feature

      // unidirectional flow: app can import from features, not the reverse
      {
        target: './src/features',
        from: './src/app',
      },
      // features and app can import shared modules, not the reverse
      {
        target: [
          './src/components',
          './src/hooks',
          './src/lib',
          './src/types',
          './src/utils',
        ],
        from: ['./src/features', './src/app'],
      },
    ],
  },
],
```

## Avoid barrel files

Skip barrel files (an `index.ts` that re-exports everything from a feature). They break tree shaking in bundlers like Vite and can cause real performance regressions. Import directly from the specific file instead.

This structure applies equally well to Next.js, Remix, and React Native apps. Only the `app/` folder's internals tend to differ based on the meta-framework in use.

# API layer

## One API client instance

Create a single, preconfigured API client and reuse it everywhere, rather than constructing a new client per call. Use `fetch` directly or a library like axios, graphql-request, or apollo-client, configured once in `lib/api-client.ts` or similar.

## Declare requests separately, don't inline them

Define and export each API request as its own module instead of writing fetch calls inline wherever they're needed. This keeps every available endpoint discoverable in one place and makes typing the response, and inferring types downstream, straightforward.

Each request declaration should have three parts:

1. **Types and validation schemas** for the request and response
2. **A fetcher function** that calls the endpoint using the shared API client
3. **A hook** that wraps the fetcher with a data-fetching library (react-query, swr, apollo-client, urql) to handle caching, loading state, and refetching

Put these in the feature's `api/` folder, one file per endpoint or logical group: `get-discussions.ts`, `create-discussion.ts`, and so on.

# State management

Before reaching for a global store, figure out which of these five categories the state actually belongs to. Most bugs and unnecessary re-renders come from putting state one level higher than it needs to be.

## Component state

State used by one component and its children. Start here by default; only lift state higher if something outside the component tree actually needs it.

- `useState` for independent, simple values
- `useReducer` when a single action needs to update several related pieces of state at once

## Application state

Global, cross-cutting state: modal visibility, notifications, color mode. Keep this category as small as possible. Don't globalize state just because it's convenient; localize it to the components that need it whenever you can.

Reasonable options: React context + hooks, Redux + Redux Toolkit, MobX, Zustand, Jotai, XState.

## Server cache state

Data that originated on the server and is cached client-side. Don't put this in a general-purpose store like Redux. Use a library built for it: react-query, swr, apollo-client, urql, or RTK Query. These handle caching, invalidation, and refetching in ways a plain store doesn't.

## Form state

Use a form library rather than hand-rolling validation and field state: React Hook Form, Formik, or React Final Form. Wrap the library in a small `Form` component plus reusable field components so the rest of the app doesn't touch the library's API directly.

Pair the form library with a schema validator for client-side validation: zod or yup.

## URL state

Data that lives in the URL itself, route params or query params. Use the router (react-router-dom or equivalent) to read and update it rather than mirroring it into component or application state. The URL is the source of truth here, don't duplicate it elsewhere.

# Testing

Weight testing effort toward integration tests. Unit tests catch isolated bugs, but a suite of passing unit tests doesn't guarantee the pieces work together, and that's usually where real breakage happens.

## Test types

Unit tests cover the smallest scope: one function or component in isolation. They're good for shared components and utilities used throughout the app, and for pinning down complex logic inside a single component. They're fast and cheap, so write them freely.

Integration tests check how multiple parts work together, and this is where most testing effort should go. A suite of passing unit tests can still hide a broken integration: the connections between components, hooks, and API calls are exactly what unit tests can't see.

End-to-end tests exercise the full app, frontend and backend together, driven the way a real user would drive it. Write fewer of these since they're expensive to write and run, but treat them as the strongest signal that the app actually works.

## Tooling

Vitest is the test runner. It has a Jest-compatible API but is faster and integrates better with modern build tools.

Testing Library tests what the user sees and does, not internal state. If you swap out a state management library, tests written against rendered output shouldn't need to change, because the output itself didn't change.

Playwright handles end-to-end tests. Run it in browser mode locally when debugging, since you get a visual step-through, or headless in CI.

MSW (Mock Service Worker) intercepts real HTTP calls and returns mocked responses from handlers you define. It's useful for developing against an API that isn't finished yet, and for tests: you make real HTTP calls, MSW answers them, and you're not manually mocking `fetch`.

# Error handling

## API errors

Handle them in one place: an interceptor on the API client. Use it to surface error toasts, log out a user whose session expired, or trigger a token refresh. Don't scatter error handling across every call site.

## In-app errors

Use multiple error boundaries, not one boundary wrapping the entire app. A single top-level boundary means any error anywhere takes down the whole UI. Placing boundaries around independent sections lets a failure in one part stay contained there while the rest of the app keeps working.

## Error tracking

Track production errors with a dedicated service (Sentry or similar) rather than building your own. You get the browser, platform, and user context around each error for free, and you need source maps uploaded to see where the error actually happened in your original code, not in minified output.

# Security: authentication and authorization

Client-side auth improves the user experience, but it is not a substitute for server-side enforcement. Every resource still needs to be protected on the server regardless of what the client does. Treat everything below as complementary to that, not a replacement for it.

## Authentication

Verifies who the user is. The standard pattern for single-page apps is JWT: the user logs in, receives a token, and the token is sent with every subsequent authenticated request.

**Where to store the token matters and the tradeoffs are not optional to consider:**

- Storing the token only in application state is the most secure option against theft, but it resets on page refresh, so on its own it's not practical.
- `localStorage` and `sessionStorage` are readable by any JavaScript running on the page. If the app has an XSS vulnerability, the token can be stolen directly. Don't store tokens here unless you've deliberately decided the tradeoff is acceptable for the app in question.
- Cookies with the `HttpOnly` flag are the safer default. `HttpOnly` cookies aren't accessible to client-side JavaScript at all, which closes off the XSS token-theft path.

Regardless of storage choice, sanitize all user input before rendering it. This is the other half of XSS defense, storage security alone doesn't help if unsanitized input gets rendered back into the page. See the OWASP client-side security top 10 for the fuller threat list.

Treat the authenticated user object as global state, available anywhere in the app. If already using react-query, the react-query-auth library handles this pattern directly. Otherwise, react context + hooks or another state library works. The convention: the app considers a user authenticated if and only if a user object is present.

## Authorization

Verifies what an authenticated user is allowed to do. Two models, and they're not mutually exclusive:

**Role-based (RBAC):** Define roles (`USER`, `ADMIN`, and so on) and grant each role a set of permissions. Simple to reason about, and it's the right default for most access decisions in an app.

**Permission-based (PBAC):** Finer-grained than roles. Use it when access depends on something more specific than role membership, most commonly ownership: only the author of a comment can delete that comment, regardless of what role they hold. RBAC alone can't express that; PBAC checks a policy specific to the resource instance.

In practice, gate UI and actions with a component that accepts either allowed roles (RBAC) or a policy check function (PBAC), and use whichever fits the specific permission being enforced.

# Performance

## Code splitting

Split at the route level so the initial load only fetches what's needed for the first screen, and everything else loads lazily as the user navigates. Don't split more granularly than that by default: too many small chunks means too many requests, which can hurt performance instead of helping it.

## State and re-renders

- Don't put everything in one global state object. A single state blob means any update anywhere triggers re-renders everywhere. Split state by where it's actually used.
- Keep state as close as possible to the components that use it. State lifted higher than necessary causes components in between to re-render even though they don't care about the value.
- If a piece of state is initialized from an expensive computation, pass the initializer function to `useState`, not the computed value:

  ```javascript
  // runs the expensive function on every re-render
  const [state, setState] = useState(myExpensiveFn());

  // runs it once, on mount
  const [state, setState] = useState(() => myExpensiveFn());
  ```

- For state that tracks many independent elements at once, consider a store with atomic updates (Jotai) instead of one big object.
- React Context is fine for low-frequency data: theme, user info, small local state. For data that updates often, plain context will re-render every consumer on every change. Either use a library with built-in selectors (Zustand, Jotai) or `use-context-selector` if staying with context. And before reaching for context at all, check whether lifting state up or better component composition solves the problem without it. Context is not the default answer to prop drilling.
- If the app updates frequently and runtime styling libraries (emotion, styled-components) show up as a bottleneck, consider a zero-runtime alternative (Tailwind, vanilla-extract, CSS modules) that generates styles at build time instead.

## Use `children` to avoid re-renders

Passing JSX through the `children` prop is the cheapest optimization available. Content passed as `children` is a VDOM structure the parent doesn't own and can't re-render, so it's untouched by state changes in the parent.

```javascript
// PureComponent re-renders every time count changes
const Counter = () => {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>count is {count}</button>
      <PureComponent />
    </div>
  );
};

// PureComponent is passed as children, so it's isolated from count updates
const Counter = ({ children }) => {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>count is {count}</button>
      {children}
    </div>
  );
};
```

## Images

Lazy-load images outside the viewport. Use WebP where possible. Use `srcset` so the browser picks the right image size for the screen instead of always loading the largest.

## Web vitals

Check Lighthouse and PageSpeed Insights scores. Search ranking takes web vitals into account, so this isn't purely a UX concern.

## Data prefetching

If you can predict where the user is headed next, prefetch that data with `queryClient.prefetchQuery` (react-query) before they navigate. The data is already cached by the time the page mounts, so there's no loading state to show.

# Checking for over-applied structure

Scope: this file only looks for bulletproof-react conventions applied somewhere they don't earn their cost. It does not check correctness, security, or performance. Those are covered in the other reference files.

Read this when asked to review a React codebase for over-engineering, unnecessary structure, or premature abstraction, or when you've just generated a feature scaffold yourself and want to check it before handing it over.

## What to look for

Walk the `src/features/` tree and check each feature folder against what it actually contains, not what the pattern says a feature folder should have.

- **Empty or near-empty subfolders.** A feature with a `hooks/` folder holding one trivial hook that's only used in one place didn't need the folder. Flat is fine until a second hook shows up.
- **A `stores/` folder for state that never leaves one component.** That's component state, not application state. Creating a store for it is applying the state-management categories from [reference/state-management.md](reference/state-management.md) without checking which category actually applies.
- **A feature folder for something that isn't a feature.** A single reusable button living under `features/checkout/components/` because it was built during checkout work, when it's actually generic UI, belongs in shared `components/` instead.
- **ESLint import-restriction rules copied wholesale for a two-feature app.** The rule from [reference/project-structure.md](reference/project-structure.md) exists to stop cross-feature imports from accumulating unnoticed across a large team. On a two-feature solo project, the enforcement overhead may not be worth it yet. Note it as a candidate to add later, not a mandatory day-one setup.
- **A full request-declaration file (types, fetcher, hook) for one API call that's used once and never will be reused.** [reference/api-layer.md](reference/api-layer.md)'s pattern pays off when endpoints multiply and get reused. For a single one-off call, a plain fetch inline is not a violation.

## Reporting format

One line per finding: location, what's over-applied, what it should be instead.

```
features/notifications/stores/notification-store.ts: unnecessary store, state only used in NotificationBell. Move to local useState.
features/checkout/components/icon-button.tsx: not checkout-specific, move to shared components/.
```

Don't write paragraphs explaining why each finding matters, the reference file it points back to already explains the rule. Just say what to cut and where it goes instead.

If nothing here is over-applied, say so in one line. A clean scan is a valid outcome, not a reason to invent findings.
