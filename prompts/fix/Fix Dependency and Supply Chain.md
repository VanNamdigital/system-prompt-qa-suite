I need you to thoroughly review and fix all real dependency and supply chain risks in the project.

Scope:

- Review the report at `system-prompt-qa-suite\prompts\result\result-Dependency and Supply Chain.md`
- Write the fix report to `system-prompt-qa-suite\prompts\report\report-Dependency and Supply Chain.md` documenting all applied fixes
  - Verify each dependency defect (known CVE, unmaintained package, unpinned version, missing lock file, license conflict) is real and exploitable
  - Remove any false positives (e.g., dev-only dep with no runtime impact)
  - For confirmed issues, implement proper production-grade fixes (upgrade to patched version, replace abandoned library, pin all versions with lock files, add SBOM generation, configure automated scanning) — no temporary patches
- Ensure all dependencies are tracked, scanned, and compliant

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No build failures from dependency changes
  - No runtime errors from version upgrades
  - Lock files present and committed for all package managers
  - Vulnerability scanner (e.g., `npm audit`, `safety check`, `trivy`) reports zero critical/high CVEs
  - SBOM can be generated (e.g., `cyclonedx-bom`)
- Fix all issues until the supply chain is secure, compliant, and well-managed

Standards:

- Follow the Source of Truth: `wiki\00-source-of-truth.md`
- If dependency configuration deviates from the Wiki → refactor to match the Wiki
- If current state is more secure → update the Wiki first

Execution rules:

- Do not skip any dependency risk
- Do not assume the report is correct — verify with actual `npm audit`, `pip audit`, `trivy`, or equivalent tools
- Do not ignore inconsistencies between code and documentation
- Ensure the system is secure from supply chain attacks and aligned with defined standards
