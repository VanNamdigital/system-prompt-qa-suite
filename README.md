# System Prompt QA Suite

A comprehensive, structured prompt system for automated security scanning, quality checking, and production-readiness review on any software project.

## Features

- **Full Audit Workflow**: Run all scans and checks in sequence
- **Incremental Updates**: Smart diff-based reporting that tracks changes over time
- **Multi-Lane Coverage**: Security, quality, compliance, frontend, language-specific, project-type-specific
- **Severity Tracking**: Unified CRITICAL/HIGH/MEDIUM/LOW/INFO system
- **History Preservation**: Never lose previous scan results

## Directory Structure

```
system-prompt-qa-suite/
├── system_prompt/                          # Orchestrator prompts (run in order)
│   ├── 00-create-single-source-of-truth.md # Generate project context
│   ├── 001-small-scan.md                   # Quick security scan orchestrator
│   ├── 002-small-check.md                  # Quick system check orchestrator
│   ├── 003-full-scan.md                    # Full security scan orchestrator
│   ├── 004-full-check.md                   # Full system check orchestrator
│   ├── 005-project-type-scan.md            # Project-type scan orchestrator
│   ├── 006-language-specific-scan.md       # Language-specific scan orchestrator
│   ├── 007-compliance-scan.md              # Compliance scan orchestrator
│   ├── 008-frontend-check.md               # Frontend check orchestrator
│   └── 009-master-orchestrator.md          # Master workflow orchestrator
├── prompts/
│   ├── scan/
│   │   ├── full/00-full-scan.md            # 21 security areas
│   │   ├── small/00-small-scan.md          # 12 quick security checks
│   │   ├── project-type/                   # 13 project types
│   │   ├── language/                       # 28 language/framework areas
│   │   └── compliance/                     # OWASP/PCI/HIPAA/GDPR/SOC2
│   ├── check/
│   │   ├── full/00-full-check.md           # 20 system check categories
│   │   ├── small/00-small-check.md         # 12 quick check categories
│   │   └── frontend/00-frontend-check.md   # 25 frontend checks
│   ├── result/                             # Scan/check outputs
│   ├── report/                             # Reports
│   ├── fix/                                # Fix prompts
│   └── final/00-final-check.md             # Final unified report
└── wiki/
    └── 00-single-source-of-truth.md        # Generated project context
```

---

## Prompt Roles

### System Prompts (Orchestrators)

