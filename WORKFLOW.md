# Keroyokan Git and Delivery Workflow

Last updated: 2026-02-12

## 1) Branching Rule

- Do not work directly on `main`.
- Use short-lived feature branches from `main`.
- Naming pattern:
  - `phase-m1/coming-soon`
  - `feat/wiki-search`
  - `fix/auth-guard`
  - `chore/tooling-setup`

## 2) When To Commit

Commit when one logical unit is complete and stable, for example:

- A scoped feature slice is done
- A refactor checkpoint is safe
- A docs/config block is complete
- You are about to context switch

Do not wait for a huge batch. Prefer small, meaningful commits.

## 2.5) Beads-First Rule (Mandatory)

- Do not start implementation work without a beads issue.
- Start every work session with `bd ready`.
- If no issue exists, create one with `bd create ...` before coding.
- Claim work explicitly with `bd update <id> --status in_progress`.
- Keep work status in beads updated as the source of truth.

## 3) When To Push

Push when local commits are clean and reviewable:

- End of a focused work block
- Before break/offline time
- Before asking review or handoff
- Before opening a PR

## 4) Commit Message Convention

Format:

`type(scope): short why-focused summary`

Examples:

- `docs(plan): define roadmap and phase -1 scope`
- `feat(landing): add coming soon hero with 3d tilt card`
- `chore(tooling): initialize tailwind and shadcn-svelte`

Common `type` values:

- `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`, `build`, `ci`

## 5) Quality Gates Before Commit

Minimum gate:

1. Scope sanity:
   - changed files are intentional
   - no secrets, tokens, or accidental binaries
2. Static checks:
   - `npm run check` (or project equivalent)
   - `npm run lint` (if configured)
   - run targeted tests for touched area
3. UI gate (when UI changed):
   - run Chrome DevTools MCP check on changed route
   - verify there are no blocking console errors
   - verify responsive behavior on desktop and mobile viewport
   - take at least one screenshot as evidence

## 6) Quality Gates Before Push

- Branch is up to date enough for clean PR review
- Working tree is clean after commit
- Commit history is readable (no noisy WIP spam)
- UI evidence captured for visual changes

## 7) Chrome DevTools MCP UI Checklist

Use this checklist on every visual change:

1. Open target route with Chrome DevTools MCP
2. Check console messages for runtime errors
3. Validate desktop layout (default viewport)
4. Validate mobile layout (e.g. 390x844)
5. Interact with key element(s) at least once
6. Capture screenshot (full page or critical section)

Recommended screenshot folder:

- `artifacts/screenshots/`
- keep screenshots local-only (do not commit)

Recommended filename pattern:

- `YYYYMMDD-HHMM-route-check.png`

## 8) beads Integration Rule

- One-time setup per clone:
  - `bd onboard`
  - `bd hooks install`
- Start task:
  - `bd ready`
  - `bd show <id>`
  - `bd update <id> --status in_progress`
- During work:
  - keep notes in beads issue comments/notes
  - if scope changes, create follow-up issues instead of hidden TODOs
- Commit:
  - include beads issue ID in commit body/footer when possible
- Finish task:
  - `bd close <id>` when acceptance is met
  - `bd sync` before push
  - `git push`

## 9) Suggested Command Flow

Typical sequence:

1. `git checkout -b <branch-name>`
2. `bd ready` or `bd create ...`
3. `bd update <id> --status in_progress`
4. implement change
5. run quality gates
6. `git add <files>`
7. `git commit -m "type(scope): summary"`
8. `bd close <id>` (when done)
9. `bd sync`
10. `git push -u origin <branch-name>`
11. open PR

---

This workflow is mandatory for all upcoming phases, starting with Phase -1.
