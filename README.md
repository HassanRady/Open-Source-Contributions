# Open-Source Contributions

I contribute production-focused fixes to Python backend and infrastructure projects, with an emphasis on performance, concurrency, data integrity, security, and operational reliability.

**At a glance:** 26 merged pull requests across Dify, authentik, Itqan CMS, Celery, redis-py, Starlette, Scalene, Glances, heym, Hishel, and open-MaStR, plus a completed concurrency defect report.

## Itqan CMS

[![GitHub stars](https://img.shields.io/github/stars/Itqan-community/cms-backend?style=flat&logo=github&label=Stars)](https://github.com/Itqan-community/cms-backend/stargazers)

<details>
<summary>View contributions</summary>

Ten merged pull requests to the Django backend powering the Itqan content-management platform.

### Security and concurrency

#### [PR #517 — Prevent races in asset access requests and grants](https://github.com/Itqan-community/cms-backend/pull/517)

**Opened:** October 2, 2026 · **Merged:** October 7, 2026

- Fixed concurrent access operations that could leave rejected requests with active grants, create duplicate requests, or fail with HTTP 500 errors.
- Made reviewer decisions and submissions transactional, locked requests before validating their pending status, serialized submissions per asset, and committed approval together with grant creation; competing decisions return HTTP 409.
- Made grant creation idempotent by reusing grants for each user/asset pair while preserving their original request association and handling license and expiry updates.
- Added concurrency, rollback, and grant-reuse regression tests and documented the access-request consistency guarantees.

#### [PR #511 — Prevent cross-tenant recitation upload mutations](https://github.com/Itqan-community/cms-backend/pull/511)

**Opened:** September 26, 2026 · **Merged:** September 30, 2026

- Closed an authorization gap that allowed portal users with permissions for one publisher to target another publisher's recitation assets and multipart uploads.
- Scoped audio and timing asset lookups to the caller's publisher and authorized storage keys before signing parts, completing uploads, or aborting uploads.
- Validated canonical storage-key representations, asset/folder ownership, and completion payload consistency while preserving the existing frontend API contract; inaccessible resources return not-found responses without modifying foreign records or storage objects.
- Added regression coverage for foreign assets, malformed and noncanonical keys, and mismatched asset IDs, and documented the upload authorization boundary.

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

#### [PR #515 — Stream R2 audio fallback in bounded chunks](https://github.com/Itqan-community/cms-backend/pull/515)

**Opened:** October 1, 2026 · **Merged:** October 7, 2026

- Removed whole-file MP3 buffering from the audio-duration fallback, reducing worker memory pressure during large or concurrent uploads.
- Streamed Cloudflare R2 audio in 1 MiB chunks to a seekable temporary file and explicitly closed both the response body and temporary file.
- Added regression coverage for chunked transfer, reconstructed audio content, and response closure.

#### [PR #521 — Restore migration consistency for asset-version audit history](https://github.com/Itqan-community/cms-backend/pull/521)

**Opened:** October 6, 2026 · **Merged:** October 6, 2026

- Added a missing Django migration for historical asset-version records that was causing the CI `makemigrations --check` gate to fail.
- Aligned the audit-history schema with asset-version model changes by adding the human-readable `label` field and updating the `name` field definition.

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

</details>

## Celery

[![GitHub stars](https://img.shields.io/github/stars/celery/celery?style=flat&logo=github&label=Stars)](https://github.com/celery/celery/stargazers)

<details>
<summary>View contributions</summary>

#### [PR #10776 — Release Beat scheduler broker resources on shutdown](https://github.com/celery/celery/pull/10776)

**Opened:** October 3, 2026 · **Merged:** October 5, 2026

- Fixed broker connection and channel leaks when embedded Celery Beat stopped while its scheduler remained referenced, preventing resources from accumulating across repeated lifecycle cycles.
- Released cached publishing connections and cleared cached producers during shutdown, including synchronization failures, without creating new resources; ensured persistent scheduler storage closes even if base cleanup fails.
- Added unit coverage for cleanup and failure paths and a RabbitMQ smoke test verifying connection, channel, and socket closure across three embedded Beat lifecycle cycles for each scheduler class, with Python 3.13 SQLite shelf compatibility.

#### [PR #10745 — Synchronize event dispatcher shutdown with publishing](https://github.com/celery/celery/pull/10745)

**Opened:** September 30, 2026 · **Merged:** October 3, 2026

- Fixed a shutdown race where `EventDispatcher.close()` released a mutex held by another thread during event publication or flushing, causing `RuntimeError: release unlocked lock`.
- Made shutdown acquire the mutex normally and wait for in-flight publication before clearing the producer; added closed-state checks to prevent prepared publish or flush operations from using the producer after shutdown, with state reset on reopening.
- Added unit coverage for concurrent shutdown interleavings and a broker-backed smoke test that pauses publication while another thread closes the dispatcher.

#### [PR #10734 — Preserve pending consumer operation order](https://github.com/celery/celery/pull/10734)

**Opened:** September 29, 2026 · **Merged:** September 30, 2026

- Fixed reversed execution of deferred consumer callbacks in workers without an event-loop hub, where an add-then-cancel sequence could execute as cancel-then-add and leave a queue enabled.
- Replaced the pending-operation list with a `deque` drained through `popleft()` to preserve FIFO scheduling order while retaining deferred execution and exception handling.
- Added regression coverage verifying that callbacks execute in their scheduled order.

</details>

## redis-py

[![GitHub stars](https://img.shields.io/github/stars/redis/redis-py?style=flat&logo=github&label=Stars)](https://github.com/redis/redis-py/stargazers)

<details>
<summary>View contributions</summary>

#### [PR #4405 — Prevent synchronous token startup from deadlocking its event loop](https://github.com/redis/redis-py/pull/4405)

**Opened:** October 8, 2026 · **Merged:** October 9, 2026

- Fixed `TokenManager.start()` deadlocking when called from a thread that already runs an asyncio event loop.
- Moved synchronous token renewal to a dedicated background event loop while preserving its blocking initialization contract and public API.
- Hardened restart and shutdown handling so background loops and threads are reliably stopped and closed; added regression coverage for active-loop startup, repeated lifecycle operations, scheduling failures, and concurrent shutdown edge cases.

#### [PR #4398 — Preserve async MultiDBClient lifetime when pipeline contexts exit](https://github.com/redis/redis-py/pull/4398)

**Opened:** October 7, 2026 · **Merged:** October 7, 2026

- Fixed pipeline context exit unintentionally shutting down the parent `MultiDBClient`, canceling background health checks and disrupting client reuse or another pipeline still executing.
- Removed parent-client shutdown from pipeline exit while preserving pipeline cleanup through `reset()`, keeping the client active until its own context closes.
- Added asynchronous regression tests for client reuse, exception paths, concurrent pipelines, continued health-check scheduling, and exactly-once closure of the underlying Redis client at final shutdown.

#### [PR #4392 — Prevent event-loop blocking during async cluster WATCH retries](https://github.com/redis/redis-py/pull/4392)

**Opened:** October 6, 2026 · **Merged:** October 7, 2026

- Fixed a blocking retry path in asynchronous Redis Cluster transactions where a positive `watch_delay` after a WATCH conflict paused the entire event loop.
- Replaced `time.sleep()` with awaited `asyncio.sleep()`, allowing other coroutines to progress during retry backoff and aligning cluster behavior with the standalone async client.

</details>

## Dify

[![GitHub stars](https://img.shields.io/github/stars/langgenius/dify?style=flat&logo=github&label=Stars)](https://github.com/langgenius/dify/stargazers)

<details>
<summary>View contributions</summary>

#### [PR #39899 — Remove the N+1 insert bottleneck from Notion dataset sync](https://github.com/langgenius/dify/pull/39899)

**Opened:** August 2, 2026 · **Merged:** August 3, 2026

- Replaced a SQLAlchemy `flush()` inside the document loop with pre-generated UUID primary keys and one batch flush.
- Enabled SQLAlchemy to group inserts rather than performing a synchronous database round trip for every imported page.
- Updated the relevant unit test; local simulated benchmarks reported approximately 70% lower synchronization time from CPU-overhead savings.

</details>

## Scalene

[![GitHub stars](https://img.shields.io/github/stars/plasma-umass/scalene?style=flat&logo=github&label=Stars)](https://github.com/plasma-umass/scalene/stargazers)

<details>
<summary>View contributions</summary>

#### [PR #1099 — Preserve semaphore identity across spawned processes](https://github.com/plasma-umass/scalene/pull/1099)

**Opened:** September 26, 2026 · **Merged:** October 1, 2026

- Fixed a multiprocessing correctness defect where serialization reconstructed an independent semaphore in a child process, allowing it to acquire a lock still held by its parent and defeating mutual exclusion.
- Removed a state-discarding custom reducer and non-importable qualified name, restoring Python's inherited `SemLock` serialization to preserve the shared semaphore handle and identity while retaining process-spawning compatibility.
- Added regression coverage for `spawn` and `forkserver` where available, proving that child processes cannot acquire a parent-held lock and preserving standard restrictions on pickling locks outside process spawning.

#### [PR #1095 — Prevent API credentials from being embedded in generated HTML](https://github.com/plasma-umass/scalene/pull/1095)

**Opened:** September 25, 2026 · **Merged:** September 25, 2026

- Closed a credential-exposure vulnerability that copied OpenAI, Anthropic, and AWS API credentials from environment variables into generated HTML reports as plaintext JavaScript.
- Removed credential collection and template injection during HTML generation while preserving an empty `envApiKeys` object for compatibility with the existing GUI bundle.
- Added regression coverage for both regular and standalone HTML reports; users can still enter credentials through the GUI and retain them in browser-local storage.

</details>

## Starlette

[![GitHub stars](https://img.shields.io/github/stars/Kludex/starlette?style=flat&logo=github&label=Stars)](https://github.com/Kludex/starlette/stargazers)

<details>
<summary>View contributions</summary>

#### [PR #3583 — Stop multipart range responses on unexpected EOF](https://github.com/Kludex/starlette/pull/3583)

**Opened:** September 24, 2026 · **Merged:** September 26, 2026

- Fixed an infinite loop in `FileResponse` multipart range handling when a file reaches EOF before the requested byte range is complete, such as when cached metadata becomes stale after the file is shortened.
- Detects zero-byte reads and terminates with a clear `RuntimeError` instead of repeatedly emitting empty ASGI messages with `more_body=True`.
- Added a regression test that combines stale file metadata with a subsequently truncated file and verifies that the response exits promptly.

</details>

## Glances

[![GitHub stars](https://img.shields.io/github/stars/nicolargo/glances?style=flat&logo=github&label=Stars)](https://github.com/nicolargo/glances/stargazers)

<details>
<summary>View contributions</summary>

#### [PR #3767 — Isolate export snapshots from live plugin statistics](https://github.com/nicolargo/glances/pull/3767)

**Opened:** September 28, 2026 · **Merged:** September 30, 2026

- Fixed export preparation mutating shared plugin statistics, which could expose exporter-only fields through APIs and UIs or contaminate concurrently running exporters.
- Returned deep-copied snapshots from both export getters, isolating each exporter from live statistics and other exporters, including nested data.
- Added regression coverage for dictionary- and list-based plugin statistics and verified that standard export preparation leaves live statistics unchanged.

#### [PR #3740 — Recover TimescaleDB exports after failed transactions](https://github.com/nicolargo/glances/pull/3740)

**Opened:** September 22, 2026 · **Merged:** September 26, 2026

- Fixed a failure mode where one invalid TimescaleDB write left the persistent Psycopg connection in an aborted transaction state, stopping every subsequent export until Glances restarted.
- Wrapped each export in a Psycopg transaction context for automatic rollback and serialized access so overlapping exporter threads cannot share transaction state.
- Added regression and Docker integration coverage proving that a failed insert returns the connection to `IDLE` and that the next export succeeds using the same connection.

</details>

## authentik

[![GitHub stars](https://img.shields.io/github/stars/goauthentik/authentik?style=flat&logo=github&label=Stars)](https://github.com/goauthentik/authentik/stargazers)

<details>
<summary>View contributions</summary>

#### [PR #24023 — Close unusable PostgreSQL connections in the Dramatiq broker](https://github.com/goauthentik/authentik/pull/24023)

**Opened:** July 14, 2026 · **Merged:** July 14, 2026

- Fixed a PostgreSQL socket and file-descriptor leak in dynamically created Dramatiq consumer connections.
- Explicitly closed stale wrappers before reconnecting, immediately releasing server sessions and advisory locks during failovers or reconnect storms.
- Narrowed broad exception handling to database-specific errors; the fix was subsequently cherry-picked into two release branches.

</details>

## heym

[![GitHub stars](https://img.shields.io/github/stars/heymrun/heym?style=flat&logo=github&label=Stars)](https://github.com/heymrun/heym/stargazers)

<details>
<summary>View contributions</summary>

#### [PR #436 — Release advisory locks before closing leader sessions](https://github.com/heymrun/heym/pull/436)

**Opened:** August 5, 2026 · **Merged:** August 6, 2026

- Fixed PostgreSQL advisory locks remaining attached to pooled connections after leader-election failures.
- Introduced explicit lock ownership tracking and deterministic unlock behavior before closing the dedicated async connection.
- Prevented self-lockout during error recovery and enabled prompt leader handoff during graceful shutdown.

</details>

## Hishel

[![GitHub stars](https://img.shields.io/github/stars/karpetrosyan/hishel?style=flat&logo=github&label=Stars)](https://github.com/karpetrosyan/hishel/stargazers)

<details>
<summary>View contributions</summary>

#### [PR #119 — Correct SQLite cache-expiration logic](https://github.com/karpetrosyan/hishel/pull/119)

**Opened:** November 29, 2023 · **Merged:** November 29, 2023

- Corrected an inverted timestamp comparison that deleted valid SQLite cache entries instead of expired ones.
- Added synchronous and asynchronous cache-lifecycle assertions while preserving 100% coverage of modified lines.
- The maintainer approved the fix and scheduled it for the next release.

</details>

## open-MaStR

[![GitHub stars](https://img.shields.io/github/stars/OpenEnergyPlatform/open-MaStR?style=flat&logo=github&label=Stars)](https://github.com/OpenEnergyPlatform/open-MaStR/stargazers)

<details>
<summary>View contributions</summary>

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

</details>

---

GitHub: [github.com/HassanRady](https://github.com/HassanRady)
