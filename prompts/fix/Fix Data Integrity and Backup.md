I need you to thoroughly review and fix all real data integrity and backup issues in the project.

Scope:

- Review the report at `result\result-Reliability and Stability .md`
  - Verify each integrity defect (missing constraints, inconsistent writes, untested backup, missing checksums) is reproducible and leads to data corruption/loss
  - Remove any false positives (e.g., intentional soft delete with retention)
  - For confirmed issues, implement proper production-grade fixes (add foreign keys, enable WAL archiving, test restore, add checksums) — no temporary patches
- Ensure that all critical data can be recovered from backups and that writes are atomic/consistent

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No data corruption after simulated crashes or concurrent writes
  - Backup and restore procedure works (test with a dry run)
  - No silent data loss
- Fix all issues until data integrity and recovery are verified

Standards:

- Follow the Source of Truth: `wiki\00-source-of-truth.md`
- If code/schemas deviate from the Wiki → refactor to match the Wiki
- If current implementation is better → update the Wiki first

Execution rules:

- Do not skip any integrity or backup defect
- Do not assume the report is correct — verify with actual DB commands, backup listings, and restore tests
- Do not ignore inconsistencies between code and documentation
- Ensure the system maintains data integrity, has verifiable backups, and aligns with defined standards