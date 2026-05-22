# Small Scan Prompt Group

Role: You are a security reviewer running a fast, evidence-based scan. Read `../wiki/{target-project-name}/00-single-source-of-truth.md` before starting. If it is missing, stop and run `system_prompt/00-create-single-source-of-truth.md`.

Run the following 12 primary scan prompts. Keep each prompt isolated and save the merged result to `prompts/result/scan/small/result-small-scan.md`.

## Output Rules

- Do not modify code.
- Save only to `prompts/result/scan/small/`.
- Every finding must include: **category** | **file:line** | **evidence snippet** | **severity** | **fix guidance** | **verification command**
- If no issue is found, record a PASS note with what was checked.

### Severity Levels
| Level | Criteria |
|-------|----------|
| CRITICAL | Remote code execution, credential exposure (live), SQL injection |
| HIGH | XSS, auth bypass, IDOR, SSRF, insecure deserialization, hardcoded secrets (dev/config) |
| MEDIUM | Missing rate limit, permissive CORS, verbose error, weak crypto |
| LOW | Missing security header, cookie missing flags, debug endpoint exposed |
| INFO | Observation, hardening suggestion, future risk |

---

## Prompt 01 - Hardcoded Secrets

**Goal:** Find API keys, tokens, private keys, passwords, signing secrets, cloud credentials in source, config, CI, env examples, and docs.

