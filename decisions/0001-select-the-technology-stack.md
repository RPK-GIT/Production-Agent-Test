# 0001 — Select the technology stack

- **Status:** Accepted
- **Date:** 2026-08-22
- **Deciders:** Project Owner (accepted 2026-08-22)

## Context

Production-Agent-Test is a new production-grade application with the declared profile
`web, api-service, uses-database` (Software Engineering Playbook v4.1.0, pinned via the
`playbook/` submodule). No code exists yet; per the adoption contract
(playbook `governance/how-to-use.md` §5.7) and DOC-003 trigger 1, the technology stack must be
selected and recorded before implementation begins.

Forces acting on the selection:

- **Playbook obligations the stack must satisfy mechanically:** blocking type checks (CODE-003),
  machine-checkable component boundaries (ARCH-006), a committed machine-readable API contract
  (API-001), versioned ordered migrations (DB-001) applied by the pipeline (CI-009), bounded
  connection pooling (DB-012), contract and end-to-end tests (TEST-007, TEST-008), CI accessibility
  checks (WEB-009), CI-verified performance budgets (WEB-010), dependency/secret scanning
  (SEC-019, SEC-020), SAST (SEC-021), non-root scanned container images with SBOMs
  (INFRA-018, INFRA-020, INFRA-021), and structured, correlated telemetry with health signals and
  baseline metrics (OBS-001, OBS-003, OBS-007, OBS-008).
- **Architecture default:** a single deployable with well-separated internal components
  (ARCH-009); no speculative mechanisms (ARCH-008).
- **Future AI/LLM integration is anticipated** but has no concrete requirement yet, so no AI
  infrastructure is introduced now (ARCH-008); the stack must simply not preclude it.
- **Maintainability by a professional engineering team:** mainstream, actively maintained
  technologies with large hiring pools and mature tooling.

## Decision

Adopt a TypeScript-first, single-deployable stack:

| Concern | Selection |
|---|---|
| Language / runtime | TypeScript (strict mode) on Node.js 22 LTS; pnpm workspaces monorepo |
| Backend framework | NestJS on the Fastify adapter |
| Frontend | React with Vite, built as a SPA served by the backend (one deployable, ARCH-009) |
| Database | PostgreSQL 17 |
| Data access & migrations | Drizzle ORM with drizzle-kit SQL migrations; `node-postgres` bounded pool (DB-012) |
| API style | Resource-oriented HTTP/JSON; OpenAPI 3.1 contract committed to the repository (API-001); URL-path versioning `/v1` (API-003) |
| Trust-boundary validation | Zod schemas at every entry point (SEC-005) |
| Testing | Vitest (unit, integration); Testcontainers for real-PostgreSQL integration tests (TEST-006); Playwright for E2E (TEST-008) with `@axe-core/playwright` accessibility checks (WEB-009); implementation verified against the OpenAPI contract in CI (TEST-007) |
| Code quality | Prettier (CODE-001); ESLint with typescript-eslint strict and an enforced complexity limit (CODE-002, CODE-010); dependency-cruiser for machine-checked boundary rules (ARCH-006) |
| Commit convention | Conventional Commits enforced by commitlint (GIT-006) |
| Containerization | Docker multi-stage builds; non-root user (INFRA-018); slim/minimal base image (INFRA-019); Trivy image scanning (INFRA-020); Syft-generated SBOM (INFRA-021); Docker Compose for local development |
| CI/CD platform | GitHub Actions (repository is hosted on GitHub; consistent with org ADR-0002) |
| CI security scanning | Dependabot + osv-scanner for dependency vulnerabilities (SEC-019, SEC-028); gitleaks for secret scanning (SEC-020); CodeQL for SAST (SEC-021) |
| Web performance | Lighthouse CI enforcing the accepted default budgets — Core Web Vitals "good" thresholds and a 200 KB compressed initial-route JS budget (WEB-010; defaults per web.md §3, no WEB-029 deviation) |
| Observability | pino structured JSON logging (OBS-001); correlation-ID middleware (OBS-003/004); OpenTelemetry SDK for traces and metrics (OBS-008); liveness/readiness endpoints (OBS-007) |

**Scope.** This ADR covers the initial stack for the single deployable. It deliberately does
**not** decide the following, each of which requires its own ADR when its trigger fires:

