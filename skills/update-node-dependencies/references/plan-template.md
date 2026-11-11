# Plan template

Use this structure for the working plan. Fill in what you found; drop sections that don't apply. Keep it short — it's a working document, not a report. The example values come from a real run (pnpm, Vite/React frontend, issue #210).

```markdown
# #<issue> <Package dir> dependency updates

## Context

Why (ticket, Renovate backlog, audit). What we do: update `<dir>/` directly on a branch, one commit per update (packages that move together share one commit). Safe updates first, then a decision list of risky updates.

Today `<pm> audit` reports **N findings: a critical, b high, c moderate, d low**. Where they are (dev tooling vs. runtime), and whether every critical/high has a fix without a major.

## Setup

- Branch `<name>` from `<base>`.
- Plan file: `<path>` (committed / untracked, as the user decided).
- Baseline: `<full test command>` once before the first update.
- Commit messages: `Update vite to 7.3.7 (#210)`; the body names the relevant changelog points and code changes.
- Constraints: e.g. `.npmrc` `minimum-release-age=10080` (don't bypass), lint ratchets only shrink, no overrides without asking.

## Per-update procedure

1. Read the changelog for every version between current and target; grep our code for affected APIs.
2. `<pm> up <pkgs>@<target>`.
3. Adapt code/config.
4. Verify: always `<check>`, `<build>`, `<unit tests>`; for runtime updates (<list>) also e2e.
5. Commit; record `audit` counts below.

## Phase 1: security (critical/high first)

| # | Commit | From → To | Fixes |
|---|---|---|---|
| 1 | vitest (+ @vitest/*) | 4.0.18 → 4.1.12 | **critical** vitest; @vitest/mocker moderate |
| 2 | vite | 7.3.1 → 7.3.7 | 8 vite advisories; rollup, postcss, transitive |
| … | | | |
| n | Remaining transitive findings | — | lockfile refresh; if the parent's range blocks the fix → remaining findings, not overrides |

After phase 1: full tests and audit. Target: 0 critical, 0 high.

## Phase 2: other safe minor and patch updates

| # | Commit | From → To | Note |
|---|---|---|---|
| 9 | react, react-dom, @types/react, @types/react-dom | 19.2 → 19.3 | read the release notes (minor, but it is React) |
| 10 | lucide-react | 0.562 → 0.577 | 0.x minors can remove icons; tsc catches that |
| … | | | |

At the end, `<pm> outdated` lists only the majors below.

## Phase 3: decision list (risky updates, not done without your OK)

Verified against the changelog before you decide; effort, benefit and affected files go here.

| Update | Expected risk/effort |
|---|---|
| vite 7 → 8 + @vitejs/plugin-react 5 → 6 | Rolldown instead of Rollup; check plugins and config |
| typescript 5.9 → 7 | native compiler; check `tsc -b`, typescript-eslint, editor tooling. High risk |
| @types/node 24 → 26 | **don't**: follows the Node major |
| … | |

## Files

- `<dir>/package.json`, lockfile (every commit)
- as needed: config files, components using affected APIs
- the plan file

## Verification

- Full tests before phase 1, after each phase, at the end; fast check before every commit.
- Audit after every commit, counts below. End state: 0 critical/high; remaining findings documented.
- `outdated` at the end shows only phase 3 majors.
- Manual check of affected UI after runtime updates.

## Progress

Audit counts (critical / high / moderate / low) after each step.

| Step | Versions | Audit |
|---|---|---|
| Baseline | — | 3 / 45 / 26 / 2 |
| 1 vitest | 4.1.12 | 2 / 44 / 22 / 2 |
| … | | |

## Decided

Dated entries, with the reason. E.g.:
- Target versions are the newest ones older than 7 days on <date>.
- Baseline test run is red for reasons unrelated to this work: <what>. Compared against the run after phase 1.
- Deprecated APIs flagged by changelogs are replaced in the commit that updates the package.

## Remaining audit findings

Findings that updating the direct dependencies can't fix. Per finding: severity, package and range, path (via which parent and its range), dev-only or runtime, why it can't be fixed now, what would make it go away, and **the user's decision** (override/resolution or accept) with the reason. No override is added before the user decides. Copy into the MR/PR description so it survives the deletion of the plan.
```
