# Small Check Prompt Group

Role: You are a QA architect running a fast non-security system check. Read `../wiki/{target-project-name}/00-single-source-of-truth.md` before starting. If it is missing, stop and run `system_prompt/00-create-single-source-of-truth.md`.

Run the following 12 primary check prompts. Keep each prompt isolated and save the merged result to `prompts/result/check/small/result-small-check.md`.

## Output Rules

- Do not modify code.
- Save only to `prompts/result/check/small/`.
- Every finding must include: **category** | **file:line** | **evidence snippet** | **impact** | **severity** | **recommended fix**
- If a finding is primarily security-related, move it to Scan.
- Record a PASS note for any prompt with zero findings.

### Severity Levels
| Level | Criteria |
|-------|----------|
| CRITICAL | Blocks deployment or causes data loss |
| HIGH | Breaks core workflow or violates documented behavior |
| MEDIUM | Degrades experience, maintainability, or minor inconsistency |
| LOW | Cosmetic, docs-only, or future risk |

---

## Prompt 01 - Functional Baseline

**Goal:** Validate core user workflows match the source of truth.

### Sub-checks
- [ ] Happy path: each documented workflow completes end-to-end
- [ ] Boundary inputs: empty, null, max-length, special chars
- [ ] Error states: 4xx/5xx responses with meaningful messages
- [ ] Idempotency: repeating safe requests produces same result
- [ ] Observable side effects: DB writes, external calls, log output

**Search patterns:**
- Route definitions: `grep -rn "Route\|router\.\(get\|post\|put\|delete\)" --include="*.py" --include="*.go" --include="*.ts" --include="*.java" --include="*.cs"`
- Handler implementations: match routes to handler functions
- Test coverage: count tests per endpoint vs documented workflows

---

## Prompt 02 - Logic Consistency

**Goal:** Check business rules, state transitions, calculations, and permissions against the source of truth.

### Sub-checks
- [ ] Business rules match documented requirements
- [ ] State machines have complete transitions (no dead states)
- [ ] Calculations produce correct numeric/date results
- [ ] Permission checks enforce documented role rules
- [ ] No contradictory conditions (e.g., A > B AND B > A)