| File | Role | Input | Output |
|------|------|-------|--------|
| `00-create-single-source-of-truth.md` | Generate project context (structure, modules, users, architecture) | Target project | `wiki/00-single-source-of-truth.md` |
| `001-small-scan.md` | Orchestrate quick security scan (12 categories) | Source of truth | `prompts/result/scan/small/` |
| `002-small-check.md` | Orchestrate quick system check (12 categories) | Source of truth | `prompts/result/check/small/` |
| `003-full-scan.md` | Orchestrate full security scan (21 areas) | Source of truth | `prompts/result/scan/full/` |
| `004-full-check.md` | Orchestrate full system check (20 categories) | Source of truth | `prompts/result/check/full/` |
| `005-project-type-scan.md` | Orchestrate project-type scan (SPA, API, Mobile, E-commerce...) | Source of truth | `prompts/result/scan/project-type/` |
| `006-language-specific-scan.md` | Orchestrate language-specific scan (JS/TS, Python, Java, C#, Go, PHP, Ruby, Rust) | Source of truth | `prompts/result/scan/language/` |
| `007-compliance-scan.md` | Orchestrate compliance scan (OWASP, PCI DSS, HIPAA, GDPR, SOC 2) | Source of truth | `prompts/result/scan/compliance/` |
| `008-frontend-check.md` | Orchestrate frontend check (Performance, Accessibility, SEO, UX) | Source of truth | `prompts/result/check/frontend/` |
| `009-master-orchestrator.md` | Orchestrate full workflow + incremental update | All results | `prompts/final/00-final-check.md` |

### Inner Prompts (Detailed)

| File | Role | Check Count |
|------|------|-------------|
| `prompts/scan/full/00-full-scan.md` | 21 detailed security areas | 21 areas × N checks |
| `prompts/scan/small/00-small-scan.md` | 12 quick security checks | 12 categories |
| `prompts/scan/project-type/00-project-type-scan.md` | 13 project types | 13 project types |
| `prompts/scan/language/00-language-specific-scan.md` | 28 language/framework areas | 28 language areas |
| `prompts/scan/compliance/00-compliance-scan.md` | OWASP + PCI + HIPAA + GDPR + SOC2 | 5 standards × N controls |
| `prompts/check/full/00-full-check.md` | 20 system check categories | 20 categories × N checks |
| `prompts/check/small/00-small-check.md` | 12 quick check categories | 12 categories |
| `prompts/check/frontend/00-frontend-check.md` | 25 frontend checks | 25 categories |
| `prompts/final/00-final-check.md` | Final report + Production review | Merge all + Classify |

---

## Execution Order

### QUICK AUDIT MODE (Fast)
```
Step 0: 00-create-single-source-of-truth.md  → Generate context
Step 1: 001-small-scan.md                     → Quick security scan
Step 2: 002-small-check.md                    → Quick system check
Step 3: Merge → 00-final-check.md             → Final report
```

### FULL AUDIT MODE (Comprehensive)
```
Step 0: 00-create-single-source-of-truth.md  → Generate context
Step 1: 003-full-scan.md                      → Full security scan (21 areas)
Step 2: 004-full-check.md                     → Full system check (20 categories)
Step 3: 005-project-type-scan.md              → Project-type scan
Step 4: 006-language-specific-scan.md         → Language-specific scan
Step 5: 007-compliance-scan.md                → Compliance scan
Step 6: 008-frontend-check.md                 → Frontend check
Step 7: Merge → 00-final-check.md             → Final report + Production review
```

---

## Incremental Update Logic

All prompts support incremental updates:

```
1. Check if previous report exists?
   ├── YES → Read old report → Store PREVIOUS_CONTEXT → Mode: INCREMENTAL
   └── NO  → Mode: FULL SCAN

2. Run current scan/check → CURRENT_RESULTS

3. Compare findings:
   ├── New finding → Mark [+]
   ├── Missing finding → Mark [-] (Resolved)
   ├── Changed severity → Mark [~]
   └── Unchanged → Keep as-is

4. Generate Change Summary + Scan History

5. Save updated report
```

---

## Severity System (Unified)

| Level | Scan | Check | Final (Production) |
|-------|------|-------|-------------------|
| CRITICAL | RCE, credential exposure, SQL injection | Data loss, broken SLA | CLASS A - BLOCKER |
| HIGH | XSS, IDOR, SSRF, hardcoded secrets | Breaks core workflow | CLASS A - BLOCKER |
| MEDIUM | Missing rate limit, weak crypto | Degrades experience | CLASS B - IMPORTANT |
| LOW | Missing cookie flags, debug endpoint | Cosmetic, docs-only | CLASS C - NICE TO HAVE |
| INFO | Observation, hardening suggestion | Non-blocking improvement | CLASS C - NICE TO HAVE |

---

## Scan vs Check

| | SCAN | CHECK |
|---|------|-------|
| **Goal** | Security | Quality |
| **Focus** | Vulnerabilities, exploits, attacks | Logic, performance, UX, maintainability |
| **Output** | Attack scenarios, exploit paths | Impact analysis, remediation |
| **Examples** | SQL injection, XSS, hardcoded secrets | N+1 query, missing pagination, poor UX |

---

## How to Use

### Quick Start
1. Copy this suite into your target project
2. Run `system_prompt/009-master-orchestrator.md` with your AI assistant
3. Choose Quick Audit or Full Audit mode
4. Review the generated report in `prompts/final/00-final-check.md`

### Incremental Scans
- Run the same orchestrator again after making changes
- The system will automatically detect and report:
  - New issues found
  - Issues resolved
  - Severity changes
  - Trend analysis

### Individual Lanes
You can also run individual orchestrators for targeted scans:
- `003-full-scan.md` — Security only
- `004-full-check.md` — Quality only
- `007-compliance-scan.md` — Compliance only
- `008-frontend-check.md` — Frontend only

---

## License

See [LICENSE](LICENSE) for details.
