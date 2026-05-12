I need you to thoroughly review and fix all real usability issues in the project.

Scope:

- Review the report at `system-prompt-qa-suite\prompts\result\result-Usability.md`
- Write the fix report to `system-prompt-qa-suite\prompts\report\report-Usability.md` documenting all applied fixes
  - Verify each usability defect (confusing workflow, missing validation feedback, poor error recovery, accessibility violation) is reproducible by a human user
  - Remove any false positives (e.g., intentional design choice that does not hinder most users)
  - For confirmed issues, implement proper production-grade fixes (add inline validation, preserve form state, provide undo, improve keyboard navigation, WCAG compliance) — no temporary patches
- Ensure that primary user tasks can be completed with minimal clicks, clear feedback, and recoverable mistakes

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No loss of user input after validation errors
  - All destructive actions have confirmation or undo
  - Screen reader and keyboard navigation works for critical paths
- Fix all issues until the application meets usability and accessibility standards

Standards:

- Follow the Source of Truth: `system-prompt-qa-suite\wiki\00-source-of-truth.md`
- If UI code deviates from the Wiki → refactor to match the Wiki
- If current UX is better → update the Wiki first

Execution rules:

- Do not skip any usability defect
- Do not assume the report is correct — manually test with realistic user scenarios and assistive tools
- Do not ignore inconsistencies between code and documentation
- Ensure the system is user-friendly, accessible, and aligned with defined standards
