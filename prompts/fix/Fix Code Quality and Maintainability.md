I need you to thoroughly review and fix all real code quality and maintainability issues in the project.

Scope:

- Review the report at `system-prompt-qa-suite\prompts\result\result-Code Quality and Maintainability.md`
- Write the fix report to `system-prompt-qa-suite\prompts\report\report-Code Quality and Maintainability.md` documenting all applied fixes
  - Verify each code quality defect (high cyclomatic complexity, god classes, dead code, duplication, low test coverage) is reproducible via static analysis or code review
  - Remove any false positives (e.g., intentionally complex algorithm with no simpler alternative)
  - For confirmed issues, implement proper production-grade fixes (extract functions, refactor god classes, remove dead code, deduplicate logic, add tests for uncovered paths, enforce linting) — no temporary patches
- Ensure the codebase follows consistent naming, formatting, and architectural conventions

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No build or runtime errors introduced by refactoring
  - Cyclomatic complexity reduced below thresholds (avg <10, none >20 for critical paths)
  - No test regressions (existing tests still pass)
  - Code duplication below 5% (or acceptable project threshold)
- Fix all issues until the codebase is maintainable, consistent, and well-structured

Standards:

- Follow the Source of Truth: `wiki\00-source-of-truth.md`
- If code deviates from the Wiki → refactor to match the Wiki
- If code is better → update the Wiki first

Execution rules:

- Do not skip any code quality defect
- Do not assume the report is correct — verify with linters, complexity analyzers, and manual review
- Do not ignore inconsistencies between code and documentation
- Ensure the system is clean, maintainable, and aligned with defined standards
