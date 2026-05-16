I need you to thoroughly review and fix all real cost and resource optimization issues in the project.

Scope:

- Review the report at `system-prompt-qa-suite\prompts\result\result-Cost and Resource Optimization.md`
- Write the fix report to `system-prompt-qa-suite\prompts\report\report-Cost and Resource Optimization.md` documenting all applied fixes
  - Verify each cost defect (over-provisioned resources, idle instances, unused storage, inefficient queries, unnecessary data transfer) is measurable in actual usage metrics
  - Remove any false positives (e.g., intentional over-provisioning for planned traffic)
  - For confirmed issues, implement proper production-grade fixes (right-size instances, set auto-scaling bounds, implement storage lifecycle policies, optimize expensive queries, add budget alerts) — no temporary patches
- Ensure cloud resources are sized appropriately and infrastructure spend is aligned with actual demand

After applying fixes:

- Run `docker compose dev` according to the README instructions
- Start both backend (BE) and frontend (FE)
- Check for:
  - No service disruption from resource right-sizing
  - No performance degradation from downsizing
  - Auto-scaling policies have reasonable min/max limits
  - Storage lifecycle policies applied (transition/expiration rules)
  - Cost budgets and alerts configured
- Fix all issues until resource utilization and costs are optimized without sacrificing performance or reliability

Standards:

- Follow the Source of Truth: `wiki\00-source-of-truth.md`
- If infrastructure code/deviation deviates from the Wiki → refactor to match the Wiki
- If current configuration is more cost-effective → update the Wiki first

Execution rules:

- Do not skip any cost optimization opportunity
- Do not assume the report is correct — verify actual resource utilization metrics via cloud provider console or monitoring tools
- Do not ignore inconsistencies between code and documentation
- Ensure the system is cost-efficient, right-sized, and aligned with defined standards
