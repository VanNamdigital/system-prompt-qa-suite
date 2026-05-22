# Full Scan Prompt Group

Role: You are a full-scope application security auditor. Read `00-single-source-of-truth.md` before starting. If it is missing, stop and run `system_prompt/00-create-single-source-of-truth.md`.

This prompt group is original to this suite. Produce reproducible findings based only on the target repository, the Single Source of Truth, and the security coverage areas defined below.

Save the merged output to `prompts/result/scan/full/result-full-scan.md`.

## Output Rules

- Do not modify code.
- Save only to `prompts/result/scan/full/`.
- Every finding must map to one primary security area.
- Every claim must include technical proof: file path, line, command, snippet, or deterministic reproduction.
- Include a PASS section for applicable rules that were checked and had no findings.
- Include a JSON summary at the end with `verdict`, severity counts, files reviewed, and findings.

### Severity Levels
| Level | Criteria |
|-------|----------|
| CRITICAL | Remote code execution, credential exposure (live/prod), SQL injection, auth bypass |
| HIGH | XSS, IDOR, SSRF, insecure deserialization, hardcoded secrets (configs), path traversal |
| MEDIUM | Missing rate limit, permissive CORS, verbose error, weak crypto, missing security header |
| LOW | Missing cookie flags, debug endpoint exposed, info leakage in stack trace |
| INFO | Observation, hardening suggestion, future risk, defense-in-depth |

---

## Security Coverage Areas

### 1. Secret Exposure

**Goal:** Find committed credentials, tokens, private keys, passwords, signing keys, unsafe secret handling.

- [ ] API keys / tokens: regex patterns for AWS (`AKIA[0-9A-Z]{16}`), GitHub (`ghp_*`, `ghs_*`, `gho_*`), Slack (`xox[baprs]-`), Google (`AIza[0-9A-Za-z_-]{35}`), SendGrid (`SG\.[a-zA-Z0-9]`), Stripe (`sk_live_`, `pk_live_`)
- [ ] Private keys: `-----BEGIN (RSA|EC|DSA|OPENSSH|PGP) PRIVATE KEY-----`
- [ ] Connection strings: `grep -rn "Server=.*;Database=.*;User.*=.*;Password=.*"` — hardcoded DB creds
- [ ] `.env*` files committed to version control
- [ ] Docker/Compose/K8s manifests with plaintext secrets
- [ ] CI/CD secrets in plaintext: `grep -rn "password\|token\|apikey" --include="*.{yml,yaml}"` in `.github/`, `.gitlab-ci.yml` etc.
- [ ] Example configs with real credentials: `**/*.example*`, `**/*.sample*`, `**/*.template*`

**Search:** `rg -n "-----BEGIN.*PRIVATE KEY-----|sk_live_|pk_live_|AKIA[0-9A-Z]{16}|ghp_[a-zA-Z0-9]{36}|xox[baprs]-[0-9]|SG\.[a-zA-Z0-9]"`

---

### 2. Data Injection

**Goal:** Find unsafe database, search, template, expression, LDAP, XML, and shell-adjacent input handling.

- [ ] SQL injection: raw string concatenation in queries (`SELECT * FROM " + userInput`), raw ORM calls (`.raw()`, `RawSql`, `ExecuteSqlRaw`)
- [ ] NoSQL injection: `$where`, `$ne`, `$gt`, `$regex` operators from user input in MongoDB/NoSQL
- [ ] Template injection: user input in `render_template_string`, `Template()`, `.parse()` (SSTI)
- [ ] LDAP injection: user input in LDAP filter strings without sanitization
- [ ] Shell injection: `os.system()`, `subprocess.Popen()` with user-controlled strings
- [ ] Expression injection: `eval()`, `new Function()`, `Expression` (SpEL, JEXL, MVEL)
- [ ] XPath injection: user input in XPath query strings

**Search:**
- `rg -n "SELECT.*+.*WHERE\|DELETE.*+\|INSERT.*+.*VALUES\|UPDATE.*+" --include="*.{py,ts,go,java,cs}"` (raw SQL concat)
- `rg -n "render_template_string\|Template(\|\.parse(" --include="*.{py,ts,java}"` (SSTI)
- `rg -n "os\.system\|subprocess\.\|exec(\|execSync" --include="*.{py,ts,go,java}"` (shell)
- `rg -n "find(.*\$where\|find(.*\$regex\|find(.*\$ne" --include="*.{js,ts}"` (NoSQL)

