<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.svg" />
  <img alt="bulletproof-react" src="assets/logo-light.svg" width="400" />
</picture>


An agent skill for bulletproof-react conventions.
Feature folders, API layers, state categories, testing, error handling, auth, performance.
Knows when to skip the structure too.


I made this because I use bulletproof-react as a baseline for React architecture.
It is the most complete set of conventions I have found.
But I have also felt the pain of cargo-culting.
Applying patterns because they are popular, not because they fit.
This skill is designed to avoid that.
It checks scale, pushes back when you do not need the full skeleton,
and flags when you have overdone it.


Use agents to augment your judgment, not replace it.
A skill like this is a multiplier for someone who already knows what good
React architecture looks like.

It will not make that call for you.
It will remind you of the right questions to ask.


## Install

```bash
npx skills add aidrecabrera/bulletproof-react-skill
```

Auto-detects Claude Code, Codex, OpenCode, Cursor, Gemini CLI, and others.

For manual install or other agents, copy `skills/` to your agent's skills directory.


## Companion tools

- **[react-doctor](https://github.com/millionco/react-doctor).**
  Scans your codebase for state misuse, performance anti-patterns, architecture
  violations, and security issues.
  Run `npx react-doctor@latest` at your project root.

- **[Vercel react-best-practices](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-best-practices).**
  70 performance rules for React and Next.js.
  Install alongside this skill for performance-specific guidance.

- **[Vercel composition-patterns](https://github.com/vercel-labs/agent-skills/tree/main/skills/composition-patterns).**
  Component design patterns: compound components, avoiding boolean prop
  proliferation, React 19 APIs.
  Install for API design guidance.


## License

MIT. See [NOTICE.md](skills/NOTICE.md).
