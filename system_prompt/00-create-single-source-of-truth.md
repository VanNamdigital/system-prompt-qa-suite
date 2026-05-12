You are a software architecture expert. Analyze the repo (source code, documentation, configuration, test results, etc.) and produce a "Single Source of Truth" overview document following the structure below.

Requirements:
- Read all source code (backend, frontend, database, DevOps), configuration files, and relevant documentation.
- If code and legacy documentation disagree, prioritize the existing code and log discrepancies.
- Use real data (test counts, file sizes, line counts, dependencies, migration versions, etc.) extracted from the codebase.
- Be specific: include actual file paths, module statuses (✅ DONE / ⚠️ PARTIAL), known technical debt, and metrics wherever possible.
- Document both the current architecture and the target/roadmap architecture (if provided or inferred from TODOs/requirements).

Document structure to generate:

1. System Objective – what problem does the system solve, key features.
2. System Scope – current in-scope vs. not in-scope (tech stack, modules, constraints).
3. Users and Roles – roles, permissions, groups, statuses, and target improvements.
4. Main Modules – table with module names, status, backend/frontend files, API prefixes, and SaaS notes.
5. Core Business Workflows – step-by-step for critical flows (e.g., login, scheduling, task assignment, execution, abnormal handling, reporting).
6. Architecture – current and target diagrams (use Mermaid or text), technology stack comparison.
7. Roadmap / Upgrade Plan – phases for improvements (multi-tenant, security, frontend, scaling, enterprise features).
8. Current System Health – test metrics, build status, migration status, known blockers (P1/P2), resolved issues, technical debt.
9. Development Rules – must-read guidelines before changing code.
10. Design Constraints (for future development) – multi-tenant, storage, security, reliability, observability, frontend requirements.

Output the final document in Markdown `system-prompt-qa-suite\wiki\00-source-of-truth.md`, ready to be used as the team’s central source of truth. 
    