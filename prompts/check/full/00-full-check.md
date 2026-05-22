# Full Check Prompt Group

Role: You are a full-scope QA architect running comprehensive non-security checks. Read `../wiki/00-single-source-of-truth.md` before starting. If it is missing, stop and run `system_prompt/00-create-single-source-of-truth.md`.

Save the merged output to `prompts/result/check/full/result-full-check.md`.

## Output Rules

- Do not modify code.
- Save only to `prompts/result/check/full/`.
- Every finding must include: **category** | **file:line** | **evidence snippet** | **impact** | **severity** | **recommended fix**
- Keep security findings in Scan. This file covers system correctness, quality, and product readiness.
- Record a PASS note for any check with zero findings.

### Severity Levels
| Level | Criteria |
|-------|----------|
| CRITICAL | Blocks deployment, causes data loss, or breaks SLA |
| HIGH | Breaks core workflow, violates documented behavior, or affects paying users |
| MEDIUM | Degrades experience, maintainability, or minor inconsistency |
| LOW | Cosmetic, docs-only, low-probability edge case, or future risk |
| INFO | Observation, suggestion, or non-blocking improvement |

---

## Full Check Prompts

### 1. Functional Completeness

**Goal:** Validate all documented core features and acceptance paths against implementation.

- [ ] Every documented feature has a corresponding route/handler/module
- [ ] Acceptance criteria (from PRDs, tickets, specs) are verifiable via code
- [ ] Feature flags can toggle documented behavior correctly (on/off)
- [ ] Pagination, filtering, sorting work for list endpoints
- [ ] File download/upload functionality works for supported formats
- [ ] Batch operations process all items (not just first N)

**Search:** `grep -rn "route\|router\.\(get\|post\|put\|delete\|patch\)" --include="*.{py,ts,go,java,cs}"` then map to docs.

---

### 2. Business Logic

**Goal:** Verify calculations, state transitions, domain invariants, and edge cases.