**Search patterns:**
- `grep -rn "if\|switch\|case\|state\|status" --include="*.{py,ts,go,java,cs}"`
- `grep -rn "enum\|const.*=.*{" --include="*.{ts,go,java}"`
- `grep -rn "permission\|role\|authorize\|can_access" --include="*.{py,ts}"

---

## Prompt 03 - Configuration Consistency

**Goal:** Review environment variables, defaults, config files, feature flags, build modes.

### Sub-checks
- [ ] Required env vars are documented and validated at startup
- [ ] Default values in code match documented defaults
- [ ] Feature flags have documented purpose and removal criteria
- [ ] Local/dev/prod configs are structurally aligned (no drift)
- [ ] Config validation rejects invalid combinations early

**Search patterns:**
- `grep -rn "env\|os\.getenv\|process\.env\|config\." --include="*.{py,ts,yaml,yml,toml,json}"`
- Config files: `**/.env*`, `**/config.*`, `**/appsettings.*`, `**/application.*`

---

## Prompt 04 - Workflow Integrity

**Goal:** Trace important workflows from entry point to persistence/external side effects.

### Sub-checks
- [ ] Workflow entry → handler → business logic → persistence → response is complete
- [ ] Rollback/compensation exists for multi-step transactions
- [ ] Status transitions are complete (no gaps like `pending → done` skipping `processing`)
- [ ] Duplicate submission is prevented (idempotency keys, txn dedup)
- [ ] Timeout and retry logic covers external calls

**Search patterns:**
- `grep -rn "transaction\|rollback\|commit\|compensat" --include="*.{py,ts,go,java,cs}"`
- `grep -rn "status\|state.*flow\|workflow\|pipeline" --include="*.{py,ts,yaml}"`
- `grep -rn "idempot\|dedup\|unique.*key\|retry" --include="*.{py,ts,go}"`

---

## Prompt 05 - System Structure

**Goal:** Check folder layout, module boundaries, naming, dependency direction.

### Sub-checks
- [ ] Directory structure matches documented architecture
- [ ] No circular dependencies between modules
- [ ] Module boundaries respected (no cross-layer imports)
- [ ] File/module naming is consistent (same convention everywhere)
- [ ] Dead/unused modules, files, or directories identified

**Search patterns:**
- `Get-ChildItem -Recurse -Directory | Select-Object FullName` (folder layout)
- Module imports: `grep -rn "from\|import\|require\|using\|include" --include="*.{py,ts,go,java,cs}"`
- Dead code: search for files not referenced in any import/require statement

---

## Prompt 06 - API Contract and Integration

**Goal:** Compare handlers, clients, schemas, and status codes.

### Sub-checks
- [ ] Server handlers match API schema/OpenAPI spec
- [ ] Client requests match server's expected request shape
- [ ] Response status codes cover: 200, 201, 204, 400, 401, 403, 404, 409, 422, 500
- [ ] Request/response schemas are validated (not just typed)
- [ ] Integration points (webhooks, external APIs) are documented

**Search patterns:**
- `grep -rn "status\|HTTPStatus\|StatusCode" --include="*.{py,ts,go,java,cs}"`
- `grep -rn "openapi\|swagger\|schema\|@Api" --include="*.{yaml,yml,json,py,ts}"`
- Validation: `grep -rn "validate\|pydantic\|zod\|class-validator\|fluentValidation" --include="*.{py,ts,cs}"`

---

## Prompt 07 - Data Integrity and Backup

**Goal:** Review schema constraints, migrations, seed data, backup/restore.

### Sub-checks
- [ ] Unique constraints, foreign keys, NOT NULL exist where expected
- [ ] Migrations are reversible (have `down` or rollback)
- [ ] Seed data matches documented defaults/test data
- [ ] Backup/restore documentation exists and is testable
- [ ] Data lifecycle (archival, purge, retention) defined

**Search patterns:**
- Migrations: `**/migrations/**`, `**/alembic/**`, `**/prisma/**`
- `grep -rn "unique\|foreign\|not null\|check\|index" --include="*.{sql,py,ts,prisma}"`
- `grep -rn "backup\|restore\|dump\|pg_dump\|mysqldump"`

---

## Prompt 08 - Deployment Readiness

**Goal:** Check Docker/compose, build scripts, runtime prerequisites.

### Sub-checks
- [ ] Dockerfile/Compose files present and build without error
- [ ] Build scripts (`build.sh`, `Makefile`, `package.json` scripts) work
- [ ] Runtime prerequisites documented (Node version, DB, Redis, etc.)
- [ ] Ports, volumes, env vars in compose match code expectations
- [ ] Startup health check or readiness probe exists

**Search patterns:**
- `**/Dockerfile*`, `**/docker-compose*`, `**/Dockerfile.*`
- `grep -rn "EXPOSE\|ports\|environment\|volumes" --include="*.{yml,yaml,compose}"`
- `grep -rn "healthcheck\|health_check\|readiness\|liveness" --include="*.{yml,yaml,py,ts,go}"`

---

## Prompt 09 - Performance Smoke Check

**Goal:** Look for obvious slow queries, unbounded loops, missing pagination.

### Sub-checks
- [ ] Database queries inside loops (N+1 pattern)
- [ ] No pagination on list endpoints (unbounded results)
- [ ] Synchronous blocking calls in async contexts
- [ ] Large file/bundle size without lazy loading
- [ ] Inefficient polling patterns (poll interval < 1s without reason)
- [ ] Unbounded loops or recursion without exit condition

**Search patterns:**
- `grep -rn "for.*in\|\.forEach\|while\|\.map(" --include="*.{py,ts}"` then check if DB inside
- `grep -rn "\.find\|\.all\|select.*\*\|SelectAll\|GetAll" --include="*.{py,ts,go,java,cs}"` + check pagination
- `grep -rn "await.*sync\|sync.*over.*async\|Task\.Wait\|\.Result" --include="*.{py,cs}"`
- Bundle: check `dist/`, `build/` sizes, webpack/rollup config

---

## Prompt 10 - Logging and Observability

**Goal:** Check application logs, health checks, metrics, traceability.

### Sub-checks
- [ ] Structured logs (JSON format, not plain text or print)
- [ ] Correlation ID/trace ID passed across service boundaries
- [ ] Error context includes request ID, user, and stack trace
- [ ] Health check endpoint returns meaningful status
- [ ] Metrics exported (request count, latency, error rate, memory)
- [ ] Log volume is bounded (no logging of sensitive data or large payloads)

**Search patterns:**
- `grep -rn "log\|logger\|logging\|console\.log\|fmt\.Print\|print(" --include="*.{py,ts,go,java,cs}"`
- `grep -rn "structlog\|loguru\|serilog\|log4j\|winston\|pino" --include="*.{py,ts,cs}"`
- `grep -rn "health\|/health\|/ready\|/live\|metrics\|/metrics" --include="*.{py,ts,yaml,yml}"`
- `grep -rn "correlation\|trace_id\|span_id\|request_id" --include="*.{py,ts,go}"`

---

## Prompt 11 - Code Quality and Maintainability

**Goal:** Review duplication, complexity, TODO debt, linting, testing setup.

### Sub-checks
- [ ] Duplicate code blocks > 15 lines (similar logic repeated)
- [ ] Functions with cyclomatic complexity > 10
- [ ] TODOs/FIXMEs/HACKs older than 6 months (check git blame)
- [ ] Dead/unused code (exports/imports not consumed)
- [ ] Linter/formatter config present and passing
- [ ] Test framework configured and tests are runnable

**Search patterns:**
- `grep -rn "TODO\|FIXME\|HACK\|XXX\|WORKAROUND" --include="*.{py,ts,go,java,cs}"`
- `grep -rn "if\|else\|elif\|case\|switch" --include="*.{py,ts,go,java,cs}"` (complexity estimate)
- Lint config: `**/.eslintrc*`, `**/.pylintrc`, `**/ruff.toml`, `**/goimports`
- Test config: `**/jest.config*`, `**/pytest.ini`, `**/*.test.*`, `**/*.spec.*`

---

## Prompt 12 - Usability Baseline

**Goal:** Check navigation, forms, validation feedback, error states.

### Sub-checks
- [ ] Navigation: all pages reachable, no broken links
- [ ] Input validation: client-side + server-side matching
- [ ] Error messages: human-readable, not technical stack traces
- [ ] Empty state: "no data" message present, not blank screen
- [ ] Loading state: spinner/skeleton during async operations
- [ ] Accessibility basics: form labels, alt text, keyboard navigation
- [ ] Task completion: user can achieve goal in minimal steps

**Search patterns:**
- `grep -rn "required\|validation\|validate\|pattern\|min\|max" --include="*.{html,tsx,jsx,vue,svelte}"`
- `grep -rn "error\|message\|toast\|alert\|notification" --include="*.{tsx,jsx,vue,svelte}"`
- `grep -rn "loading\|spinner\|skeleton\|placeholder" --include="*.{tsx,jsx,vue,svelte}"`
- `grep -rn "aria-\|role=\|alt=\|label" --include="*.{html,tsx,jsx,vue,svelte}"`