---

### 3. Browser-Side Injection (XSS)

**Goal:** Find stored/reflected DOM issues, unsafe HTML rendering, rich text, markdown, scriptable attributes.

- [ ] `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write()` — CRITICAL with user data
- [ ] `v-html` (Vue), `dangerouslySetInnerHTML` (React), `{@html}` (Svelte), `{{{ }}}` (Handlebars)
- [ ] Markdown renderers without XSS sanitization (marked, markdown-it, remark without rehype-sanitize)
- [ ] `javascript:` URLs in `href`/`src`/`action` from user data
- [ ] `eval()`, `setTimeout(string)`, `setInterval(string)`, `new Function(string)`
- [ ] DOM clobbering: `id`/`name` attributes that shadow global properties
- [ ] Prototype pollution: unsafe merge/assign from user input

**Search:**
- `rg -n "\.innerHTML\|\.outerHTML\|insertAdjacentHTML\|document\.write\|v-html\|dangerouslySetInnerHTML\|{@html" --include="*.{html,tsx,jsx,vue,svelte,ts,js}"`
- `rg -n "marked\|markdown-it\|remark\|showdown\|marked" --include="*.{ts,js,html}"`
- `rg -n "eval(\|new Function(\|setTimeout.*['\"]\|setInterval.*['\"]" --include="*.{ts,js}"`
- `rg -n "__proto__\|constructor\.prototype" --include="*.{ts,js}"`

---

### 4. Object Access Control (IDOR/BAC)

**Goal:** Find missing ownership, tenant, role, permission, and resource-level authorization.

- [ ] Controllers/handlers with `:id` params — check for owner/tenant check
- [ ] Bulk operations where user can specify multiple IDs
- [ ] GraphQL resolvers without auth directives on each resolver
- [ ] File download endpoints that accept filename/ID from user
- [ ] WebSocket events without per-message authorization
- [ ] Admin functionality without server-side role check
- [ ] Tenant isolation: cross-tenant data access possible?

**Search:**
- `rg -n "req\.params\.\|:id\|{id}\|/:id\|\.params\|\.getById\|\.FirstOrDefault\|\.SingleOrDefault\|\.FindAsync" --include="*.{py,ts,go,java,cs}"`
- `rg -n "authorize\|Authorize\|@Secured\|permission\|Policy\|role\|owner" --include="*.{py,ts,cs,java}"`
- `rg -n "res\.sendFile\|FileStreamResult\|PhysicalFile\|\.Download\|return.*File(" --include="*.{cs,py,ts}"`

---

### 5. Authentication and Session Design

**Goal:** Review login, reset, MFA, remember-me, cookies, refresh tokens, expiry, logout.

- [ ] Password hashing: bcrypt/argon2/scrypt/PBKDF2? reject MD5/SHA1/SHA256 for passwords
- [ ] JWT: verify algorithm not `none`, `HS256` with strong secret, `RS256` with trusted public key, `aud`/`iss`/`exp` validated
- [ ] Session: httpOnly + secure + SameSite cookies; no predictable session IDs
- [ ] Login: rate limit per-IP + per-account; no verbose error (user exists vs password wrong)
- [ ] Password reset: token expiry 15-60 min, single-use, tied to account, sent to verified email only
- [ ] MFA: if documented but not implemented → HIGH finding
- [ ] Remember-me: token rotation on use; not stored in plaintext
- [ ] Logout: session destroyed server-side, token blacklisted/revoked
- [ ] Refresh tokens: stored securely, rotated on use, short-lived access tokens
- [ ] Account enumeration: login/password reset/resend should have identical messages for valid/invalid users

**Search:**
- `rg -n "bcrypt\|argon2\|scrypt\|PBKDF2\|password_hash" --include="*.{py,ts,go,java,cs}"`
- `rg -n "jwt\|JWT\|jsonwebtoken" --include="*.{py,ts,go,java,cs}"`
- `rg -n "session\|cookie\|httpOnly\|secure\|SameSite" --include="*.{py,ts,go,java,cs}"`

---

### 6. Abuse Resistance

**Goal:** Find brute force, missing throttles, costly endpoints, account enumeration, automation defenses.

