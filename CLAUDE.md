# CLAUDE.md — Production-Agent-Test

## Playbook adoption

```yaml
playbook:
  repository: https://github.com/RPK-GIT/Software-Engineering-Playbook
  version: v4.1.0
  profile: [web, api-service, uses-database]
```

All rules whose applicability matches `all`, a profile tag above, or a trigger raised by your
change apply to this repository. Load them per the playbook's `agents/context-map.md`, starting
from its `CLAUDE.md`. The playbook is vendored read-only at [`playbook/`](playbook/) as a git
submodule pinned to the version above.

The project's Definition of Done is the playbook's `checklists/definition-of-done.md` filtered
by the profile above (AGENT-010); no stricter project extensions exist yet (RULE-009 permits
only adding strictness).

## Decisions

Project ADRs: [`decisions/`](decisions/) — scan the index before designing; accepted ADRs bind you.
Org-level ADRs live in the playbook repository.

Accepted: [ADR 0001](decisions/0001-select-the-technology-stack.md) — technology stack
(TypeScript on Node.js 22 LTS, NestJS on Fastify, React + Vite, PostgreSQL 17, Drizzle ORM,
OpenAPI 3.1, Vitest/Testcontainers/Playwright, Docker, GitHub Actions).

## Commands

No application code exists yet. Build, test, lint, and run commands are added to this section in
the same pull request that introduces the toolchain (DOC-002). Until then the only automated
checks are the repository-level CI jobs in `.github/workflows/ci.yml`.

## Project conventions

- Commit messages follow Conventional Commits (GIT-006), linted in CI via
  `commitlint.config.mjs`.
- Configuration keys are enumerated and documented in `.env.example` (REPO-004, DOC-005);
  `.env` files are never tracked (REPO-003).

## Boundaries

- `playbook/` — the pinned standards submodule. Never modify it (AGENT-021); it changes only
  through a deliberate pin-bump pull request (playbook `governance/how-to-use.md` §3). Rule
  defects are proposed through the playbook's own change process.
