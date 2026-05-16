You are a software architecture expert. Perform a **deep, verifiable analysis** of the repository (source code, documentation, configuration, test artifacts, DB migrations, logs, deployment files, etc.) and produce a **"Single Source of Truth"** document. 

In addition to documenting the current state, you MUST answer three strategic questions based on evidence from the codebase:

1. **Deployment profile** – Is this system designed for **local individual use** (e.g., desktop app, single-user CLI, localhost web app) or **enterprise use** (multi-user, high availability, compliance, SSO, audit logs)? Justify with specific evidence (e.g., presence of user management, authentication complexity, scalability patterns, deployment configs like Kubernetes vs docker-compose).
2. **Multi-tenant requirement** – Does the system currently need or implement multi-tenancy? If not, should it be added based on the intended users/scale? Provide evidence (e.g., `tenant_id` columns, schema separation, isolation logic, or lack thereof).
3. **Architecture style** – Should the system remain **monolith** or evolve to **microservices**? Base your answer on current modularity, team size, deployment frequency, database coupling, and communication patterns (sync/async). Give a clear recommendation with trade-offs.

## CRITICAL RULES
1. **Data extraction must be real** – Run or simulate analysis commands (e.g., `cloc`, `grep`, `find`, `pytest --collect-only`, `ls -la`) and embed **actual counts** (file sizes, line counts, test counts, migration versions, dependency versions, API endpoint counts). Do NOT make up numbers.
2. **Conflict resolution** – When code and legacy docs disagree, **code wins**. Log every discrepancy in a dedicated `## Discrepancies Log` section with file paths and dates.
3. **Be path-specific** – Reference actual absolute/relative paths (`/backend/src/services/payment.py:124-150`). Use module status icons: ✅ COMPLETE, ⚠️ PARTIAL, ❌ MISSING/PLACEHOLDER, 🚧 IN PROGRESS (with PR link if available), 🔄 DEPRECATED.
4. **Include "How to verify"** – For each major claim (e.g., “User roles are RBAC”), add a small verification snippet (e.g., `grep -r "role" backend/auth/` or a direct file reference).
5. **Roadmap prerequisites** – For each upgrade phase, list blocking conditions (e.g., “Requires migration v42 applied”, “Needs Redis 7.0+”).

## Output structure (strict Markdown, to be saved as `wiki/00-source-of-truth.md`)

# Single Source of Truth – [Project Name]

## 0. Document Metadata
- **Last updated** (auto from git head commit date)
- **Git commit hash**
- **Analysis tool versions** (cloc, jq, pytest, etc.)

## 1. System Objective
- Problem statement (max 3 sentences)
- Key features (bullet list, each mapped to a module or file)
- Non-goals (explicitly out of scope)

## 2. System Scope
- **Tech stack table** (layer → technology → version → config file path)
- **Modules in scope** (list with status icon)
- **Modules explicitly out of scope** (with reason)
- **Constraints** (e.g., “Must support 500 concurrent users”, “Runs only on Ubuntu 20.04”)

## 3. Users and Roles
- Role matrix (role → permissions → enforced in code? → file location)
- Authentication mechanism (OAuth2/JWT/LDAP – with exact library and config)
- Statuses (active/suspended/invited – where defined)
- Target improvements (e.g., “Add SSO by Q3”)

## 4. Main Modules
| Module | Status | Backend files (path) | Frontend files (path) | API prefix | DB tables | SaaS notes |
|--------|--------|----------------------|-----------------------|------------|-----------|-------------|
| ...    | ✅     | `...` (LOC: 234)     | `...` (LOC: 89)       | `/api/v1/x`| `x,y,z`   | Multi-tenant? |

## 5. Core Business Workflows
For each workflow (login, scheduling, task execution, etc.):
- **Step-by-step** (with file:line references)
- **Alternate flows** (error handling, timeout)
- **State transitions** (as Mermaid state diagram)
- **Verification** (e.g., `curl -X POST ...` or test name)