- [ ] Pricing, discounts, tax calculations are numerically correct
- [ ] Date/time calculations handle timezone, DST, and leap year
- [ ] Status transitions follow defined state machine (no illegal moves)
- [ ] Domain invariants enforced (e.g., `end_date > start_date`, `quantity >= 0`)
- [ ] Rounding behavior is consistent (banker's rounding, truncation, etc.)
- [ ] Conditional logic covers all branches (no hidden else-path)

**Search:** `grep -rn "price\|cost\|tax\|discount\|total\|amount" --include="*.{py,ts,go,java,cs}"`; look for numeric operations.

---

### 3. Workflow Orchestration

**Goal:** Trace multi-step user and background workflows end to end.

- [ ] Multi-step wizards preserve state between steps
- [ ] Background jobs (queues, cron, schedulers) handle failure and retry
- [ ] Long-running workflows have timeout and graceful cancellation
- [ ] Async workflows emit completion notifications (email, webhook, push)
- [ ] Saga/compensation transactions roll back all steps on failure
- [ ] Step ordering is deterministic and race-free

**Search:** `grep -rn "queue\|job\|worker\|celery\|bull\|hangfire\|sidekiq\|cron\|schedule" --include="*.{py,ts,cs,rb}"`; `grep -rn "workflow\|saga\|step\|stage\|phase" --include="*.{py,ts}"`

---

### 4. API Contract

**Goal:** Compare server handlers, client calls, schemas, and status codes.

- [ ] OpenAPI spec matches actual request/response shapes (test each endpoint)
- [ ] Client SDKs/API calls match server expectations (field names, types, nesting)
- [ ] Status codes used correctly: 200 (OK), 201 (Created), 204 (No Content), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 409 (Conflict), 422 (Unprocessable), 429 (Rate Limited), 500 (Server Error), 503 (Service Unavailable)
- [ ] Error response bodies have consistent structure (`{error, message, code, details}`)
- [ ] Deprecated endpoints return `Sunset` or `Deprecation` header
- [ ] Versioning strategy is consistent (URL path, header, or accept header)

**Search:** `grep -rn "200\|201\|204\|400\|401\|403\|404\|409\|422\|429\|500\|503" --include="*.{py,ts,go,java,cs}"`; `**/*openapi*`, `**/*swagger*`, `**/*schema*`

---

### 5. Integration Behavior

**Goal:** Verify external services, queues, webhooks, retries, and failure modes.

- [ ] External API calls have timeout (default <= 30s) and circuit breaker
- [ ] Webhook delivery has retry with exponential backoff + dead letter
- [ ] Message queue consumption is idempotent and order-respecting
- [ ] 3rd-party service degradation is handled gracefully (fallback, cached response)
- [ ] Credentials for external services are not hardcoded (use secrets/ env)
- [ ] Integration tests can run against test doubles (mocks/stubs/sandboxes)

**Search:** `grep -rn "http\|fetch\|axios\|requests\|HttpClient\|WebClient\|RestTemplate" --include="*.{py,ts,cs,java}"`; `grep -rn "webhook\|callback\|event.*emit\|notify" --include="*.{py,ts}"`

---

### 6. Configuration

**Goal:** Check env vars, defaults, feature flags, config validation, and local/prod drift.

- [ ] All required env vars are documented and validated at startup (fail-fast)
- [ ] Default config values are safe for production (no debug mode, no weak secrets)
- [ ] Feature flags have description, owner, and removal criteria (technical debt item)
- [ ] Local/dev/staging/prod configs are structurally identical (only values differ)
- [ ] Secret injection (env vars, vault, K8s secrets) is not readable in logs
- [ ] Config changes can be applied without code deployment (runtime reloadable)

**Search:** `grep -rn "env\|os\.getenv\|process\.env\|config\|settings\|appsettings" --include="*.{py,ts,yaml,yml,json,cs}"`; `**/.env*`, `**/appsettings.*`

---

### 7. Architecture Structure

**Goal:** Inspect module boundaries, coupling, dependency direction, and layering.

- [ ] Layering is respected: presentation → application → domain → infrastructure (no skipping)
- [ ] No circular dependencies between packages/modules/services
- [ ] Internal packages are not exposed as public API
- [ ] Shared libraries have stable interfaces (semantic versioning or internal compat)
- [ ] Plugin/module system has documented extension points
- [ ] Database access is centralized (no raw queries in controllers/views)
- [ ] Event/message contracts are versioned

**Search:** `Get-ChildItem -Recurse -Directory -Depth 2` (module map); `grep -rn "import\|require\|from\|using\|include" --include="*.{py,ts,go,java,cs}"`; check for cycles with `ts-sort`/`go mod graph`

---

### 8. Data Integrity

**Goal:** Review schema constraints, migrations, transactions, data repair, and lifecycle.

- [ ] Database schema constraints match domain invariants (unique, FK, CHECK, NOT NULL)
- [ ] Migrations are reversible (have down/rollback) and tested in CI
- [ ] Transactions are scoped correctly (all-or-nothing for related writes)
- [ ] Soft-delete has cascade/migration/restore logic, not just `is_deleted = true`
- [ ] Audit fields present: `created_at`, `updated_at`, `created_by`, `updated_by`
- [ ] Data repair/backfill scripts exist for corrupted states
- [ ] Orphaned records are cleaned up (cascade delete or scheduled job)

**Search:** `**/migrations/**`, `**/prisma/schema.prisma`, `**/*.sql`, `**/alembic/**`; `grep -rn "transaction\|commit\|rollback" --include="*.{py,ts,go,java,cs}"`

---

### 9. Backup and Restore

**Goal:** Verify backup documentation, restore commands, retention, and disaster recovery.

- [ ] Backup strategy is documented (full, incremental, WAL archiving)
- [ ] Restore procedure exists and is tested (RTO/RPO documented)
- [ ] Retention policy is configured (log rotation, backup expiry)
- [ ] Disaster recovery runbook covers: DB restore, file restore, config restore
- [ ] Backups are stored separately from production data (different region/bucket)
- [ ] Backup encryption and access control are configured

**Search:** `grep -rn "backup\|restore\|dump\|pg_dump\|mysqldump\|mongodump\|velero\|restic\|borg"`; `**/docs/**disaster*`, `**/docs/**dr*`, `**/docs/**runbook*`

---

### 10. Reliability and Stability

**Goal:** Inspect timeouts, retries, circuit breakers, crash behavior, and graceful degradation.

- [ ] All network calls have timeouts (connect, read, write) with sensible defaults
- [ ] Retry policy exists for transient failures (with jitter, cap, and backoff)
- [ ] Circuit breaker configured for critical external dependencies
- [ ] Crash recovery: process restarts automatically (Docker restart, supervisor)
- [ ] Graceful shutdown: ongoing requests drain before process exits
- [ ] Out-of-memory protection: pagination, streaming, or limits on large datasets
- [ ] Rate limiter responses include `Retry-After` header

**Search:** `grep -rn "timeout\|retry\|circuit.*breaker\|hystrix\|resilience\|polly" --include="*.{py,ts,go,java,cs}"`; `grep -rn "SIGTERM\|SIGINT\|graceful_shutdown\|shutdown" --include="*.{py,ts,go}"`

---

### 11. Scalability

**Goal:** Review pagination, queues, concurrency model, horizontal scaling assumptions.

- [ ] All list endpoints support cursor- or offset-based pagination with configurable page size
- [ ] Background processing uses work queues (not inline, not threads)
- [ ] Concurrency model is explicit (async/await, goroutines, threads) with controlled pool sizes
- [ ] Horizontal scaling: stateless app instances, shared session store, distributed cache
- [ ] Database connection pooling is configured (not 1 conn/request)
- [ ] No shared-memory state across instances (use Redis/DB for coordination)
- [ ] Load testing results available or performance baselines established

**Search:** `grep -rn "page\|limit\|offset\|cursor\|pagination\|pageSize" --include="*.{py,ts,go,java,cs}"`; `grep -rn "ThreadPool\|max_connections\|pool_size\|concurrency" --include="*.{py,ts,go,java,cs}"`

---

### 12. Performance

**Goal:** Inspect hot paths, query efficiency, caching, bundle size, blocking work, and memory risks.

- [ ] Hot path endpoints are identified and optimized (< 200ms p95)
- [ ] Database queries have indexes on WHERE/ORDER/JOIN columns (check `EXPLAIN`)
- [ ] N+1 query pattern absent (check ORM usage in loops)
- [ ] Caching strategy defined for expensive operations (Redis, in-memory, CDN)
- [ ] Frontend bundle has tree-shaking, code splitting, and lazy loading
- [ ] No synchronous blocking I/O in async contexts (thread pool starvation risk)
- [ ] Large file processing uses streaming, not loading everything into memory
- [ ] Missing index detection: search for sequential scans in query patterns

**Search:** `grep -rn "\.all()\|\.find()\|select\|from\|left_join\|inner_join" --include="*.{py,ts}"` inside loops; `grep -rn "cache\|redis\|memcached\|CDN" --include="*.{py,ts,yaml,yml}"`; check `package.json` bundle size

---

### 13. Logging

**Goal:** Verify structured logs, correlation IDs, error context, and log volume.

- [ ] Log format is structured (JSON) and machine-parseable
- [ ] Every request has a correlation ID propagated through the entire chain
- [ ] Error logs include: stack trace, user ID, request ID, input context (sanitized)
- [ ] Log level configuration is runtime-adjustable (debug on demand)
- [ ] No sensitive data in logs: passwords, tokens, PII, credit cards
- [ ] Log volume is bounded: no per-item logging inside batch loops > 100 items
- [ ] Startup/shutdown events are logged with version and config fingerprint

**Search:** `grep -rn "log\|logger\|logging\|console\.log\|fmt\.Print\|print(" --include="*.{py,ts,go,java,cs}"`; `grep -rn "correlation\|trace_id\|span_id\|request_id\|transaction_id" --include="*.{py,ts,go}"`; `grep -rn "password\|secret\|token\|ssn\|credit.*card" --include="*.py" | grep -v "hash\|encrypt"`

---

### 14. Observability

**Goal:** Check health checks, metrics, tracing, alertable failure signals, and dashboards.

- [ ] Health endpoint: `/health` returns overall + dependency status (DB, Redis, queue)
- [ ] Readiness probe: `/ready` returns true only when fully initialized
- [ ] Liveness probe: `/live` returns false only when process is stuck/deadlocked
- [ ] Metrics exported (Prometheus, StatsD, OpenTelemetry): request count, latency (p50/p95/p99), error rate, memory/CPU, goroutines
- [ ] Distributed tracing configured (Jaeger, Zipkin, OpenTelemetry) for key workflows
- [ ] Alerts exist for: error rate spike, p95 latency spike, 5xx > 1%, disk nearly full, queue backlog
- [ ] Dashboard (Grafana, DataDog) view available for main SLOs

**Search:** `grep -rn "health\|/health\|/ready\|/live\|/metrics" --include="*.{py,ts,go,yaml,yml}"`; `grep -rn "prometheus\|opentelemetry\|statsd\|datadog\|newrelic" --include="*.{py,ts,yaml,yml}"`; `grep -rn "alert\|alarm\|pager\|slack.*webhook\|pagerduty" --include="*.{yaml,yml,json}"`

---

### 15. Testing

**Goal:** Inspect unit, integration, e2e, fixtures, coverage, flakiness, and missing critical tests.

- [ ] Unit tests cover business logic, state transitions, calculations, and edge cases
- [ ] Integration tests cover DB, external API, and message queue interactions
- [ ] E2E tests cover critical user journeys (login → work → logout)
- [ ] Test fixtures are versioned and deterministic (no shared mutable fixtures)
- [ ] Code coverage meets team threshold (config in CI) — check for uncovered critical paths
- [ ] Flaky tests are identified and quarantined (retry mechanism or skip list)
- [ ] Performance/load tests exist for SLAs
- [ ] Test execution is parallelized and completes < 10 minutes

**Search:** `**/*.test.*`, `**/*.spec.*`, `**/tests/**`, `**/__tests__/**`; `grep -rn "coverage\|istanbul\|pytest-cov\|jacoco\|coverlet"`; `grep -rn "flaky\|retry\|quarantine\|skip.*flaky" --include="*.{py,ts,java}"`

---

### 16. CI/CD Quality

**Goal:** Review build/test/release gates, artifact handling, versioning, rollback, and environment promotion.

- [ ] CI pipeline runs linter → unit tests → integration tests → build → security scan
- [ ] Release gates: all tests pass, no critical vulns, manual approval for prod
- [ ] Artifacts are versioned (semver or git sha) and immutable once published
- [ ] Rollback plan: previous version can be redeployed within minutes
- [ ] Environment promotion: dev → staging → prod with consistent artifact (no rebuild)
- [ ] Database migrations run as separate step (not in app startup)
- [ ] Pipeline secrets are stored in CI secrets manager (not in plaintext)

**Search:** `**/.github/workflows/**`, `**/.gitlab-ci.yml`, `**/Jenkinsfile`, `**/azure-pipelines.yml`, `**/ci/**`; `grep -rn "semver\|version\|tag\|release\|rollback" --include="*.{yml,yaml,json}"`

---

### 17. Documentation

**Goal:** Compare README, setup docs, API docs, architecture docs, and code behavior.

- [ ] README: install, configure, run, test, build, deploy steps are correct
- [ ] API docs match actual endpoints, request/response shapes (no drift)
- [ ] Architecture docs reflect current module structure and data flow
- [ ] Onboarding docs (CONTRIBUTING, DEVELOPMENT) are complete
- [ ] Changelog tracks breaking changes, new features, and fixes
- [ ] Inline code documentation (docstrings, comments) exists for public APIs
- [ ] Security policy / responsible disclosure documented

**Search:** `**/README*`, `**/CONTRIBUTING*`, `**/CHANGELOG*`, `**/docs/**`, `**/*.md`; `grep -rn "TODO\|FIXME\|HACK" --include="*.{py,ts,go,java,cs}"` (undocumented debt)

---

### 18. Usability

**Goal:** Review navigation, forms, error states, accessibility basics, and user task completion.

- [ ] Navigation is intuitive: clear labels, consistent layout, breadcrumbs
- [ ] Form validation: client-side + server-side, inline errors, clear messages
- [ ] Error states: 404, 403, 500 pages are styled and helpful (not white screen)
- [ ] Empty states: no-data messages, call to action, or onboarding prompt
- [ ] Loading states: skeleton/spinner, optimistic UI where appropriate
- [ ] Accessibility: form labels (`<label for>`), alt text on images, role attributes, keyboard navigation, focus management
- [ ] Mobile responsive: layout adapts, touch targets >= 44px
- [ ] Task completion: most common task can be done in <= 3 clicks

**Search:** `grep -rn "<label\|<input\|required\|validate\|form" --include="*.{html,tsx,jsx,vue,svelte}"`; `grep -rn "404\|403\|500\|error.*page\|not.*found" --include="*.{html,tsx,jsx,vue,svelte}"`; `grep -rn "aria-\|role=\|alt=\|tabindex\|focus" --include="*.{html,tsx,jsx,vue,svelte}"`

---

### 19. Cost and Resource Optimization

**Goal:** Inspect resource sizing, expensive jobs, storage growth, and unnecessary external calls.

- [ ] Compute resources (CPU/memory requests, limits) are right-sized for expected load
- [ ] Expensive cron/background jobs are scheduled off-peak
- [ ] Storage growth rate is monitored and has an archival strategy
- [ ] Unnecessary external API calls eliminated (batch when possible, cache responses)
- [ ] Idle resources identified (stale pods, unattached volumes, unused instances)
- [ ] Database query cost: missing indexes causing full scans on large tables (>100K rows)
- [ ] Log/event retention costs: aggressive retention may be over-provisioned

**Search:** `grep -rn "resources\|limits\|requests\|memory\|cpu" --include="*.{yml,yaml}"`; `grep -rn "cron\|schedule\|job\|worker" --include="*.{py,ts,yaml,yml}"`; check DB table sizes

---

### 20. Maintainability

**Goal:** Review duplication, complexity, naming, dead code, TODOs, and refactor hotspots.

- [ ] Duplicate code blocks > 15 lines: extract to shared function/module
- [ ] Functions/methods with cyclomatic complexity > 15: refactor into smaller units
- [ ] Consistent naming: PascalCase for classes, camelCase for variables/functions, UPPER_CASE for constants
- [ ] Dead code: exported symbols/variables not imported/referenced elsewhere
- [ ] TODOs/FIXMEs/HACKs are tracked (git grep) and have associated issues
- [ ] Long files > 500 lines: consider splitting by concern
- [ ] Deep nesting > 4 levels: early return, guard clauses, extraction
- [ ] Hardcoded values (magic numbers, strings): extract to constants or config

**Search:** `grep -rn "TODO\|FIXME\|HACK\|XXX\|WORKAROUND\|BUG" --include="*.{py,ts,go,java,cs}"`; `grep -rn "^\s*if.*if.*if" --include="*.{py,ts}"` (nesting); count lines per file with `Measure-Object`

---

## Final Report Sections

The merged output must contain:

### Metadata
- Target project name and version
- Source of truth revision (commit hash)
- Date of check run
- Tool versions used

### Executive Summary
- Total findings: CRITICAL / HIGH / MEDIUM / LOW / INFO
- Top 3 most critical issues
- Overall health score (PASS / WARN / FAIL)

### Findings Table (sorted by severity)
| # | Category | Severity | File:Line | Description | Impact | Verification | Recommended Fix |
|---|----------|----------|-----------|-------------|--------|--------------|-----------------|

### Critical Defects
Detailed section for each CRITICAL finding: reproduction steps, root cause, impact scope.

### High Priority Defects
Detailed section for each HIGH finding.

### Medium and Low Priority Findings
Grouped lists with brief descriptions.

### Verification Commands Run
Exact commands executed (grep, find, test, etc.)

### Coverage Gaps
Areas not checked (e.g., "No mobile client to verify", "No load test results available")

### Prioritized Remediation Roadmap
Recommended order of fixes with estimated effort (XS/S/M/L/XL) and owner.
