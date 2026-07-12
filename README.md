An agent skill for bulletproof-react conventions. Feature folders, API layers, state categories, testing, error handling, auth, performance. Knows when to skip the structure too.

## Install

```bash
npx skills add aidrecabrera/bulletproof-react-skill
```

Auto-detects Claude Code, Codex, OpenCode, Cursor, Gemini CLI, and others.

For manual install or other agents, copy `skills/` to your agent's skills directory.

## Companion tools

- **[react-doctor](https://github.com/millionco/react-doctor).** Scans your codebase for state misuse, performance anti-patterns, architecture violations, and security issues. I added this to the skill because it's a great tool for catching what agents miss. Run `npx react-doctor@latest` at your project root.
- **[Vercel react-best-practices](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-best-practices).** 70 performance rules for React and Next.js. Install alongside this skill for performance-specific guidance.
- **[Vercel composition-patterns](https://github.com/vercel-labs/agent-skills/tree/main/skills/composition-patterns).** Component design patterns (compound components, avoiding boolean prop proliferation, React 19 APIs). Install for API design guidance.

## License

MIT. See [NOTICE.md](skills/NOTICE.md).
