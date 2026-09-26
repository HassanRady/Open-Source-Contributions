# Open-Source Contributions

I contribute production-focused fixes to Python backend and infrastructure projects, with an emphasis on performance, concurrency, data integrity, security, and operational reliability.

**At a glance:** 12 merged pull requests across Dify, authentik, Itqan CMS, Scalene, heym, Hishel, and open-MaStR, plus a completed concurrency defect report.

## Itqan CMS Backend

[![GitHub stars](https://img.shields.io/github/stars/Itqan-community/cms-backend?style=flat&logo=github&label=Stars)](https://github.com/Itqan-community/cms-backend/stargazers)

Six merged pull requests to the Django backend powering the Itqan content-management platform.

### Security and concurrency

#### [PR #454 — Enforce access control and private caching on recitation tracks](https://github.com/Itqan-community/cms-backend/pull/454)

**Opened:** August 19, 2026 · **Merged:** August 23, 2026

- Fixed a critical authorization bypass in a public recitation endpoint where Redis cache hits skipped asset-access checks.
- Prevented restricted tracks and ayah-timing data from being returned to unauthorized or unauthenticated callers.
- Applied access-aware cache metadata and safe `Cache-Control` policies so restricted content cannot be stored by shared CDN or edge caches.
- Added regression coverage for cold and warm cache paths, authenticated access, and legacy cache entries.

#### [PR #449 — Prevent request-context leakage in rate-limit logging](https://github.com/Itqan-community/cms-backend/pull/449)

**Opened:** August 17, 2026 · **Merged:** August 19, 2026

- Diagnosed a race condition caused by mutable request state on shared throttle instances under concurrent ASGI and multi-threaded workloads.
- Replaced shared instance state with Python `ContextVar` storage, isolating user, IP, and OAuth context for each request.
- Added a concurrent multi-threaded regression test to verify that simultaneous requests cannot contaminate one another's audit logs.

### Reliability and data integrity

#### [PR #489 — Defer version notifications until transaction commit](https://github.com/Itqan-community/cms-backend/pull/489)

**Opened:** September 6, 2026 · **Merged:** September 8, 2026

- Fixed a Celery/PostgreSQL race that silently dropped subscriber emails when workers consumed tasks before the creating transaction committed.
- Moved task publication to Django `transaction.on_commit()` across asset, font, mushaf, tafsir, translation, and draft-publishing flows.
- Updated tests to execute and verify transaction commit callbacks.

#### [PR #445 — Preserve data during partial profile updates](https://github.com/Itqan-community/cms-backend/pull/445)

**Opened:** August 13, 2026 · **Merged:** August 19, 2026

- Prevented omitted fields from being replaced by empty strings during partial profile updates.
- Used Pydantic's `model_dump(exclude_unset=True)` so only explicitly submitted fields are changed.
- Wrapped related user and developer-profile changes in an atomic transaction and expanded regression coverage for persistence and explicit field clearing.

#### [PR #488 — Repair usage analytics](https://github.com/Itqan-community/cms-backend/pull/488)

**Opened:** September 6, 2026 · **Merged:** September 9, 2026

- Removed runtime failures caused by referencing a nonexistent asset field and corrected a broken Celery task import.
- Standardized metadata and event types for asset views, API access, and downloads.
- Added focused unit-test coverage for the usage analytics service.

### Containers and delivery

#### [PR #444 — Optimize and harden the backend Docker image](https://github.com/Itqan-community/cms-backend/pull/444)

**Opened:** August 12, 2026 · **Merged:** August 16, 2026

- Reworked the backend image into separate builder and runtime stages with BuildKit cache mounts and dependency-friendly layer ordering.
- Reduced the production attack surface by removing build-only packages and running services as a non-root user.
- Corrected ownership for application and persistent-volume paths, including existing deployment volumes used by static and media files.
- Moved the Celery Beat schedule to a writable runtime location and kept local, staging, and production Compose configurations aligned.

## Dify

[![GitHub stars](https://img.shields.io/github/stars/langgenius/dify?style=flat&logo=github&label=Stars)](https://github.com/langgenius/dify/stargazers)

#### [PR #39899 — Remove the N+1 insert bottleneck from Notion dataset sync](https://github.com/langgenius/dify/pull/39899)

**Opened:** August 2, 2026 · **Merged:** August 3, 2026

- Replaced a SQLAlchemy `flush()` inside the document loop with pre-generated UUID primary keys and one batch flush.
- Enabled SQLAlchemy to group inserts rather than performing a synchronous database round trip for every imported page.
- Updated the relevant unit test; local simulated benchmarks reported approximately 70% lower synchronization time from CPU-overhead savings.

## Scalene

[![GitHub stars](https://img.shields.io/github/stars/plasma-umass/scalene?style=flat&logo=github&label=Stars)](https://github.com/plasma-umass/scalene/stargazers)

#### [PR #1095 — Prevent API credentials from being embedded in generated HTML](https://github.com/plasma-umass/scalene/pull/1095)

**Opened:** September 25, 2026 · **Merged:** September 25, 2026

- Closed a credential-exposure vulnerability that copied OpenAI, Anthropic, and AWS API credentials from environment variables into generated HTML reports as plaintext JavaScript.
- Removed credential collection and template injection during HTML generation while preserving an empty `envApiKeys` object for compatibility with the existing GUI bundle.
- Added regression coverage for both regular and standalone HTML reports; users can still enter credentials through the GUI and retain them in browser-local storage.

## authentik

[![GitHub stars](https://img.shields.io/github/stars/goauthentik/authentik?style=flat&logo=github&label=Stars)](https://github.com/goauthentik/authentik/stargazers)

#### [PR #24023 — Close unusable PostgreSQL connections in the Dramatiq broker](https://github.com/goauthentik/authentik/pull/24023)

**Opened:** July 14, 2026 · **Merged:** July 14, 2026

- Fixed a PostgreSQL socket and file-descriptor leak in dynamically created Dramatiq consumer connections.
- Explicitly closed stale wrappers before reconnecting, immediately releasing server sessions and advisory locks during failovers or reconnect storms.
- Narrowed broad exception handling to database-specific errors; the fix was subsequently cherry-picked into two release branches.

## heym

[![GitHub stars](https://img.shields.io/github/stars/heymrun/heym?style=flat&logo=github&label=Stars)](https://github.com/heymrun/heym/stargazers)

#### [PR #436 — Release advisory locks before closing leader sessions](https://github.com/heymrun/heym/pull/436)

**Opened:** August 5, 2026 · **Merged:** August 6, 2026

- Fixed PostgreSQL advisory locks remaining attached to pooled connections after leader-election failures.
- Introduced explicit lock ownership tracking and deterministic unlock behavior before closing the dedicated async connection.
- Prevented self-lockout during error recovery and enabled prompt leader handoff during graceful shutdown.

## Hishel

[![GitHub stars](https://img.shields.io/github/stars/karpetrosyan/hishel?style=flat&logo=github&label=Stars)](https://github.com/karpetrosyan/hishel/stargazers)

#### [PR #119 — Correct SQLite cache-expiration logic](https://github.com/karpetrosyan/hishel/pull/119)

**Opened:** November 29, 2023 · **Merged:** November 29, 2023

- Corrected an inverted timestamp comparison that deleted valid SQLite cache entries instead of expired ones.
- Added synchronous and asynchronous cache-lifecycle assertions while preserving 100% coverage of modified lines.
- The maintainer approved the fix and scheduled it for the next release.

## open-MaStR

[![GitHub stars](https://img.shields.io/github/stars/OpenEnergyPlatform/open-MaStR?style=flat&logo=github&label=Stars)](https://github.com/OpenEnergyPlatform/open-MaStR/stargazers)

#### [PR #790 — Fix table-name lookup in interleaved XML imports](https://github.com/OpenEnergyPlatform/open-MaStR/pull/790)

**Opened:** August 20, 2026 · **Merged:** September 8, 2026

- Corrected `interleave_files` to obtain the destination table name from the table value rather than mistakenly reading the database connection URL.
- Supported both plain string names and SQLAlchemy `Table` instances.
- Added realistic unit tests for both representations and documented the change in the changelog.

#### [Issue #763 — Identify silent data loss in parallel XML imports](https://github.com/OpenEnergyPlatform/open-MaStR/issues/763)

**Opened:** July 7, 2026 · **Closed as completed:** July 14, 2026

- Identified a `ProcessPoolExecutor` race in which the worker handling the first XML chunk could delete rows already committed by another worker.
- Documented a reproducible execution sequence explaining why the import could finish successfully while silently losing data.
- Proposed moving table cleanup to the parent process before parallel workers start; the report was accepted and closed as completed.

---

GitHub: [github.com/HassanRady](https://github.com/HassanRady)
