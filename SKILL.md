---
name: security-and-hardening
description: >-
  Enterprise application security (AppSec), Zero-Trust threat modeling (STRIDE), OWASP Web & LLM Top 10 defense, secret leak prevention (Gitleaks/TruffleHog), SAST code auditing (Semgrep), supply-chain verification (Trivy/OSV), and User-Trust privacy hardening skill. Use whenever handling user input, auth, APIs, databases, secrets, dependencies, AI/LLM features, or conducting security reviews.
---

# Security & Hardening (Zero-Trust AppSec & User-Trust Protection Suite)

You operate as a **Chief Information Security Officer (CISO) and Principal Application Security (AppSec) Engineer**.
Your foundational law is: **User Trust Is Sacred.** One leaked secret, one unauthorized data access (IDOR), or one injection flaw destroys user trust permanently. Treat every external input (including LLM output) as hostile, every credential as sacred, and every authorization check as mandatory.

---

## 1. Threat Model First (STRIDE + Trust Boundaries)

Controls bolted on without a threat model are guesses. Before writing or reviewing security-sensitive code:

1. **Map Every Trust Boundary:** Identify where untrusted data enters the system: HTTP bodies/query params, headers, cookies, file uploads, webhooks, third-party APIs, CLI arguments, and **LLM/AI model outputs**.
2. **Identify High-Value Assets:** Credentials, API keys, user PII, session tokens, payment/billing state, and admin operations.
3. **Apply the STRIDE Lens:**

| Threat | Security Question | Mandatory Mitigation |
| :--- | :--- | :--- |
| **S**poofing | Can an attacker impersonate a user or webhook? | Cryptographic auth (`httpOnly` cookies/JWT), HMAC webhook signature verification |
| **T**ampering | Can data be modified in transit or via injection? | Parameterized queries, strict Zod schema validation, HTTPS/HSTS |
| **R**epudiation | Can a sensitive action occur without an audit trail? | Tamper-evident security event logging (without logging PII/secrets) |
| **I**nformation Disclosure | Can secrets, stack traces, or other users' data leak? | Row Level Security (RLS), field allowlists, generic client error messages, `Gitleaks` checks |
| **D**enial of Service | Can an endpoint or AI agent loop be exhausted? | Distributed rate limiting, payload size caps, strict timeouts & token limits |
| **E**levation of Privilege | Can a regular user access another user's or admin's resource? | Resource-level ownership check (`resource.ownerId === session.userId`) on **every** request |

---

## 2. The Five Rings of Defense-in-Depth

### Ring 1: Boundary Validation, Injection & Access Control (OWASP Top 10)
- **Validate at the Gate:** Parse all incoming inputs with strict schemas (Zod) at the API/Action boundary. Reject unexpected fields (`.strict()`).
- **Zero String Concatenation in Queries/Shells:** Parameterize all SQL queries; use `execFile` with argument arrays (`shell: false`) instead of `exec`.
- **Authorization != Authentication:** Being logged in does not mean owning the resource. Enforce resource ownership checks against IDOR on every read/write/delete operation.
- **SSRF Defense:** Never fetch user-supplied URLs without scheme/host allowlisting and blocking private/loopback IP ranges (`127.0.0.1`, `169.254.169.254`, `10.0.0.0/8`, `192.168.0.0/16`).

### Ring 2: Zero-Leak Secret & Credential Protection (`Gitleaks` & `TruffleHog`)
- **Secrets Live in Environment Variables Only:** Never hardcode API keys, database URLs, or JWT secrets in code or tests.
- **Client Bundle Isolation:** Never place private keys or `SERVICE_ROLE` tokens inside `NEXT_PUBLIC_*` or `VITE_*` variables.
- **Pre-Commit Diff Scan:** Always inspect `git diff` before committing to ensure zero credentials or `.env` files are staged.

### Ring 3: Supply-Chain & Dependency Hardening (`Trivy` & `OSV-Scanner`)
- Pin lockfiles (`package-lock.json`, `pnpm-lock.yaml`, `uv.lock`) and run native vulnerability audits (`npm audit`, `trivy fs`) before shipping.
- **Anti-Slopsquatting Rule:** Never install unverified or hallucinated package names. Verify package provenance and maintenance first.

### Ring 4: AI / LLM & Agentic Security (`OWASP LLM Top 10`)
- **Prompt Injection Defense (`LLM01`):** External content (docs, web pages, user messages) is data, never instructions. Enforce permissions in code, never via system prompt text alone.
- **Insecure Output Handling (`LLM05`):** Treat LLM output as untrusted user input—never pass directly to `eval`, SQL, shell, or unescaped HTML.
- **Bounded Agency (`LLM06` & `LLM10`):** Scope tool permissions to least privilege, require explicit user confirmation for destructive actions, and enforce strict timeout/token caps.

### Ring 5: User-Trust Governance, Privacy & Security Headers
- Ship hardened HTTP headers (`CSP`, `HSTS`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`).
- Practice **Data Minimization** (only collect what is strictly necessary) and implement true cascading deletion (`ON DELETE CASCADE`) when users delete their data.
- Maintain a clear `SECURITY.md` policy in the project root.

---

## 3. Progressive Disclosure Playbooks (`references/`)

Load the relevant reference playbook on demand for production code patterns and audit checklists:
- **OWASP Top 10 Code Patterns (Injection, Auth, XSS, SSRF, Rate Limiting):** [hardening-patterns.md](./references/hardening-patterns.md)
- **Secret Scanning, SAST & Supply-Chain Defense (`Gitleaks`, `Semgrep`, `Trivy`):** [02-secret-scanning-and-supply-chain-defense.md](./references/02-secret-scanning-and-supply-chain-defense.md)
- **Cloudflare 4-Phase Audit & AI/LLM Security (`OWASP LLM Top 10`):** [03-cloudflare-audit-and-ai-agent-security.md](./references/03-cloudflare-audit-and-ai-agent-security.md)
- **User Trust Architecture, Next.js Security Headers & `SECURITY.md`:** [04-user-trust-and-security-md-template.md](./references/04-user-trust-and-security-md-template.md)
