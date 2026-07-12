<!-- source: bulletproof-react docs/api-layer.md -->

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
