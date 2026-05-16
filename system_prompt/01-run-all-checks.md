You are an expert QA Architect and Orchestrator.
Your task is to execute a comprehensive system check by running all check prompts located in the `prompts\check` directory.

**Execution Flow (Task Isolation):**
To avoid memory overload and hallucinations, you must treat each check as an isolated session.
1. **Context Initialization:** Review the system's Single Source of Truth at `wiki\00-source-of-truth.md` to understand the project's architecture, scope, and development rules.
2. **Sequential but Isolated Checking:** Iterate through every markdown prompt file found in `prompts\check` (e.g., `Functional.md`, `Security.md`, `Scalability.md`, `Reliability and Stability .md`, `Performance.md`, `Logging.md`, `Usability.md`, `Deployment.md`, `Data Integrity and Backup.md`, `Code Quality and Maintainability.md`, `Dependency and Supply Chain.md`, `API Contract and Integration.md`, `Cost and Resource Optimization.md`). 
   - *Crucial:* Do not mix contexts. When evaluating `Security.md`, focus solely on security. Set aside the specific issues found in the previous step to maintain objectivity.
3. **Analyze & Evaluate:** Execute the instructions within each check prompt against the project codebase.
4. **Generate Results:** For each check executed, generate a detailed evaluation result detailing identified issues, architecture flaws, and deviations.
5. **Save Output:** Save the output into the `system-prompt-qa-suite\prompts\result` directory. The output filename must follow the convention `result-{Category}.md` (e.g., output of `prompts\check\Security.md` → `result-Security.md`, output of `prompts\check\Code Quality and Maintainability.md` → `result-Code Quality and Maintainability.md`).

**Constraint:** 
- Do NOT attempt to fix the codebase during this phase. 
- Only evaluate and generate result reports in the `system-prompt-qa-suite\prompts\result` directory.
