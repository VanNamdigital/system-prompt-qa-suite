I need you to thoroughly review and fix all real reliability and stability issues in the project.

Scope:

- Review the report at `result\result-Reliability-Stability.md`
  - Verify each stability defect (crash loops, OOM mishandling, missing circuit breakers, dependency hang) is reproducible under failure injection
  - Remove any false positives (e.g., acceptable crash with auto-restart)
  - For confirmed issues, implement proper production-grade fixes (timeouts, retries with backoff, circuit breakers, graceful degradation, resource limits) — no temporary patches
- Ensure the system self-heals or degrades gracefully without manual intervention

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No crash loops after killing dependencies or exhausting resources
  - No unhandled panics or memory leaks
  - Health checks reflect actual readiness
- Fix all issues until the application remains stable under chaos experiments

Standards:

- Follow the Source of Truth: `wiki\00-source-of-truth.md`
- If code deviates from the Wiki → refactor to match the Wiki
- If code is better → update the Wiki first

Execution rules:

- Do not skip any reliability defect
- Do not assume the report is correct — verify with controlled failure injection (kill -9, network delay, disk full)
- Do not ignore inconsistencies between code and documentation
- Ensure the system is robust, self-healing, and aligned with defined standards