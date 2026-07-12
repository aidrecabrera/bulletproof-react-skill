<!-- source: bulletproof-react docs/testing.md -->

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
