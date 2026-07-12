<!-- source: bulletproof-react docs/state-management.md -->

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