### Search methodology
1. **Pattern matching:** `grep -rn "-----BEGIN.*KEY\|sk_live_\|pk_live_\|AKIA[0-9A-Z]\{16\}\|ghp_\|ghs_\|gho_\|xox[baprs]-\|AIza[0-9A-Za-z_-]\{35\}\|SG\.[a-zA-Z0-9]"

2. **Common filenames:** `**/.env*`, `**/*.pem`, `**/*.key`, `**/credentials*`, `**/id_rsa*`, `**/secrets*`, `**/secret*`

3. **CI exposure:** `grep -rn "password\|token\|secret\|apikey\|api_key" --include="*.{yml,yaml}"` in `.github/`, `.gitlab-ci.yml`, etc.

4. **Example configs:** `**/*.example*`, `**/*.sample*`, `**/*.template*` for placeholder values that look real

### Validating findings
- Is the value a known pattern (regex) or just a variable name?
- Is the file committed (check .gitignore)?
- Is the credential still valid? Do not test against live services.

---

## Prompt 02 - Injection Hotspots

**Goal:** Find SQL, NoSQL, LDAP, template, shell, XPath, expression injection sinks.

### Sub-checks
- [ ] Raw SQL construction via string concatenation: `grep -rn "SELECT.*+.*WHERE\|DELETE.*+\|INSERT.*+.*VALUES\|UPDATE.*+"` — includes `.py`, `.ts`, `.go`, `.cs`, `.java`
- [ ] ORM raw queries: `grep -rn "\.raw(\|raw_sql\|RawSql\|ExecuteSqlRaw\|FromSqlRaw\|NativeQuery"`
- [ ] Template injection: `grep -rn "render_template_string\|Template(\|\.parse(\|eval_template\|handlebars\.compile"` (user input in templates)
- [ ] Shell injection: `grep -rn "os\.system\|subprocess\.Popen\|exec\|child_process\.exec\|Runtime\.getRuntime\(\)\.exec"` with user input
- [ ] LDAP injection: `grep -rn "ldap\|LDAP\|DirectorySearcher\|LdapConnection"` (input not sanitized)
- [ ] NoSQL injection: `grep -rn "find(.*\$where\|find(.*\$regex\|find(.*\$ne\|find(.*\$gt"` (MongoDB operators from user input)

### Validation
For each sink, trace back: is user input (body, query, header, param) reaching this function unsanitized?

---

## Prompt 03 - XSS and Client-Side Injection

**Goal:** Review HTML rendering, markdown, template escapes, DOM writes, rich text.

### Sub-checks
- [ ] `innerHTML`, `outerHTML`, `insertAdjacentHTML` usage: `grep -rn "\.innerHTML\|\.outerHTML\|insertAdjacentHTML" --include="*.{ts,js,tsx,jsx,vue,svelte}"`
- [ ] `v-html`, `dangerouslySetInnerHTML`, `{{{`, `{@html` usage: `grep -rn "v-html\|dangerouslySetInnerHTML\|{@html" --include="*.{html,tsx,jsx,vue,svelte}"`
- [ ] Markdown renderers: `grep -rn "marked\|markdown-it\|remark\|showdown\|marked"` — check if XSS sanitization enabled
- [ ] URL injection in `href`/`src`/`action`: `grep -rn "href=\|src=\|action="` with dynamic content — check for `javascript:` protocol
- [ ] `eval`-like: `grep -rn "eval(\|setTimeout.*string\|setInterval.*string\|new Function(" --include="*.{ts,js}"`

### Validation
- Is user-controlled content reaching any of these sinks?
- Is there a DOMPurify/sanitize-html/CSP mitigation in place?

---

## Prompt 04 - Access Control and IDOR

**Goal:** Find routes/handlers where a user-supplied ID accesses another user's resource.

### Sub-checks
- [ ] Resource IDs from URL params: `grep -rn "req\.params\|request\.params\|:id\|{id}\|/{id}\|/:id"` — for each, check if owner check exists
- [ ] Missing ownership check pattern: `grep -rn "\.findById\|\.findOne\|\.getById\|FirstOrDefault\|\.FindAsync"` near controller actions
- [ ] Admin endpoints without role check: `grep -rn "admin\|/api/v1/admin\|AdminOnly\|RequireAdmin"` — then verify authorization attribute
- [ ] GraphQL resolvers with missing auth: `grep -rn "resolver\|Query\|Mutation"` — check each has auth directive
- [ ] Direct file access: `grep -rn "res\.sendFile\|res\.download\|FileStreamResult\|PhysicalFile"` — check path traversal and auth

### Validation
- Can user A access user B's data by changing an ID in the URL/body?
- Is authorization checked serverside or only in the UI?

---

## Prompt 05 - Authentication and Session Safety

**Goal:** Review login, logout, password reset, MFA, cookies, JWT, sessions.

### Sub-checks
- [ ] Password hashing: `grep -rn "bcrypt\|argon2\|scrypt\|PBKDF2\|password_hash"` — reject md5/sha1/sha256 hashing
- [ ] JWT: `grep -rn "jwt\|JWT\|jsonwebtoken"` — verify algorithm validation (no `none`), expiry check, secret strength
- [ ] Session: `grep -rn "session\|cookie"` — httpOnly, secure, sameSite flags; no predictable session IDs
- [ ] Login rate limit: `grep -rn "rate.?limit\|throttle\|brute.*force\|lockout\|max.*attempt"` on login routes
- [ ] Password reset: `grep -rn "reset\|forgot\|forgot-password"` — check token expiry (>= 15 min, <= 1 hour), single-use
- [ ] MFA/2FA: `grep -rn "mfa\|2fa\|totp\|otp\|authenticator"` — if in docs but not in code, flag
- [ ] Remember-me: `grep -rn "remember\|persist.*login\|keep.*login"` — check token rotation and expiry
- [ ] Logout: verify session is destroyed server-side, tokens invalidated

### Validation
- `curl -X POST ...` to test missing auth on protected routes
- Decode JWT to check algorithm, payload, expiry

---

## Prompt 06 - CSRF and CORS

**Goal:** Check state-changing endpoints, cookie usage, CSRF tokens, CORS configuration.

### Sub-checks
- [ ] CSRF protection exists for cookie-authenticated state-changing endpoints: `grep -rn "csrf\|csrfProtect\|anti.*forgery\|_csrf\|X-CSRF"` — if absent on POST/PUT/DELETE with cookies, flag HIGH
- [ ] SameSite cookie policy: check if cookies have `SameSite=Lax` or `SameSite=Strict`
- [ ] CORS permissive: `grep -rn "Access-Control-Allow-Origin\|cors()\|enableCors"` — reject `*` with credentials
- [ ] CORS allowed origins: verify against `Access-Control-Allow-Origin: null` or regex bypasses
- [ ] Preflight (`OPTIONS`): check if CORS preflight is restricted by origin, method, headers

### Search patterns
- `grep -rn "cors\|CorsPolicy\|enableCors\|@CrossOrigin" --include="*.{py,ts,cs,java,yaml,yml}"`
- `grep -rn "SameSite\|same_site\|sameSite" --include="*.{py,ts,cs,java}"`
- `grep -rn "csrf\|X-CSRF\|CSRF\|_csrf\|csrfProtect" --include="*.{py,ts,cs,java}"`

---

## Prompt 07 - File Upload and Path Traversal

**Goal:** Review upload endpoints, archive extraction, filename handling, download routes.

### Sub-checks
- [ ] File extension validation: `grep -rn "\.ext\|extension\|\.pdf\|\.jpg\|mimetype\|content-type"` — reject by allowlist, not blocklist
- [ ] Content sniffing: `grep -rn "Content-Type\|magic.*number\|file.*signature\|MimeType\|detect_content"` — validate content, not just extension
- [ ] Path traversal in download: `grep -rn "\.\.\/\|\.\.\\\\|path.*join\|Path\.Combine\|path\.join\|normalize.*path"` — ensure `..` is rejected
- [ ] File size limits: `grep -rn "max.*size\|maxSize\|upload.*limit\|bodyParser.*limit"` — ensure limit exists (e.g., 10MB)
- [ ] Archive extraction: `grep -rn "unzip\|unzip\|tar.*extract\|decompress\|extractArchive"` — check zip-slip protection, decompression bomb guard
- [ ] Storage isolation: uploaded files should not be in webroot or executable path
- [ ] Executable payload: `grep -rn "\.exe\|\.sh\|\.bat\|\.ps1\|\.jar\|\.wasm"` in allowed extensions

### Search patterns
- `grep -rn "upload\|file\|blob\|storage\|s3\|azure.*blob\|gcs" --include="*.{py,ts,go,java,cs}"`
- `grep -rn "multipart\|form-data\|formdata\|IFormFile\|UploadedFile" --include="*.{py,ts,cs,java}"`

---

## Prompt 08 - SSRF and Network Egress

**Goal:** Search for server-side URL fetches, webhook validation, proxy endpoints, metadata service access.

### Sub-checks
- [ ] Server-side HTTP requests: `grep -rn "fetch(\|axios\.\|requests\.\|HttpClient\|WebClient\|RestTemplate\|curl"` — check user-controlled URL
- [ ] Webhook handlers: `grep -rn "webhook\|hook\|callback"` — check if URL is user-provided, verified with allowlist
- [ ] Image/URL importers: `grep -rn "import\|download\|fetch.*image\|scrape\|proxy"` — check URL validation
- [ ] Cloud metadata endpoints: `grep -rn "169\.254\.169\.254\|metadata.*google\|metadata\.amazonaws\|metadata\.azure"` — blacklisted?
- [ ] Internal network: `grep -rn "localhost\|127\.0\.0\.1\|10\.\|172\.\(1[6-9]\|2[0-9]\|3[01]\)\|192\.168\|internal"` — verify NOT bypassable
- [ ] Redirect following: `grep -rn "follow.*redirect\|allow.*redirect\|redirect.*follow\|max.*redirect"` — check Not following by default

### Validation
- Can user provide a `url` param that's fetched server-side? Against what allowlist?
- Can they redirect to `file://`, `gopher://`, `dict://`, or internal IPs?

---

## Prompt 09 - Deserialization and Unsafe Parsing

**Goal:** Look for unsafe deserialization, dynamic import, eval, pickle, YAML, XML.

### Sub-checks
- [ ] Pickle/unsafe Python: `grep -rn "pickle\.loads\|pickle\.load\|cPickle\|dill\|shelve\|marshal"` — CRITICAL if user-controlled
- [ ] YAML: `grep -rn "yaml\.load\|yaml_load\|Yaml\.load\|YamlParser"` (not `yaml.safe_load`) — HIGH
- [ ] XML: `grep -rn "lxml\|SAX\|DOM\|XmlDocument\|XDocument\|XmlReader"` — check XXE: `<!ENTITY`, `DOCTYPE`, external entity resolution disabled
- [ ] Dynamic import/exec: `grep -rn "importlib\|__import__\|exec(\|compile(\|eval(\|new Function\|Function("` — HIGH if user input reaches these
- [ ] JSON with prototype pollution: `grep -rn "__proto__\|constructor\.prototype\|merge(\|assign("` — check if user-controlled
- [ ] .NET `BinaryFormatter`, `SoapFormatter`, `LosFormatter`: `grep -rn "BinaryFormatter\|SoapFormatter\|LosFormatter\|JavaScriptSerializer"` — CRITICAL
- [ ] Ruby `Marshal.load`, `YAML.load`: `grep -rn "Marshal\.load\|Marshal\.restore\|YAML\.load"`

### Validation
- Trace if any untrusted data (API body, query param, file upload) reaches these sinks
- Check for type allowlists or safe parsing alternatives

---

## Prompt 10 - Brute Force and Rate Limiting

**Goal:** Check login, OTP, password reset, invite, webhook, expensive endpoints for throttling.

### Sub-checks
- [ ] Login rate limit (per-IP and per-account): `grep -rn "rate.?limit\|RateLimit\|throttle\|Throttle\|TokenBucket\|LeakyBucket\|SlidingWindow"` applied to `/login`, `/auth`, `/signin`
- [ ] OTP/resend endpoint limited: `grep -rn "otp\|totp\|verify-otp\|resend"` — check if rate limited
- [ ] Password reset limited: `grep -rn "reset\|forgot-password\|forgot"` — per-account, per-IP
- [ ] Account enumeration: login and password reset should return identical messages for existing/non-existing users
- [ ] Invite endpoint limited: `grep -rn "invite\|register\|signup"` — prevent mass registration
- [ ] Webhook/expensive endpoint: `grep -rn "webhook\|callback\|execute\|render\|export"` — check for abuse protection
- [ ] Lockout policy: account locks after N failed attempts, unlocks after timeout
- [ ] Telemetry: failed attempts logged with IP, user, timestamp, user agent

### Search patterns
- `grep -rn "rate\|limit\|throttl\|429\|TooManyRequests\|Too Many Requests" --include="*.{py,ts,cs,java,yaml,yml}"`
- `grep -rn "middleware\|middleware\|decorator\|attribute\|filter" --include="*.{py,ts,cs,java}"` around auth routes

---

## Prompt 11 - Dependency and Supply Chain

**Goal:** Review package manifests, lockfiles, install scripts, unpinned versions, suspicious packages.

### Sub-checks
- [ ] Unpinned dependencies: `grep -rn "\"package\": \">=\|\"package\": \"\^\|\"package\": \"~\|\"package\": \"\*\|\"package\": \"x\.\|name.*:.*\*$"` — flag unpinned major/minor
- [ ] Known vulnerabilities: if possible, run `npm audit`, `pip-audit`, `dotnet list package --vulnerable`, `go list -m all` + `govulncheck`
- [ ] Typo-suspicious packages: look for names similar to popular packages (e.g., `event-stream` vs `event-stream-`, `babel-eslint` vs `babel-eslint-`)
- [ ] Install scripts: `grep -rn "postinstall\|preinstall\|prebuild\|postbuild\|preuninstall" --include="*.json"` — review for malicious activity
- [ ] Vendored binaries: `**/*.exe`, `**/*.dll`, `**/*.so`, `**/*.dylib` in repo — review origin
- [ ] Lockfile drift: `package.json` vs `package-lock.json` / `yarn.lock` mismatch
- [ ] Deprecated/unmaintained packages: review for packages not updated in > 2 years
- [ ] Indirect dependency risk: deep dependency chain with no review

### Search patterns
- `**/package.json`, `**/requirements.txt`, `**/Pipfile*`, `**/poetry.lock`, `**/go.mod`, `**/go.sum`, `**/*.csproj`, `**/*.fsproj`, `**/Cargo.toml`, `**/Gemfile`, `**/Gemfile.lock`
- `grep -rn "\"scripts\"" --include="package.json"` (install scripts)

---

## Prompt 12 - Malware and Command Execution Indicators

**Goal:** Search for persistence hooks, obfuscated code, encoded payloads, network beacons, shell execution, schedule creation.

### Sub-checks
- [ ] Obfuscation: `grep -rn "String\.fromCharCode\|unescape\|decodeURIComponent\|atob\|base64_decode\|Convert\.FromBase64String\|System\.Text\.Encoding\.UTF8\.GetString"` — check if decoding to executable code
- [ ] Encoded/long numeric strings: inspect `grep -rn "[0-9]\{50,\}"` for suspicious large numbers (potential encoded payload)
- [ ] Remote download + execute: `grep -rn "curl.*|.*sh\|wget.*|.*bash\|Invoke-WebRequest.*\|iex\|Start-Process.*-Uri"` — CRITICAL
- [ ] Shell execution: `grep -rn "os\.system\|subprocess\.call\|exec\|Runtime.*exec\|Process\.Start\|child_process\.exec\|spawn"` — check what's being executed
- [ ] Persistence hooks: `grep -rn "cron\|task.?scheduler\|schtasks\|systemd.*service\|Startup\|Run.*Registry\|\.bashrc\|\.zshrc\|profile"` — unexpected scheduled tasks
- [ ] Network beacons: `grep -rn "setInterval\|setTimeout.*1000\|while.*true.*sleep\|\.loop"` — periodic callbacks to external URLs
- [ ] Reverse shell: `grep -rn "nc\|netcat\|bash.*-i\|/dev/tcp/\|Invoke-Expression.*"` — immediate CRITICAL
- [ ] Startup scripts: `**/.vscode/tasks.json`, `**/.git/hooks/*` — unexpected code execution

### Validation
- Trace obfuscated string decoding to determine if it becomes executable code
- Check cron/cronjob/task definitions against project documentation
