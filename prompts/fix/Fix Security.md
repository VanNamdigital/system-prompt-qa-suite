 I need you to thoroughly review and fix all real security vulnerabilities in the project.

  Scope:


  * Review the report at `system-prompt-qa-suite\result\result-Security.md

    * Verify each vulnerability is real, reproducible, and exploitable
    * Remove any false positives
    * For confirmed issues, implement proper production-grade fixes (no temporary patches)

  After applying fixes:

  * Run `docker compose dev` according to the README instructions
  * Start both backend (BE) and frontend (FE)
  * Check for:

    * No build errors
    * No runtime errors
  * Fix all issues until everything runs cleanly and consistently

  Standards:

  * Follow the Source of Truth: `system-prompt-qa-suite\wiki\00-source-of-truth.md`
  * If code deviates from the Wiki → refactor code to match the Wiki
  * If code is better than the Wiki → update the Wiki first, then align the code accordingly

  Execution rules:

  * Do not skip any vulnerabilities
  * Do not assume the report is correct — verify everything
  * Do not ignore inconsistencies between code and documentation
  * Ensure the system is fully functional, secure, and aligned with the defined standards