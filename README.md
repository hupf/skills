# Claude Code skills

## Skills

- **update-node-dependencies**: updates the Node.js dependencies (npm, pnpm or yarn) of the current project in three phases: security fixes, safe minor/patch updates, and a decision list of risky updates. Works from a plan file, makes one commit per update and checks the changelog of each.

## Install

```
/plugin marketplace add hupf/skills
/plugin install node-deps@hupf-skills
```

Run it with `/node-deps:update-node-dependencies [package dir]`, or ask Claude to update the dependencies.
