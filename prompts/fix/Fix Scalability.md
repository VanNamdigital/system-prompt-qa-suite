I need you to thoroughly review and fix all real scalability bottlenecks in the project.

Scope:

- Review the report at `system-prompt-qa-suite\prompts\result\result-Scalability.md`
- Write the fix report to `system-prompt-qa-suite\prompts\report\report-Scalability.md` documenting all applied fixes
  - Verify each scalability defect (statelessness violation, global lock, missing sharding key, O(n²) algorithm, connection limit) is reproducible under increased load or data volume
  - Remove any false positives (e.g., acceptable single master for expected size)
  - For confirmed issues, implement proper production-grade fixes (make stateless, add pagination, implement caching, sharding preparation, index optimization) — no temporary patches
- Ensure that adding more instances/resources proportionally increases throughput

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No session affinity requirements (if horizontal scaling intended)
  - No dramatic latency increase when data volume doubles
  - No hard-coded connection limits that block scaling
- Fix all issues until the system can scale to at least 2x current load with <20% latency increase

Standards:

- Follow the Source of Truth: `system-prompt-qa-suite\wiki\00-source-of-truth.md`
- If code deviates from the Wiki → refactor to match the Wiki
- If code is better → update the Wiki first

Execution rules:

- Do not skip any scalability defect
- Do not assume the report is correct — verify with load tests and instance count variations
- Do not ignore inconsistencies between code and documentation
- Ensure the system is horizontally scalable and aligned with defined standards