- **Cloud/deployment platform and environment topology** — hard to reverse (DOC-003 trigger 8),
  requires business input on cost, region, and recovery objectives (INFRA-023).
- **Authentication/authorization mechanism** — security-relevant selection (DOC-003 trigger 6),
  requires a threat model (SEC-027) and security review (SEC-026 trigger 1).
- **LLM provider and SDK** — no concrete requirement exists yet; introducing it now would violate
  ARCH-008. The chosen stack keeps the path open: mature TypeScript SDKs exist for the major
  providers, and PostgreSQL supports `pgvector` should retrieval workloads arrive.
- Exact dependency versions are pinned in the lockfile at implementation time (REPO-006); this
  ADR fixes technologies and major lines, not patch versions.

## Consequences

**Positive:**

- One language across frontend, backend, tests, and tooling minimizes context-switching, hiring
  surface, and duplicated validation logic (Zod schemas shared across the API boundary).
- Strict TypeScript satisfies CODE-003 with zero additional infrastructure.
- NestJS's module system maps directly onto ARCH-001…004 (declared responsibilities, DI-enforced
  dependency direction) and generates the OpenAPI contract from the code that serves it,
  keeping API-001's contract from drifting.
- SQL-first migrations (plain files, ordered, committed) satisfy DB-001/DB-002 transparently and
  keep query plans inspectable for DB-008.
- Every `ci`-tagged mandatory check in the enforcement matrix has a concrete, mainstream tool
  assigned — the pipeline can be built without further research decisions.
- Single deployable keeps operational surface minimal (ARCH-009) while dependency-cruiser keeps
  internal boundaries real enough to extract a service later if a recorded requirement demands it.

**Negative / accepted costs:**

- Node.js is single-threaded per process; CPU-bound workloads (a plausible shape for future AI
  pre/post-processing) need worker threads or horizontal scaling rather than in-process
  parallelism. Accepted: the anticipated workload is I/O-bound; INFRA-013 scaling strategy will
  be declared at deployment time.
- NestJS adds a framework layer and DI conventions that must be learned; less code-transparent
  than minimal Fastify. Accepted for the structural guarantees and OpenAPI integration.
- Drizzle is younger than Prisma/TypeORM; smaller community. Accepted for SQL transparency,
  lighter runtime, and native SQL migration files; reversal cost is moderate and localized to the
  persistence layer (ARCH-003 keeps the domain free of it).
- A SPA weakens no-JavaScript resilience (WEB-016 is SHOULD-level; the deviation will be recorded
  per AGENT-009 when the frontend is implemented, or revisited if requirements demand otherwise).
- TypeScript's AI/ML ecosystem is thinner than Python's; if future AI work becomes model-heavy
  (fine-tuning, local inference) rather than API-mediated, a separate service in another language
  may be justified — that is a future ADR under ARCH-009's recorded-requirement bar.

## Alternatives considered

**Backend language/framework:**

- **Python (FastAPI + SQLAlchemy + Alembic)** — strongest AI/ML ecosystem, excellent OpenAPI
  support. Lost: forces a two-language stack (Python API + TypeScript frontend), doubling tooling,
  CI matrix, and hiring surface; current AI integration need is API-mediated, where TypeScript
  SDKs are first-class.
- **Java (Spring Boot) / C# (ASP.NET Core)** — mature, enterprise-proven, strong typing. Lost:
  heavier operational and cognitive footprint for a small team, slower iteration, no language
  sharing with the frontend.
- **Go (chi/echo + sqlc)** — excellent performance and deployment simplicity. Lost: no frontend
  language sharing, thinner ORM/OpenAPI/E2E ecosystem, smaller overlap with AI SDK maturity.
- **Fastify alone (no NestJS)** — smaller, faster to learn. Lost: module boundaries, DI, and
  OpenAPI generation would each need hand-rolled conventions that NestJS provides and enforces.

**Frontend:**

- **Next.js (SSR/RSC)** — better first-paint and progressive enhancement (WEB-016). Lost: adds a
  second server runtime or couples the API into Next.js, violating the single-deployable default
  (ARCH-009) or blurring API-001 contract ownership; SSR complexity is unjustified by current
  requirements (ARCH-008).
- **Vue/Nuxt, SvelteKit** — capable alternatives. Lost on team-familiarity assumption and
  ecosystem depth for testing/accessibility tooling; no decisive technical advantage here.

