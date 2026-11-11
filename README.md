# Agent skills

Skills in the [Agent Skills](https://agentskills.io) format (`skills/<name>/SKILL.md`). They work with Claude Code and other agents that support the format.

## Skills

- **update-node-dependencies**: updates the Node.js dependencies (npm, pnpm or yarn) of the current project in three phases: security fixes, safe minor/patch updates, and a decision list of risky updates. Works from a plan file, makes one commit per update and checks the changelog of each.

## Install

With the [`skills`](https://skills.sh) CLI, which asks which skills and which agent:

```
npx skills add hupf/skills
```

Or copy a skill folder by hand, e.g. for Claude Code into `~/.claude/skills/` (all projects) or `.claude/skills/` (one project).

## Use

Ask your agent to update the dependencies, or in Claude Code run `/update-node-dependencies [package dir]`.

## License

Apache License 2.0, see [LICENSE](LICENSE).
