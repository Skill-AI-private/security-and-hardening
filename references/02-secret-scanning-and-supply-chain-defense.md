# Playbook 02: Secret Scanning, SAST & Supply-Chain Defense (`Gitleaks` + `TruffleHog` + `Semgrep` + `Trivy`)

Synthesized from the industry's most trusted open-source security engines:
- **`gitleaks/gitleaks`** & **`trufflesecurity/trufflehog`** (Secret & credential leak prevention)
- **`semgrep/semgrep`** (Deterministic Static Application Security Testing - SAST)
- **`aquasecurity/trivy`** & **`google/osv-scanner`** (Supply-chain CVE & lockfile auditing)

---

## 1. Zero-Leak Secret Defense (`Gitleaks` & `TruffleHog` Standard)

A single leaked API key, database URI, or signing secret destroys user trust immediately. Enforce these mandatory controls:

### A. Pre-Commit Secret Audit
Before running `git commit` or `git push`, always scan staged changes for high-entropy strings and known token patterns:
```bash
# If gitleaks is installed:
gitleaks protect --staged --verbose

# Or fast regex audit on staged diff before committing:
git diff --cached | grep -E "(?i)(api[_-]?key|secret|password|token|private[_-]?key|sk-[a-zA-Z0-9]{20,}|ghp_[a-zA-Z0-9]{36}|AIza[0-9A-Za-z-_]{35})"
```

### B. Environment Variable Isolation Architecture
1. **Never hardcode secrets** in source files, test fixtures, or configuration scripts.
2. **Client vs. Server Env Separation (Next.js / Web):**
   - Variables prefixed with `NEXT_PUBLIC_` or `VITE_` are **bundled into public browser JavaScript**.
   - **IRON LAW:** Never put database URLs, service-role keys (`SUPABASE_SERVICE_ROLE_KEY`), OpenAI/Anthropic API keys, or JWT secrets into `NEXT_PUBLIC_*` variables.
3. **Mandatory `.gitignore` Baseline:**
   Ensure `.gitignore` always blocks:
   ```gitignore
   .env
   .env.*
   !.env.example
   *.pem
   *.key
   *credentials*.json
   ```

---

## 2. Static Code Analysis (`Semgrep` High-Confidence Rules)

When writing or reviewing code, actively scan for and eliminate these high-severity SAST patterns:

| Vulnerability Class | Dangerous Pattern (Forbidden) | Hardened Pattern (Required) |
| :--- | :--- | :--- |
| **Command Injection** | `exec(\`git clone \${userInput}\`)` | `execFile('git', ['clone', validatedUrl], { shell: false })` |
| **SQL Injection** | `db.query(\`SELECT * FROM users WHERE email = '\${email}'\`)` | Parameterized queries (`$1`, `?`) or typed ORM bindings |
| **Path Traversal** | `fs.readFileSync(path.join(baseDir, req.query.file))` | Resolve realpath & verify `resolvedPath.startsWith(allowlistedRoot + path.sep)` |
| **Prototype Pollution** | Deep merging raw JSON payloads into config objects | Validate with strict Zod schema (`z.object({...}).strict()`) before merging |
| **Open Redirect** | `redirect(req.query.next)` | Allowlist relative paths starting with single `/` (`/^\/(?!\/)/`) |

---

## 3. Supply-Chain & Dependency Hardening (`Trivy` & `OSV-Scanner`)

90% of modern web code comes from third-party packages. Protect the supply chain:

1. **Lockfile Integrity:**
   - Always commit `package-lock.json`, `pnpm-lock.yaml`, `uv.lock`, or `Cargo.lock`.
   - In CI and production builds, use immutable installs (`npm ci`, `pnpm install --frozen-lockfile`).
2. **Typosquatting & Hallucinated Package Defense (Slopsquatting):**
   - **AI Slopsquatting Warning:** Attackers register fake npm/PyPI package names that LLMs commonly hallucinate.
   - **Rule:** Never add a package to `package.json` or `requirements.txt` unless you are 100% certain of its exact official package name, maintainer, and weekly adoption.
3. **Automated Vulnerability Scan Commands:**
   ```bash
   # Node.js native audit
   npm audit --audit-level=high

   # Trivy filesystem & dependency scan (if installed)
   trivy fs --scanners vuln,secret,misconfig .
   ```