**Database:**

- **MySQL/MariaDB** — comparable relational capability. Lost: PostgreSQL's richer constraint
  and index toolkit (DB-006, DB-008), transactional DDL for safer migrations (DB-002), JSONB,
  and `pgvector` for future AI retrieval.
- **MongoDB** — flexible schema. Lost: the domain is expected to be relational; datastore-enforced
  integrity (DB-006) and migration discipline are weaker fits.
- **SQLite** — operationally trivial. Lost: unsuitable for a horizontally scaled production web
  service (INFRA-011); write concurrency limits.

**API style:**

- **GraphQL** — flexible client queries. Lost: no requirement for client-shaped queries exists
  (ARCH-008); complicates rate limiting (SEC-023), caching (API-011/HTTP semantics), and
  per-resource authorization (SEC-002).
- **gRPC** — efficient service-to-service RPC. Lost: browser consumption requires a proxy layer;
  this API's first consumer is the web frontend.
- **tRPC** — end-to-end TS type safety. Lost: contract exists only in TypeScript types, weakening
  API-001's language-neutral machine-readable contract for future non-TS consumers.

**ORM/migrations:** **Prisma** (most popular, mature migrate tooling — lost on runtime weight,
less SQL control for DB-008 plan verification) and **TypeORM** (NestJS default — lost on long-standing
maintenance concerns and weaker migration ergonomics).

**Testing:** **Jest** (lost: slower, ESM friction; Vitest is API-compatible and Vite-native) and
**Cypress** (lost: Playwright has broader browser coverage, first-class parallelism, and the
axe-core integration used for WEB-009).

**CI/CD:** **GitLab CI / CircleCI / Jenkins** — lost: the repository is on GitHub; GitHub Actions
removes an external system, and the org already uses it (playbook ADR-0002 precedent).

**Containerization:** **none (platform buildpacks / bare VMs)** — lost: containers give
build-once-promote-unchanged artifacts (CI-003/CI-004) and a portable local dev environment
without committing to a cloud platform yet.

**Do nothing** (decide per-feature as code is written) — lost: violates DOC-003 trigger 1 and
guarantees incoherent, unreviewable technology drift.

## Standards impact

- **Complies with:** DOC-003 (trigger 1 — this record); ARCH-008/ARCH-009 (single deployable, no
  speculative mechanisms, AI infrastructure deferred); CODE-001/002/003/010 (tooling assigned);
  ARCH-006 (dependency-cruiser); API-001/003 (OpenAPI 3.1, declared versioning); DB-001/012
  (drizzle-kit migrations, bounded pool); TEST-006/007/008 (Testcontainers, contract tests,
  Playwright); WEB-009/010 (axe-core, Lighthouse CI with accepted default budgets — no WEB-029
  deviation); SEC-019/020/021/028 (scanners assigned); INFRA-018…021 (container rules);
  OBS-001/003/007/008 (pino, correlation IDs, health endpoints, OTel); GIT-006 (commitlint);
  REPO-006 (pnpm lockfile).
- **Deviations (SHOULD-level):** none taken by this ADR. (WEB-016 progressive enhancement is
  expected to be deviated from by the SPA choice; it will be recorded per AGENT-009 with the
  implementing change, since no browser-delivered UI exists yet.)
- **Waivers required (MUST-level):** none.

## Assumptions

Recorded per AGENT-011; each needs confirmation or will be resolved by a future ADR:

1. The application is **single-tenant** until stated otherwise (SEC-025 scoping deferred).
2. **No internationalization requirement** is currently declared (WEB-026/027 inapplicable until
   one exists; if i18n is excluded permanently, that exclusion must be recorded in the README or
   an ADR).
3. The **browser support matrix** (WEB-014) will be declared from actual user-base data before
   the first UI ships; assumed evergreen browsers meanwhile.
4. Whether the application **handles PII** is undetermined; the `handles-pii` trigger tag will be
   raised when data classification (SEC-014) happens at schema design.
5. Future AI/LLM integration will be **API-mediated** (hosted models over HTTP), not local
   inference; if that changes, the language choice must be revisited by a superseding ADR.
6. Team familiarity with TypeScript/React is assumed adequate; no team-skills inventory was
   available to this decision.