## 6. Architecture (Current & Target)
- **Current** – Mermaid diagram (C4 level 2), component relationships, data flow.
- **Technology stack comparison table** (current vs target: language, framework, DB, message queue, observability).
- **Deployment** (Docker? K8s? Nomad? – actual compose files or manifests).

## 7. Strategic Recommendations (Evidence-Based)
This section answers the three mandatory questions.

### 7.1 Deployment Profile: Local/Personal vs Enterprise
- **Verdict:** (e.g., "Enterprise" or "Local/Personal" or "Hybrid")
- **Evidence:** List specific findings:
  - User management complexity (simple password vs OIDC/LDAP/SSO)
  - Concurrency handling (e.g., async workers, connection pooling settings)
  - Deployment artifacts (docker-compose for single node vs Helm charts for k8s)
  - Compliance features (audit logs, data retention policies)
  - Scalability indicators (horizontal scaling configs, load balancer)
- **Recommendation:** (if misaligned, suggest changes)

### 7.2 Multi-Tenancy: Required or Not?
- **Verdict:** (e.g., "Currently not multi-tenant, but should be added" or "Multi-tenant already implemented" or "Not needed for foreseeable future")
- **Evidence:**
  - Database schema: any `tenant_id`, `org_id`, `workspace_id` columns? Shared tables or separate databases?
  - Code patterns: middleware checking tenant context? Row-level security?
  - Business requirements inferred from features (e.g., teams, organizations, billing)
- **If needed:** Propose isolation level (row-level, schema-per-tenant, db-per-tenant) and estimated effort.

### 7.3 Architecture Style: Monolith vs Microservices
- **Verdict:** (e.g., "Keep Monolith" or "Split into Microservices" or "Modular Monolith first")
- **Evidence:**
  - Module coupling (import cycles, shared database tables, internal API calls)
  - Team size and structure (from git history, number of active contributors)
  - Deployment frequency (CI config, ability to deploy parts independently)
  - Communication patterns (sync REST vs async message queue)
  - Current performance bottlenecks (if any)
- **Recommendation:** Provide pragmatic steps (e.g., "Stay monolith until 5+ teams or >100 endpoints; then extract payment service").

## 8. Roadmap / Upgrade Plan (Phased)
| Phase | Goal | Changes (file/scope) | Prerequisites | Owner | Estimated effort | Success metric |
|-------|------|----------------------|---------------|-------|------------------|----------------|

## 9. Current System Health
- **Test metrics** (total, passed, failed, skipped, coverage % per module)
- **Build status** (CI badge, last 5 runs)
- **Migration status** (latest migration version, pending migrations, rollback risk)
- **Known blockers** (P1: blocks release, P2: workaround exists) – with links to issues
- **Resolved issues** (last 30 days, notable ones)
- **Technical debt** (list with file:line, type: duplication, TODOs > 6 months, complex cyclomatic > 10)
- **Dependencies** (outdated packages, vulnerabilities – `npm audit` / `pip-audit` summary)

## 10. Development Rules (Must read before changing code)
- Code style (linter config path, pre-commit hooks)
- Testing requirements (min coverage per module, which tests to run)
- Branch strategy (`git flow`? `trunk`? example)
- PR review checklist (link to `CONTRIBUTING.md` or inline)
- Environment setup (exact commands to run dev environment)

## 11. Design Constraints (for future development)
- **Multi-tenant** (isolation level: schema/row/database – file implementing it)
- **Storage** (max file size, backup policy, encryption at rest – config location)
- **Security** (rate limiting config, CORS origins, auth timeout)
- **Reliability** (retry policies, circuit breakers, health check endpoint)
- **Observability** (logs, metrics, traces – actual exporters: Prometheus, Jaeger)
- **Frontend** (browser support, bundle size limit, SSR/CSR)

## 12. Discrepancies Log (code vs docs)
| Date | Location (doc vs code) | Discrepancy | Resolution (code wins) | Fixed in commit |
|------|------------------------|-------------|------------------------|------------------|

## 13. Appendix – How to regenerate this document
Provide a one-liner (e.g., `make source-of-truth`) that re‑runs all data extraction scripts.