- [ ] Rate limiting on auth, registration, password reset, OTP, invite endpoints
- [ ] Expensive endpoints (export, report, batch, render) without rate limiting
- [ ] Account enumeration via timing attacks or distinct error messages
- [ ] CAPTCHA/challenge on critical forms (login, registration, contact)
- [ ] Webhook endpoints without origin validation or rate limit
- [ ] API key usage without usage quota or rate limiting

**Search:**
- `rg -n "rate.*limit\|RateLimit\|throttl\|429\|TooManyRequests\|TooManyRequest" --include="*.{py,ts,cs,java}"`
- `rg -n "recaptcha\|captcha\|hcaptcha\|turnstile\|challenge" --include="*.{html,tsx,jsx,py,ts}"`
- `rg -n "export\|report\|render\|batch\|bulk" --include="*.{py,ts,cs}"` (costly endpoints)

---

### 7. Request Binding Risks

**Goal:** Find over-posting, privileged fields, unsafe model binding, schema gaps, trust boundary.

- [ ] Mass assignment: auto-binding request body to entity models (e.g., `@RequestBody Entity`, `FromBody`, `request.POST`)
- [ ] Privileged fields: `role`, `isAdmin`, `isActive`, `balance`, `price` in request body
- [ ] DTO/ViewModels not used (entity exposed directly in API)
- [ ] GraphQL: query depth/complexity limits missing, allowing expensive nested queries
- [ ] Over-fetching: API returns more fields than UI needs (exposes internal data)
- [ ] Trust boundary: server trusting `X-Forwarded-For`, `X-Real-IP`, `X-User` from client

**Search:**
- `rg -n "@RequestBody\|FromBody\|\[FromBody\]\|request\.body\|req\.body" --include="*.{py,ts,cs,java}"`
- `rg -n "isAdmin\|isadmin\|role\|isActive\|is_active\|user\.role" --include="*.{py,ts,cs,java}"`
- `rg -n "X-Forwarded-For\|X-Real-IP\|X-User\|X-Auth-Token" --include="*.{py,ts,cs}"`

---

### 8. Unsafe Parsing and Dynamic Execution

**Goal:** Find object loading, dynamic imports, eval-like behavior, XML entity risks, plugin loading.

- [ ] Unsafe deserialization: `pickle.loads()`, `BinaryFormatter`, `SoapFormatter`, `Marshal.load()`, `YAML.load()` (not safe)
- [ ] Dynamic import: `importlib.import_module()`, `__import__()`, `require()` with variable, `import()` with user input
- [ ] XXE: XML parsers without `resolveExternalEntities=false` / `DTD=false` / `expandEntityReferences=false`
- [ ] Expression language injection: SpEL, JEXL, MVEL, OGNL with user input
- [ ] Plugin/module loading from user-writable paths
- [ ] Regex injection: user-provided regex patterns (ReDoS risk)

**Search:**
- `rg -n "pickle\.\|BinaryFormatter\|SoapFormatter\|LosFormatter\|Marshal\.load\|YAML\.load\|yaml_load" --include="*.{py,cs,rb}"`
- `rg -n "importlib\|__import__\|exec(\|eval(\|new Function(" --include="*.{py,ts}"`
- `rg -n "XmlDocument\|XmlReader\|SAXParser\|DocumentBuilder" --include="*.{py,cs,java}"`

---

### 9. Server-Side Request Risks (SSRF)

**Goal:** Find URL fetches, webhook validation, metadata access, redirect handling, internal network reachability.

- [ ] User-controlled URL via `fetch()`, `axios()`, `requests.*`, `HttpClient`, `WebClient`, `RestTemplate`, `curl`
- [ ] Webhook/callback with user-supplied URL → require allowlist validation
- [ ] Image/URL importers that fetch from user-provided URLs
- [ ] Cloud metadata access: `169.254.169.254` (AWS/GCP/Azure) — verify blocked or restricted
- [ ] Internal network scan risk: `localhost`, `127.0.0.1`, `10.x.x.x`, `172.16-31.x.x`, `192.168.x.x`
- [ ] Redirect following: client follows redirects automatically to internal/SSRF targets
- [ ] Protocol smuggling: user can specify `file://`, `gopher://`, `dict://` protocols

**Search:**
- `rg -n "fetch(\|axios\..*get\|requests\.\|HttpClient\|WebClient\|RestTemplate\|curl" --include="*.{py,ts,cs,java,go}"`
- `rg -n "localhost\|127\.0\.0\.1\|169\.254\|metadata" --include="*.{py,ts,cs,java}"`
- `rg -n "follow.*redirect\|allow.*redirect\|max.*redirect" --include="*.{py,ts,cs}"`

