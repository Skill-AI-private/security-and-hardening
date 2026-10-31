# Playbook 04: User Trust Architecture, Next.js Security Headers & `SECURITY.md`

User trust is built on two pillars: **Technical Impenetrability** (preventing breaches) and **Transparent Governance** (showing users how their data is protected).

---

## 1. Production HTTP Security Headers (Next.js `next.config.ts` Recipe)

Every web application must ship with hardened HTTP response headers to block XSS, clickjacking, MIME sniffing, and downgrade attacks:

```ts
// next.config.ts
import type { NextConfig } from 'next';

const securityHeaders = [
  {
    // Prevent clickjacking by forbidding iframe embedding
    key: 'X-Frame-Options',
    value: 'DENY',
  },
  {
    // Block MIME-type sniffing
    key: 'X-Content-Type-Options',
    value: 'nosniff',
  },
  {
    // Protect user privacy when navigating to external links
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin',
  },
  {
    // Enforce HTTPS strictly for 2 years
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload',
  },
  {
    // Restrict browser hardware access (Camera, Mic, Geolocation) unless needed
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=(), browsing-topics=()',
  },
];

const nextConfig: NextConfig = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: securityHeaders,
      },
    ];
  },
};

export default nextConfig;
```

---

## 2. The 5 Commandments of User Trust (Privacy by Design)

1. **Data Minimization:** Never collect personal data (PII) that the application does not strictly need to function. Data you do not store cannot be leaked.
2. **Local-First & Zero-Knowledge Option:** For sensitive user tools (API keys, private documents, chat history), prefer encrypting data at rest or storing sensitive user keys locally on the user's device rather than transmitting them to third-party servers.
3. **Honest Delete (Right to Erasure):** When a user clicks *"Xóa tài khoản / Xóa dữ liệu"* (Delete Account/Data), perform a true cascading delete (`ON DELETE CASCADE`) across primary tables, caches, and storage buckets—not a fake soft-delete that secretly retains their private content.
4. **Sanitized Error & Audit Logs:** Log security events (failed logins, permission denials) with timestamps and request IDs, but **strip all passwords, tokens, and private message bodies** from server logs.
5. **Visible Security Posture (`SECURITY.md`):** Include a clear `SECURITY.md` in the repository root so users and security researchers know exactly how the tool protects their data.

---

## 3. Standard `SECURITY.md` Template for Your Projects

```markdown
# Security Policy & User Trust Commitment

## Our Security Philosophy
User trust is our highest priority. This project is engineered under a **Zero-Trust, Privacy-First** architecture:
- **Zero Plaintext Secrets:** All credentials and tokens are isolated in environment variables or encrypted stores.
- **Strict Input Validation:** Every external input and API boundary is validated via strict runtime schemas.
- **Least-Privilege Access:** Database tables enforce Row Level Security (RLS) and strict ownership verification.
- **Supply-Chain Integrity:** Dependencies are locked and audited against known CVEs prior to every release.

## Reporting a Vulnerability
If you discover a potential security issue, please report it privately rather than opening a public issue so we can issue a verified patch immediately.
```
