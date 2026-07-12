<!-- source: bulletproof-react docs/project-structure.md -->

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