---

### 10. Filesystem and Upload Safety

**Goal:** Check file reads/writes, path joins, archive extraction, content validation, public storage.

- [ ] Path traversal: `..` / `../` in user-supplied filenames not sanitized
- [ ] File upload: extension allowlist (not blocklist), MIME/content validation
- [ ] Archive extraction (zip/tar): zip-slip vulnerability (`../../../etc/passwd` in entry name)
- [ ] Decompression bomb: no size/extraction limit on archives
- [ ] File storage in webroot: uploaded files accessible directly via URL
- [ ] Symlink following: unsafe symlink resolution in archives
- [ ] Temporary files: created in predictable paths, not cleaned up
- [ ] Server-Side Include / SSI in uploaded files

**Search:**
- `rg -n "\.\./\\\\|\.\.\/\|Path\.Combine\|path\.join\|Path\.GetFullPath\|path\.resolve\|path\.normalize" --include="*.{py,ts,cs,java}"`
- `rg -n "unzip\|unzip\|tar\|extract\|decompress\|zipfile\|ZipFile\|SharpZipLib" --include="*.{py,ts,cs}"`
- `rg -n "upload\|multipart\|form-data\|IFormFile\|UploadedFile\|FilePart" --include="*.{py,ts,cs,java}"`

---

### 11. Browser Trust Boundaries (CSRF/CORS)

**Goal:** Check CSRF, CORS, cookie policy, origin validation, cross-site credential exposure.

- [ ] CSRF: state-changing endpoints (POST/PUT/DELETE) protected by CSRF token or SameSite=Strict cookie
- [ ] CORS: `Access-Control-Allow-Origin: *` with credentials? → HIGH (any site can read response)
- [ ] CORS: `Access-Control-Allow-Origin: null` → bypassable via `null` origin in sandboxed iframe
- [ ] Cookie attributes: `Secure`, `HttpOnly`, `SameSite=Lax|Strict` missing
- [ ] `Content-Type` sniffing: missing `X-Content-Type-Options: nosniff`
- [ ] Clickjacking: missing `X-Frame-Options` or `Content-Security-Policy: frame-ancestors`
- [ ] PostMessage: unsafe origin validation in `message` event handlers

**Search:**
- `rg -n "cors\|CorsPolicy\|enableCors\|@CrossOrigin\|CrossOrigin" --include="*.{py,ts,cs,java,yaml,yml}"`
- `rg -n "csrf\|X-CSRF\|CSRF\|_csrf\|csrfProtect\|Csrf\|Antiforgery" --include="*.{py,ts,cs,java}"`
- `rg -n "SameSite\|same_site\|HttpOnly\|httpOnly\|Secure\|secure\|X-Frame-Options\|X-Content-Type-Options" --include="*.{py,ts,cs,java,conf,config}"`

---

### 12. Cryptography and Token Validation

**Goal:** Check password hashing, token signing, algorithm choices, key rotation, issuer/audience checks.

- [ ] Weak hash algorithms: MD5, SHA1 used for passwords/signatures → HIGH
- [ ] JWT: algorithm confusion (uses `RS256` public key to verify `HS256` token)
- [ ] JWT: `kid` header injection (unsanitized key ID leading to path traversal)
- [ ] ECB mode encryption (deterministic, patterns visible)
- [ ] Fixed IV/nonce in encryption (reuse of nonce with same key)
- [ ] Weak RSA key size (< 2048 bits)
- [ ] Hardcoded encryption keys in source code
- [ ] Tokens: missing expiry, missing signature validation

**Search:**
- `rg -n "MD5\|SHA1\|md5\|sha1\|SHA-1" --include="*.{py,ts,cs,java,go}"` (for security, not just hashing)
- `rg -n "AES.*ECB\|DES\|3DES\|RC4\|ARC4" --include="*.{py,ts,cs,java,go}"`
- `rg -n "jwt\|JWT\|jsonwebtoken" --include="*.{py,ts,cs,java,go}"` — then check algorithm handling

---

### 13. Error and Debug Exposure

**Goal:** Find stack traces, debug modes, sensitive logs, verbose responses, dev-only controls exposed in production.

