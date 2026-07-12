<!-- GENERATED FILE. Do not edit directly. -->
<!-- Source: SKILL.md and reference/*.md in this skill's directory. -->
<!-- Regenerate with: python scripts/build_agents_md.py -->

# Bulletproof React architecture

Conventions adapted from [bulletproof-react](https://github.com/alan2207/bulletproof-react), MIT licensed.

# Project structure

The upstream repo is a monorepo with three app variants: `apps/react-vite/`, `apps/nextjs-app/`, `apps/nextjs-pages/`. Everything below follows the `react-vite` version.

## Contents

- [Top-level layout](#top-level-layout)
- [Config patterns](#config-patterns)
- [Feature folder layout](#feature-folder-layout)
- [Import rules](#import-rules)
- [Additional ESLint rules](#additional-eslint-rules)
- [Avoid barrel files](#avoid-barrel-files)

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
├── testing/        # test utilities and mocks
├── types/          # shared types
└── utils/          # shared utility functions
```

(The original docs also list `stores/` for global state, but the current codebase keeps stores colocated in features or uses Zustand ad-hoc without a top-level stores directory.)

### Config patterns

The `config/` folder has two files worth noting:

**`env.ts`** uses Zod to validate environment variables at runtime. It reads from `import.meta.env`, strips the `VITE_APP_` prefix, validates against a schema, and throws a detailed error if validation fails. This catches misconfigured deployments before they cause silent failures.

```ts
const envSchema = z.object({
  API_URL: z.string().url(),
  APP_URL: z.string().url().optional(),
  ENABLE_API_MOCKING: z.boolean().default(false),
});
```

**`paths.ts`** defines route paths centrally with a `path` and `getHref()` for each route. This keeps navigation type-safe and avoids hardcoded path strings scattered across components.

```
src/config/
├── env.ts
└── paths.ts
```

## Feature folder layout

Each feature under `src/features/` gets its own folder. Only create the subdirectories a feature actually needs.

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

A small feature might just have `api/` and `components/`. Add folders when they earn their keep.

If a lot of API calls are shared across features, keep a dedicated top-level `api/` folder instead of duplicating fetchers per feature.

## Import rules

**No cross-feature imports.** `features/discussions` should never import from `features/comments`. Pull shared logic into `components/`, `hooks/`, `lib/`, or `utils/`. Compose features at the app layer, not by reaching into each other.

**Imports flow one direction: shared → features → app.** Shared code can be used anywhere. Features can use shared code but not other features. The app layer can use features and shared code. Nothing shared imports from a feature or from `app/`.

Enforce with ESLint's `import/no-restricted-paths`:

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

      // unidirectional flow: app can import from features
      {
        target: './src/features',
        from: './src/app',
      },
      // features and app can import shared modules
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

## Additional ESLint rules

The repo also uses these. Not all are mandatory on day one, but add them as the codebase grows:

- **`import/no-cycle`.** Prevents circular dependencies. Worth adding early, circular deps are hard to untangle later.
- **`import/order`.** Enforces consistent import grouping and alphabetical ordering. Optional but reduces noise in diffs.
- **`check-file/filename-naming-convention`** and **`check-file/folder-naming-convention`.** Enforces `KEBAB_CASE` for all files and folders. Use from the start if you care about naming consistency; skip it if you're prototyping.

## Avoid barrel files

Don't use barrel files (an `index.ts` that re-exports everything from a feature). They break tree shaking in Vite. Import directly from the specific file.

This structure applies to Next.js, Remix, and React Native. Only the `app/` folder's internals differ based on the meta-framework.

# API layer

## One API client instance

Create a single preconfigured API client and reuse it everywhere. The upstream repo uses an Axios instance in `lib/api-client.ts` with an `authRequestInterceptor` that sets the Accept header, enables credentials, and extracts `response.data`. On errors, it shows a toast and redirects to login on 401.

Don't construct a new client per call. One instance, configured once.

## Declare requests separately, don't inline them

Each endpoint gets its own file in the feature's `api/` folder. This makes every endpoint discoverable and keeps typing straightforward.

Each file has three parts:

1. **Types and validation schemas** for the request and response
2. **A fetcher function** that calls the endpoint using the shared API client
3. **A hook or `queryOptions`** that wraps the fetcher for caching, loading state, and refetching

```
features/discussions/api/
├── get-discussions.ts
└── create-discussion.ts
```

## Use `queryOptions` for TanStack Query v5

Instead of writing raw `useQuery` calls inline, export a `queryOptions` object. This lets you reuse the same query configuration across `useQuery`, `prefetchQuery`, and `queryClient.invalidateQueries` with full type inference.

```ts
export const getDiscussionsQueryOptions = ({ page }) => {
  return queryOptions({
    queryKey: page ? ['discussions', { page }] : ['discussions'],
    queryFn: () => getDiscussions(page),
  });
};
```

Then consume it:

```ts
const { data } = useQuery(getDiscussionsQueryOptions({ page }));
```

Or prefetch before the user navigates:

```ts
queryClient.prefetchQuery(getDiscussionsQueryOptions({ page: 1 }));
```

## Auth pattern with react-query-auth

The upstream repo uses `react-query-auth` to wire up authentication. In `lib/auth.tsx`, call `configureAuth` once with your `getUser`, `login`, `register`, and `logout` functions. Export the hooks you need:

```ts
const { useUser, useLogin, useLogout, useRegister, AuthLoader, ProtectedRoute } =
  configureAuth({
    userFn: getProfile,
    loginFn: loginWithEmailAndPassword,
    registerFn: registerWithEmailAndPassword,
    logoutFn: logout,
  });
```

This gives you typed hooks for all auth operations. `AuthLoader` renders a spinner while the user object loads. `ProtectedRoute` redirects unauthenticated users to the login page.

The same pattern works for any backend. The login and register functions are just async calls that return a user object and set the token.

# State management

Before reaching for a global store, figure out which of these five categories the state belongs to. Most bugs and unnecessary re-renders come from putting state one level higher than it needs to be.

## Component state

State used by one component and its children. Start here. Only lift state higher if something outside the component tree needs it.

- `useState` for simple values
- `useReducer` when one action updates several related pieces of state at once

## Application state

Global, cross-cutting state: modal visibility, notifications, color mode. Keep this category small. Don't globalize state just because it's convenient.

The upstream repo uses **Zustand**. It's a good default: simple API, no boilerplate, no provider wrapping, built-in selectors to avoid unnecessary re-renders. Alternatives exist (Redux Toolkit, Jotai, MobX, context + hooks) but Zustand covers most needs with less code.

Usage in the repo: `useNotifications.getState()` in the API client interceptor to surface error toasts. No store directory at the top level. Stores are colocated in features or used ad-hoc via Zustand.

## Server cache state

Data that originated on the server and is cached client-side. Don't put this in a general-purpose store like Zustand or Redux. Use TanStack Query, SWR, Apollo, or RTK Query. These handle caching, invalidation, and refetching in ways a plain store doesn't.

The upstream repo uses **TanStack Query v5** with the `queryOptions` pattern (see [api-layer.md](api-layer.md)). Stale time is set to 60 seconds by default, no refetch on window focus, no retry.

## Form state

Use a form library: React Hook Form, Formik, or React Final Form. The upstream repo uses **React Hook Form** with **Zod** for validation. Wrap the library in a small `Form` component plus reusable field components so the rest of the app doesn't touch the library's API directly.

## URL state

Data in the URL: route params, query params. Use the router to read and update it. Don't mirror it into component or application state. The URL is the source of truth.

The upstream repo uses **react-router v7** (not `react-router-dom`). Route paths are defined centrally in `config/paths.ts` with type-safe `path` and `getHref()` helpers.

# Testing

Weight testing effort toward integration tests. Unit tests catch isolated bugs, but they don't tell you the pieces work together. That's where real breakage happens.

## Test types

**Unit tests.** One function or component in isolation. Good for shared components, utilities, and complex logic inside a single component. Fast and cheap, write them freely.

**Integration tests.** How multiple parts work together. This is where most testing effort should go. Unit tests can't see broken connections between components, hooks, and API calls.

**End-to-end tests.** Full app, frontend and backend, driven like a real user. Expensive to write and run. Write fewer of these, but treat them as the strongest signal.

## Tooling

**Vitest.** Test runner. Jest-compatible API, faster, better build tool integration.

**Testing Library.** Tests what the user sees and does, not internal state. Swap out a state management library, and tests against rendered output shouldn't change.

**Playwright.** E2E tests. Run in browser mode locally for visual debugging, headless in CI.

**MSW.** Intercepts real HTTP calls and returns mocked responses. Useful when the API isn't finished. In tests, you make real HTTP calls, MSW answers them, and you avoid mocking `fetch`.

## Upstream repo patterns

### renderApp helper

The test utility exports `renderApp` which creates a memory router, wraps in the app provider, and optionally authenticates a user.

```ts
import { renderApp, createUser } from '@/testing/test-utils';

it('displays the discussion list', async () => {
  const user = await createUser();
  const { findByText } = renderApp(<DiscussionsPage />, { user });
  expect(await findByText('My Discussion')).toBeInTheDocument();
});
```

This sets up everything: router, providers, auth, data loading. You don't rewire these per test.

### In-memory database with MSW data

`@mswjs/data` creates an in-memory database with typed models. Combined with MSW, API calls during tests hit this database instead of a real server.

```ts
// db.ts
import { factory, primaryKey } from '@mswjs/data';
import { nanoid } from '@ngneat/falso';

const models = {
  user: {
    id: primaryKey(nanoid),
    firstName: String,
    lastName: String,
    email: String,
    role: String,
  },
  discussion: {
    id: primaryKey(nanoid),
    title: String,
    body: String,
    authorId: String,
  },
};

export const db = factory(models);
```

Write to it directly in tests via `db.user.create(...)` / `db.discussion.create(...)`.

### Data generators

`@ngneat/falso` generates realistic fake data. Combine with the in-memory DB for seeded test scenarios.

```ts
import { randCompanyName, randUserName, randEmail, randParagraph, randUuid } from '@ngneat/falso';

export const generateUser = () => ({
  id: randUuid(),
  firstName: randUserName(),
  lastName: randUserName(),
  email: randEmail(),
});
```

Note: the data generator returns plain objects. The `db.user.create(...)` call that inserts into the database happens in the test utilities or setup, not in the generator itself.

### Standalone mock server for E2E

The repo includes a standalone mock server using `vite-node` and `@mswjs/http-middleware`. It runs the same MSW handlers as the test suite but as an HTTP server, so Playwright E2E tests hit a real API without standing up a backend.

# Error handling

## API errors

Handle them in one place: an interceptor on the API client. Use it to surface error toasts, log out a user whose session expired, or trigger a token refresh. Don't scatter error handling across every call site.

## In-app errors

Use multiple error boundaries, not one boundary wrapping the entire app. A single top-level boundary means any error anywhere takes down the whole UI. Placing boundaries around independent sections lets a failure in one part stay contained there while the rest of the app keeps working.

## Error tracking

Track production errors with a dedicated service (Sentry or similar) rather than building your own. You get the browser, platform, and user context around each error for free, and you need source maps uploaded to see where the error actually happened in your original code, not in minified output.

# Security: authentication and authorization

Client-side auth improves the user experience but doesn't replace server-side enforcement. Every resource still needs to be protected on the server. What follows is how to wire up the client side; it's complementary, not a replacement.

## Authentication

Standard pattern for SPAs is JWT: the user logs in, receives a token, and the token is sent with every authenticated request.

**Where to store the token matters:**

- **Application state only.** Most secure against theft, but resets on page refresh. Not practical on its own.
- **`localStorage` / `sessionStorage`.** Readable by any JavaScript on the page. If the app has an XSS vulnerability, the token can be stolen directly.
- **HttpOnly cookies.** Safer default. Not accessible to client-side JavaScript, which closes the XSS token-theft path.

Sanitize all user input before rendering it. That's the other half of XSS defense. Storage security alone doesn't help if unsanitized input gets rendered back into the page.

The upstream repo uses **react-query-auth** to wire up auth. In `lib/auth.tsx`, call `configureAuth` with your API functions, then export typed hooks (see [api-layer.md](api-layer.md) for the pattern). The app considers a user authenticated if and only if a user object is present.

## Authorization

Two models. You can use both in the same app.

**Role-based (RBAC):** Define roles (`USER`, `ADMIN`) and grant each role a set of permissions. Simple. Right default for most access decisions.

**Permission-based (PBAC):** Finer-grained. Use it when access depends on something more specific than role membership, most commonly ownership: only the author of a comment can delete it. RBAC alone can't express that.

### The Authorization component pattern

The upstream repo has `lib/authorization.tsx` with a reusable component. It takes either `allowedRoles` (RBAC) or a `policyCheck` function (PBAC) and renders children only when access is granted.

```tsx
// RBAC. Only admins can see this
<Authorization allowedRoles={['ADMIN']}>
  <DeleteUserButton />
</Authorization>

// PBAC. Only the comment author can delete
// Evaluate the policy and pass the boolean result
<Authorization policyCheck={POLICIES['comment:delete'](user, comment)}>
  <DeleteCommentButton />
</Authorization>
```

Define roles and policies in one place:

```ts
export enum ROLES {
  ADMIN = 'ADMIN',
  USER = 'USER',
}

const POLICIES = {
  'comment:delete': (user: User, comment: Comment) => {
    if (user.role === ROLES.ADMIN) return true;
    if (user.role === ROLES.USER && comment.author?.id === user.id) return true;
    return false;
  },
} as const;
```

The `useAuthorization` hook returns `checkAccess` and `role` so you can use them in logic too, not just in JSX.

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

- For state that tracks many independent elements at once, consider a store with atomic updates (Zustand, Jotai) instead of one big object.
- React Context is fine for low-frequency data: theme, user info, small local state. For data that updates often, plain context will re-render every consumer on every change. Either use a library with built-in selectors (Zustand, Jotai) or `use-context-selector` if staying with context. And before reaching for context at all, check whether lifting state up or better component composition solves the problem without it. Context is not the default answer to prop drilling.
- If the app updates frequently and runtime styling libraries (emotion, styled-components) show up as a bottleneck, consider a zero-runtime alternative (Tailwind, vanilla-extract, CSS modules) that generates styles at build time instead.

## Use `children` to avoid re-renders

Passing JSX through `children` is the cheapest optimization. Content passed as `children` is a VDOM structure the parent doesn't own and can't re-render. State changes in the parent leave it untouched.

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

# Quality check tools

These tools and companion skills enforce or extend the conventions in this skill. Use them when the task goes beyond organizing files: when you need to verify code quality, catch performance issues, or design better component APIs.

## react-doctor

Scans your React codebase for issues across state and effects, performance, architecture, security, and accessibility. Run it after applying this skill's conventions to catch what the agent missed.

```bash
npx react-doctor@latest
```

Works with Next.js, Vite, TanStack, React Native, Expo. Can run in CI on pull requests. 13.5k stars.

Notable: react-doctor catches exactly the kinds of problems this skill teaches you to avoid: state in the wrong place, barrel files breaking tree shaking, missing error boundaries. The two tools complement each other.

[https://github.com/millionco/react-doctor](https://github.com/millionco/react-doctor)

## Vercel react-best-practices

70 rules across 8 categories, prioritized by impact: eliminating waterfalls, bundle size, server-side performance, client-side data fetching, re-render optimization, rendering performance, JavaScript performance, and advanced patterns.

Load this skill when performance is the specific concern, not just "where do I put these files" but "why is this page slow and how do I fix it."

```bash
npx skills add vercel-labs/agent-skills --skill react-best-practices
```

[https://github.com/vercel-labs/agent-skills/tree/main/skills/react-best-practices](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-best-practices)

## Vercel composition-patterns

Component architecture patterns: compound components with shared context, avoiding boolean prop proliferation, lifting state into providers, and React 19 API changes (no more `forwardRef`, `use()` instead of `useContext()`).

Load this skill when designing component APIs, not file organization but the contract between a component and its callers.

```bash
npx skills add vercel-labs/agent-skills --skill composition-patterns
```

[https://github.com/vercel-labs/agent-skills/tree/main/skills/composition-patterns](https://github.com/vercel-labs/agent-skills/tree/main/skills/composition-patterns)
