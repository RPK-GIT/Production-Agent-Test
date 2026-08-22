# Production-Agent-Test

A production-grade test application — a browser web interface backed by an HTTP API and a
relational database (profile: `web` + `api-service` + `uses-database`). The repository is
currently in the **adoption phase**: engineering standards, decisions, and CI scaffolding are in
place; application implementation has not started. The selected technology stack is recorded in
[ADR 0001](decisions/0001-select-the-technology-stack.md) (TypeScript / Node.js 22 LTS, NestJS on
Fastify, React + Vite, PostgreSQL 17, Drizzle ORM, OpenAPI 3.1).

## Prerequisites

- git (the standards playbook is a submodule — clone with `git clone --recurse-submodules`, or
  run `git submodule update --init --recursive` after a plain clone)

Toolchain prerequisites (Node.js 22 LTS, pnpm, Docker) become binding when the first
implementation change lands; this section is updated in that same pull request (DOC-002).

## Build · Test · Run

No application code exists yet. The commands are added here, copy-pasteable, in the pull request
that introduces the toolchain (DOC-001, DOC-002).

## Configuration

All configuration keys are enumerated and documented in the tracked template
[`.env.example`](.env.example) per REPO-004 / DOC-005. Copy it and fill values for your
environment; value files (`.env`) are never committed (REPO-003). No keys exist yet.

## Repository layout

| Path | Contents |
|---|---|
| `decisions/` | Project architecture decision records (ADRs) |
| `playbook/` | Software Engineering Playbook v4.1.0 (git submodule, read-only) |
| `.github/` | CI pipeline and pull request template |

Source, tests, scripts, and migrations directories are established with the first implementation
change per ecosystem convention (REPO-007).

## Engineering standards

This repository adopts the Software Engineering Playbook — version and profile are declared in
[CLAUDE.md](CLAUDE.md). Decisions live in `decisions/`. The Definition of Done is the playbook's
`checklists/definition-of-done.md` filtered by the declared profile.