- [ ] Debug mode enabled: `DEBUG=True`, `NODE_ENV=development`, `ASPNETCORE_ENVIRONMENT=Development` in prod configs
- [ ] Stack traces in API responses (ASP.NET detailed errors, Django DEBUG, Flask debugger)
- [ ] Verbose SQL error messages exposing schema
- [ ] Internal path disclosure via error messages: `C:\app\`, `/var/www/`, `/home/`
- [ ] Minified source maps deployed to production (exposes full source code)
- [ ] Swagger/OpenAPI UI enabled in production (exposes full API surface)
- [ ] Health endpoint leaking sensitive info (DB versions, internal IPs, dependency info)

**Search:**
- `rg -n "DEBUG\|debug\|DEVELOPMENT\|development\|STACK_TRACE\|detailedErrors" --include="*.{py,ts,cs,yaml,yml,env}"`
- `rg -n "app\.useDeveloperExceptionPage\|UseDeveloperExceptionPage\|ShowDetailedErrors" --include="*.{cs,py}"`
- `**/.map` files in production build output

---

### 14. Concurrency and Business Abuse

**Goal:** Find double spend, stale authorization, duplicate submissions, race-prone inventory/balance logic.

- [ ] Balance/budget operations without atomicity: `read → modify → write` without locks or transactions
- [ ] Inventory/decrement operations: check-and-set without optimistic/pessimistic locking
- [ ] Coupon/promo code: single-use not enforced with DB constraint
- [ ] Idempotency: critical mutations (payment, order) lack idempotency key
- [ ] Race window: time-of-check/time-of-use (TOCTOU) in auth or ownership validation
- [ ] Concurrent session handling: session fixation, concurrent login edge cases

**Search:**
- `rg -n "SELECT.*balance\|balance.*=\|credit.*=\|decrement\|increment\|count.*=\|\-\-.*balance" --include="*.{py,ts,cs,java,sql}"` — atomic or not?
- `rg -n "transaction\|lock\|pessimistic\|optimistic\|atomic\|synchronized\|mutex" --include="*.{py,ts,cs,java,go}"`
- `rg -n "idempot\|unique.*key\|dedup\|idempotency" --include="*.{py,ts,cs}"`

---

### 15. Dependency and Supply Chain

**Goal:** Find vulnerable versions, suspicious packages, install scripts, lockfile drift, vendored binaries.

- [ ] Known CVEs: run `npm audit`/`pip-audit`/`dotnet list package --vulnerable`/`go list -m all` + `govulncheck`
- [ ] Unpinned versions: `^`, `~`, `>=`, `*`, `x.x` in package manifests — review policy
- [ ] Typo-squatting: suspicious package names similar to popular ones
- [ ] Postinstall scripts: `rg -n "postinstall\|preinstall\|prebuild\|postbuild" --include="*.json"` — review each
- [ ] Vendored binaries (`*.exe`, `*.dll`, `*.so`, `*.dylib`, `*.jar`) — no source/verification
- [ ] Lockfile integrity: `package-lock.json` / `yarn.lock` hash mismatch with `package.json`
- [ ] Deprecated packages: not updated in > 2 years with known alternatives
- [ ] Bundle integrity: CDN-loaded libs without SRI (Subresource Integrity)

**Search:**
- `**/package.json`, `**/requirements.txt`, `**/Pipfile*`, `**/go.mod`, `**/*.csproj`, `**/*.fsproj`, `**/Cargo.toml`, `**/Gemfile`, `**/Gemfile.lock`, `**/build.gradle`, `**/pom.xml`
- `rg -n "https://\|http://" --include="*.{html,tsx,jsx}"` — external script sources with SRI?

---

### 16. Command and Process Execution

**Goal:** Find user-controlled shell/process calls, script runners, task schedulers, and argument escaping.

- [ ] Shell execution: `os.system()`, `subprocess.*()`, `exec()`, `child_process.exec()`, `Process.Start()`, `Runtime.getRuntime().exec()`
- [ ] Command argument injection: user input passed to shell without escaping
- [ ] Script runners: `eval()`, `execScript`, `runScript` with dynamic content
- [ ] Task schedulers/cron: jobs that execute user-controlled commands
- [ ] Database execution: `xp_cmdshell` (MSSQL) or `COPY FROM PROGRAM` (PostgreSQL)
- [ ] Code generation: templates generating code from user input

**Search:**
- `rg -n "os\.system\|subprocess\|exec(\|execSync\|Process\.Start\|Runtime.*exec" --include="*.{py,ts,cs,java,go}"`
- `rg -n "child_process\|spawn\|fork" --include="*.{ts,js}"`
- `rg -n "xp_cmdshell\|COPY.*PROGRAM\|pg_read_file\|pg_ls_dir" --include="*.{sql,cs,py}"`

---

### 17. Malware Indicators

**Goal:** Find obfuscation, encoded payloads, unexpected network beacons, persistence hooks, remote execution.

- [ ] Obfuscated code: `String.fromCharCode()`, large `charCodeAt` chains, `btoa`/`atob` decoding to executable, `eval(decode())` patterns
- [ ] Suspicious base64 strings: `rg -n "[A-Za-z0-9+/]{100,}={0,2}"` — large base64 blocks
- [ ] Hidden network calls: URL shorteners, raw IPs, unusual ports, `fetch()`/`XMLHttpRequest` to suspicious domains
- [ ] Crypto miners: `rg -n "miner\|cryptonight\|cryptonight\|coin.*hive\|webmine"` — including minified
- [ ] Reverse shell: `rg -n "nc\|netcat\|bash.*-i\|/dev/tcp\|Invoke-Expression\|iex.*"` — CRITICAL
- [ ] Persistence: `rg -n "schtasks\|schtasks\|TaskScheduler\|ScheduledTasks\|.bashrc\|.zshrc\|Startup\|Run.*Registry"` — unexpected
- [ ] Data exfiltration: `rg -n "nslookup\|dig.*@\|ping.*[0-9]"` — DNS tunneling indicators
- [ ] Time bombs: `rg -n "Date.*>.*new Date\|getTime.*>"` — conditional logic based on date

**Search:**
- `rg -n "atob(\|btoa(\|unescape(\|decodeURIComponent(\|base64_decode\|Convert\.FromBase64String" --include="*.{js,ts,py,cs}"` — decode chains to eval
- `rg -n "curl.*|.*sh\|wget.*|.*bash\|iex.*Invoke-WebRequest" --include="*.{sh,ps1,py,ts}"`
- `rg -n "schedule\|cron\|timer.*job\|setInterval" --include="*.{js,ts}"`

---

### 18. Infrastructure Exposure

**Goal:** Check containers, compose, Kubernetes, cloud config, public ports, root users, secret injection.

- [ ] Dockerfile: `USER root`, unnecessary packages, `ADD` from untrusted URLs, no healthcheck
- [ ] Docker Compose: exposed ports (`0.0.0.0:port`), `privileged: true`, no resource limits
- [ ] Kubernetes: privileged containers, hostPath volumes, `securityContext` missing, no network policy
- [ ] Docker socket mounted (`/var/run/docker.sock`) in containers
- [ ] Cloud IAM: over-permissive roles, public buckets, unencrypted storage
- [ ] Network: unnecessary open ports in security groups/firewalls
- [ ] Secrets: plaintext secrets in compose files, K8s ConfigMap, or environment files
- [ ] Terraform/CloudFormation: hardcoded access keys, permissive security groups

**Search:**
- `**/Dockerfile*`, `**/docker-compose*`, `**/*.yml` / `**/*.yaml` with K8s resources (Deployment, StatefulSet, Pod)
- `rg -n "privileged\|allowPrivilegeEscalation\|readOnlyRootFilesystem\|runAsNonRoot\|runAsUser" --include="*.{yml,yaml}"`
- `rg -n "AKIA\|secret.*:\|password.*:" --include="*.{yml,yaml,tf}"` — infrastructure secrets

---

### 19. CI/CD Exposure

**Goal:** Check plaintext secrets, unsafe pull request execution, publish-token leakage, artifacts, deployment gates.

- [ ] CI/CD variables in plaintext: `rg -n "password\|token\|secret\|apikey" --include="*.{yml,yaml}"` — should use `${{ secrets.* }}` or equivalent
- [ ] PR triggers on untrusted forks: `pull_request_target` (GitHub) triggers that run untrusted code in privileged context
- [ ] Artifact exposure: CI artifacts with secrets, published to public storage
- [ ] Publish-token leakage: npm/GitHub/registry tokens in CI logs or artifacts
- [ ] Deployment gates: missing manual approval for production deployment
- [ ] Self-hosted runners: PR code executed on self-hosted runner without isolation

**Search:**
- `**/.github/workflows/**`, `**/.gitlab-ci.yml`, `**/Jenkinsfile`, `**/azure-pipelines.yml`, `**/ci/**`
- `rg -n "pull_request_target\|workflow_run\|issue_comment" --include="*.{yml,yaml}"` — untrusted PR triggers
- `rg -n "NPM_TOKEN\|GITHUB_TOKEN\|GITLAB_TOKEN\|DOCKER_PASSWORD\|REGISTRY_PASSWORD" --include="*.{yml,yaml,sh}"`

---

### 20. Data Exfiltration Paths

**Goal:** Check exports, backups, logs, public buckets, debug endpoints, client-side sensitive data.

- [ ] Export/download endpoints: can user export other users' data? Is export rate-limited?
- [ ] Log aggregation: sensitive data (PII, tokens, passwords) flowing to log aggregation service
- [ ] Backup files accessible: `.sql` dumps, `.tar.gz` backups in public directories
- [ ] Public cloud storage: S3/GCS/Azure Blob buckets with public read access
- [ ] Client-side storage: sensitive data in `localStorage`/`sessionStorage`/IndexedDB (accessible via XSS)
- [ ] Debug/Admin endpoints: exposed metrics, heap dumps, thread dumps accessible externally
- [ ] PII in URLs: user IDs, emails, personal data in query params (logged in server logs, Referer header)
- [ ] GraphQL: `__schema` introspection returning full schema in production

**Search:**
- `rg -n "localStorage\|sessionStorage\|IndexedDB" --include="*.{ts,js,tsx,jsx}"` — what's stored?
- `rg -n "export\|download.*csv\|download.*pdf\|download.*xlsx" --include="*.{py,ts,cs}"`
- `rg -n "email\|phone\|ssn\|address\|birth.*date" --include="*.{py,ts,cs}"` — in logs or URLs?
- `**/*.sql`, `**/*.bak`, `**/*.dump` — backup files in repository

---

### 21. Attack-Chain Synthesis

**Goal:** Describe the most realistic end-to-end compromise path from initial access to privilege escalation, persistence, exfiltration, or denial of service.

For each HIGH/CRITICAL finding found in areas 1-20, construct one or more realistic attack chains by connecting multiple weaknesses.

### Attack Chain Template
```
## Attack Chain: [Name]
- **Initial Access:** (e.g., SSRF via webhook endpoint)
- **Escalation:** (e.g., metadata service returns IAM credentials)
- **Persistence:** (e.g., create new admin user via API)
- **Exfiltration/Impact:** (e.g., download all S3 buckets with exposed credentials)
- **Prerequisites:** (e.g., attacker needs valid tenant account, SSRF target not firewalled)
- **Realism:** (e.g., HIGH — commonly exploited in similar architectures)
- **Mitigation Priority:** (e.g., CRITICAL — block SSRF immediately)
```

### Chain Construction Rules
1. Use only real findings from this scan (do not hypothecate)
2. Connect 2+ findings from different security areas (e.g., SSRF + IAM over-permission)
3. Include realistic preconditions and attacker skill level
4. Estimate impact scope (single user, tenant, entire system, data breach)
5. Recommend defensive layering for each step in the chain

---

## Final Report Sections

### Metadata
- Target project, version, commit hash
- Scan date
- Tool versions used (rg, jq, npm audit, etc.)

### Executive Summary
- Total findings by severity: CRITICAL / HIGH / MEDIUM / LOW / INFO
- Attack chains identified
- Overall security posture: PASS / WARN / FAIL
- Top 3 risks

### Findings Table (sorted by severity)
| # | Area | Severity | File:Line | Description | Proof/Command | Fix Guidance |
|---|------|----------|-----------|-------------|---------------|-------------|

### Findings by Security Area
For each area 1-20, provide:
- PASS or list of findings
- If PASS: what was checked, how many files, key search terms

### Attack Chains
All synthesized attack chains from area 21

### PASS Summary
List all security areas with zero findings and verification methodology.

### JSON Summary
```json
{
  "verdict": "PASS|WARN|FAIL",
  "severity_counts": { "critical": 0, "high": 0, "medium": 0, "low": 0, "info": 0 },
  "files_reviewed": 0,
  "total_findings": 0,
  "attack_chains": 0,
  "scan_date": "YYYY-MM-DD",
  "target": "project-name"
}
```
