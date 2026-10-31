# Playbook 03: Cloudflare 4-Phase Security Audit & AI Agent Security

Synthesized from:
- **`cloudflare/security-audit-skill`** (4-Phase Reconnaissance, Hunting, Validation, and Reporting)
- **`agentnode-dev/skills-security-audit`** & **OWASP Top 10 for LLM & Agentic Applications (2025)**

---

## 1. The Cloudflare 4-Phase Security Audit Methodology

When asked to audit a project, feature, or codebase for security vulnerabilities, execute these 4 phases systematically:

```text
[Phase 1: Architecture Reconnaissance]
  └── Map entry points, auth middleware, database boundaries, external APIs, and privilege levels.
        │
        ▼
[Phase 2: Coverage-Led Vulnerability Hunting]
  └── Inspect each trust boundary using STRIDE + OWASP Web Top 10 + OWASP LLM Top 10.
        │
        ▼
[Phase 3: Candidate Validation & False-Positive Filtering]
  └── Trace actual data flow from source (untrusted input) to sink (DB/DOM/Shell) to prove exploitability.
        │
        ▼
[Phase 4: Actionable Remediation & Proof Verification]
  └── Provide exact file:line location, exploit scenario, and drop-in hardened code replacement.
```

---

## 2. Securing AI & Agentic Applications (OWASP LLM Top 10)

If the application integrates LLMs, AI agents, MCP servers, or local CLI runners, enforce these specialized defenses:

### A. Prompt Injection & Indirect Injection Defense (`LLM01`)
- **Treat all external content (web pages, uploaded files, database rows, chat messages) as untrusted data**, never as executable system instructions.
- **Privilege Separation:** Never rely on a system prompt (*"Do not reveal secret X"*) to enforce security. Enforce access control deterministically in backend code before data ever enters the LLM prompt.

### B. Insecure Output Handling (`LLM05`)
- **LLM Output = Untrusted User Input:**
  - Never pass raw LLM output into `eval()`, `Function()`, `child_process.exec()`, raw SQL queries, or unescaped `dangerouslySetInnerHTML`.
  - When an LLM generates structured commands or tool calls, validate every argument against a strict allowlist schema (Zod) before execution.

### C. Excessive Agency & Tool Sandboxing (`LLM06`)
- **Least Privilege for AI Tools:**
  - Read-only agents must only have read-only database/filesystem credentials.
  - Any destructive action (deleting files, dropping records, sending emails, charging payments) **MUST** require explicit human-in-the-loop confirmation (`Confirmation Gate`).

### D. Unbounded Consumption & Denial of Wallet (`LLM10`)
- Enforce strict rate limits, `max_tokens` caps, request timeouts (`AbortSignal.timeout(15000)`), and maximum agent loop iterations (`maxSteps: 10`) so a stuck loop or malicious user cannot drain API credits or hang the server.
