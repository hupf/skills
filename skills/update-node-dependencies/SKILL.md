---
name: update-node-dependencies
description: Update the Node.js dependencies (npm, pnpm or yarn) of the current project, from security fixes to user-decided major upgrades. Use also for fixing `audit` findings or working through Renovate/Dependabot backlogs.
argument-hint: optional package directory, e.g. "frontend/"
---

# Update Node.js dependencies

Update the dependencies of one Node.js package (the argument, or the package in the current directory, or ask if the repo has several). Work in three phases:

1. **Security (critical/high first).** Safe updates (patch/minor, no breaking change) that fix audit findings. Goal: bring `audit` down to 0 critical and 0 high.
2. **Other safe minor and patch updates.** Goal: `outdated` lists only the majors of phase 3.
3. **Decision list.** Updates with more effort or risk (mostly majors). Not done without the user's OK; the user picks.

The plan file is the working document from start to end. Its shape is in `references/plan-template.md`.

## 1. Find out how the project works

Before writing the plan, read — don't assume:

- **Package manager**: the lockfile decides (`pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn (check `yarnrc.yml`/version for Berry vs. classic), `package-lock.json` → npm), plus `packageManager` in `package.json`. Use the commands for that manager everywhere (table below).
- **Project rules**: `CLAUDE.md`, `AGENTS.md`, contributing docs. Look for: branch naming and base branch, commit message convention, where working plans go, test commands (`Makefile`, `package.json` scripts), lint ratchets/suppression files, rules about overrides/resolutions, pinned majors (e.g. `@types/node` follows the Node major), Renovate/Dependabot config.
- **Registry constraints**: `.npmrc`/`.yarnrc.yml` (e.g. `minimum-release-age`, registry mirror). Respect them, never bypass.
- **Commit convention**: from the docs and `git log --oneline -20`. If messages carry an issue number, **ask the user which issue** these updates belong to (unless the branch name already makes it unambiguous — then confirm).
- **Branch**: if on the default branch, propose a branch name following the convention and create it.

## 2. Collect the facts

- `audit` (JSON output) — counts per severity, and for each finding: package, path (direct or transitive, via which parent), fixed version, whether the fix is reachable without a major.
- `outdated` — current, wanted, latest per package. Split into patch/minor vs. major.
- Group packages that move together (e.g. `react` + `react-dom` + `@types/react*`, `vitest` + `@vitest/*`, `eslint` + `@eslint/js`). One group = one commit.
- Target version = newest version allowed by the registry constraints (e.g. older than the minimum release age). Note the date.

## 3. Write the plan and stop

Write the plan (see `references/plan-template.md`) to the project's plan location (e.g. `.plans/<issue>.md`), or `dependency-updates.md` in the repo root if the project has none. One table row = one commit.

- Phase 1 sorted by severity fixed (critical first), then by how many findings a row clears. The last row handles remaining transitive findings via a lockfile refresh. If the parent's range doesn't allow the fix, the finding goes into the "remaining findings" list (see 6).
- Phase 2: the rest of the safe updates, with a note per row on what to watch.
- Phase 3: each risky update with its expected effort/risk (to be verified against the changelog before the user decides).

Then **ask the user**:
- Should the plan file be committed (on the branch, deleted before merge) or stay untracked?
- Which phases/rows to do now, and how to split them (one MR/PR for everything, one per phase, risky ones separately)?

Do not start updating before the answer.

## 4. Per-update procedure

For every row, in order:

1. **Changelog.** Read the changelog/GitHub releases for **every version between current and target** (`gh release view`, `gh api repos/<o>/<r>/releases`, `CHANGELOG.md` in the package or repo, or the web-fetch agent). Note breaking changes, deprecations, new defaults and new lint rules that affect this project. Grep the code for the APIs involved. "Minor" does not mean safe — 0.x minors and large minor jumps can break.
2. **Update** to the target, keeping the existing range style (`^`/`~`/exact).
3. **Adapt** code and config to what the changelog requires. Replace deprecated APIs/settings in the same commit. If a new lint rule fires, fix the code — don't suppress it or grow a ratchet.
4. **Verify**: the project's fast check, build, unit tests always; e2e for updates that affect the runtime (UI framework, router, component library, styling, i18n). Batch e2e runs over several runtime commits only if the user agrees; bisect if something fails.
5. **Commit** with the project convention. Subject like `Update vite to 7.3.7 #210`; the body names the changelog points that matter for this project and any code changes. Never start a message line with `#`.
6. **Record** in the plan's Progress table: versions, audit counts after the commit. Record decisions (and why) in the Decided section.

Baseline: run the full test suite once before the first update and record the result (including failures unrelated to the updates), then again after each phase and compare.

If an update turns out riskier than planned, stop, move it to phase 3 with what you learned, and tell the user.

## 5. Phase 3

Before the user decides, verify each item against its changelog and write into the plan: effort, benefit, affected files, dependencies on other items (e.g. "vitest 5 needs vite 8"). Do only what the user accepts; record declined ones with the reason.

## 6. Remaining findings

Findings that updating the direct dependencies can't fix (the parent's range doesn't allow the fixed version) are **never fixed with overrides/resolutions on your own**. For each one, write into the plan: severity, package and vulnerable range, path (via which parent and its range), dev-only or runtime, whether it is exploitable here, and what would make it go away (e.g. a parent release, a phase 3 major). Then let the user decide per finding:

- **Override/resolution**: force the fixed version. Mention the risk (the parent wasn't tested with it) and when the override can be removed.
- **Accept**: leave it, documented in the plan and the MR/PR description.

Record the decision and its reason in Decided.

## 7. Finish

- Full test suite, `audit`, `outdated`. End state: 0 critical/high, or every remaining finding decided by the user (see 6); `outdated` shows only declined/deferred majors.
- Move lasting decisions to where the project keeps them (ADR, docs, MR/PR description) — the plan file gets deleted before merge.
- Summarise for the user: what was updated, code changes made, remaining findings, open phase 3 items.

## Commands

| | pnpm | npm | yarn (Berry) | yarn classic |
|---|---|---|---|---|
| audit | `pnpm audit --json` | `npm audit --json` | `yarn npm audit --all --recursive --json` | `yarn audit --json` |
| outdated | `pnpm outdated` | `npm outdated` | `yarn upgrade-interactive` (or `yarn outdated` via plugin) | `yarn outdated` |
| update to target | `pnpm up <pkg>@<v>` | `npm install <pkg>@<v>` | `yarn up <pkg>@<v>` | `yarn upgrade <pkg>@<v>` |
| refresh transitive | `pnpm update --depth Infinity <pkg>` | `npm update <pkg>` | `yarn up -R <pkg>` | `yarn upgrade <pkg>` |
| why | `pnpm why <pkg>` | `npm explain <pkg>` | `yarn why <pkg>` | `yarn why <pkg>` |
| override (only if the user decides so) | `pnpm.overrides` | `overrides` | `resolutions` | `resolutions` |

Check the installed version's help (`<pm> <cmd> --help`) when a flag doesn't behave as expected.
