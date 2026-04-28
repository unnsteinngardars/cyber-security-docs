# OWASP Top 10

> The most critical web-application security risks — plus the identity concepts that underpin them.

---

## IAAA — Identity, Authentication, Authorisation, Accountability

Before talking about access-control bugs, it helps to be precise about the four pieces of the puzzle. They run in order — you **claim** an identity, **prove** it, get **permissions**, and every action is **logged**. Most OWASP top-tier risks map onto a breakdown in one of these four stages.

| Stage | Tagline | Question | What it is |
|---|---|---|---|
| **I — Identity** | The claim | *"Who are you?"* | Who you say you are. A username, email, employee ID, or client certificate subject — just an assertion, not yet verified. |
| **A — Authentication** | The proof | *"Prove it."* | Proving the claim — something you *know*, *have*, or *are*. Passwords, TOTP, hardware keys, biometrics. MFA combines factors. |
| **A — Authorisation** | The permission | *"What can you do?"* | What the proven identity is allowed to do. Role checks, ACLs, ownership checks — enforced on every sensitive request, not just at login. |
| **A — Accountability** | The record | *"Who did what, and when?"* | Every action is attributable to a specific identity through logs and audit trails. Without it, neither investigation nor non-repudiation is possible. |

---

## The Risks

OWASP ranks web-app risks by how often they appear in real apps and how much damage they cause. Three of them line up almost directly with the IAAA stages above.

---

### A01 — Broken Access Control

*Authorisation failure · users do things they shouldn't be allowed to.*

Access control decides *what* an authenticated user is allowed to reach. When the check is missing, weak, or only enforced in the UI, attackers can read or modify resources that don't belong to them — the single most common class of web-app vulnerability.

**Symptoms:** URL/ID tampering works, hidden admin endpoints are reachable directly, client-side role checks are trusted, force-browsing bypasses menus.

#### Example — IDOR (Insecure Direct Object Reference)

> **Swap the ID in the URL, read someone else's data.**
>
> The app exposes a database key (or any predictable identifier) straight in the URL or request body, and the server trusts it without checking that the current user actually owns the resource.

1. Alice views her invoice: `GET /invoice/1043` — server returns it.
2. Alice changes the ID: `GET /invoice/1044`.
3. Server authenticates the session but never checks *ownership* — returns Bob's invoice.
4. **Fix:** enforce a server-side check — `invoice.owner_id == session.user_id` — or use unguessable identifiers (UUIDs) plus per-request authorisation.

#### Horizontal vs Vertical Privilege Escalation

| | Horizontal | Vertical |
|---|---|---|
| **Direction** | Same level · different user | Lower → higher privilege |
| **What happens** | You stay at your current privilege level but reach another user's data or actions. Classic IDOR territory — Alice reading Bob's invoices, one tenant seeing another tenant's dashboard. | You move up a role — regular user to admin, read-only to write, tenant-user to platform-operator. Usually caused by missing server-side role checks or exposed admin endpoints. |
| **Example** | A standard user changes `user_id=42` to `user_id=43` and opens another customer's profile. | A standard user calls `POST /admin/users/delete` directly and it succeeds because the check lived only in the UI. |

---

### AS02 — Security Misconfigurations

*Defaults left on, unnecessary surface exposed, permissions too wide.*

The most boring risk and one of the most exploited. Every stack ships with defaults tuned for developer convenience — open dashboards, debug modes, permissive CORS, admin/admin logins. Every one of those left on in production becomes a pre-authenticated foothold.

#### Common Patterns

- Default credentials or weak passwords left unchanged
- Unnecessary services or endpoints exposed to the internet
- Misconfigured cloud storage or permissions (**S3, Azure Blob, GCP buckets**)
- Unrestricted API access or missing authentication / authorisation
- Verbose error messages exposing stack traces or system details
- Outdated software, frameworks, or containers with known vulnerabilities
- Exposed AI / ML endpoints without proper access controls

#### How to Prevent It

- Harden default configurations, remove unused features and services
- Enforce strong authentication and least privilege everywhere
- Limit network exposure and segment sensitive resources
- Patch software, frameworks, and container images regularly
- Hide stack traces and system info from user-facing errors
- Audit cloud configurations and IAM permissions on a schedule
- Secure AI endpoints with auth, rate limits, and monitoring
- Bake configuration review and automated scans into the deployment pipeline

---

### AS03 — Software Supply Chain Failures

*Trusting code, packages, and models you didn't write — and can't audit.*

Modern apps are mostly other people's code. One compromised dependency, build step, or model weight propagates to every downstream consumer. Attackers increasingly target the pipeline itself because a single successful injection hits thousands of targets at once.

#### Common Patterns

- Pulling unverified or unmaintained libraries and dependencies
- Auto-installing updates without signature or integrity checks
- Over-reliance on third-party AI models with no monitoring or auditing
- Insecure build pipelines or CI/CD processes that allow tampering
- Poor license or provenance tracking for components
- No monitoring for new vulnerabilities in dependencies after deployment

#### How to Protect the Supply Chain

- Verify third-party components, libraries, and AI models before use
- Monitor and patch dependencies regularly
- Sign, verify, and audit software updates and packages
- Lock down CI/CD pipelines and build processes against tampering
- Track provenance and licensing for every dependency
- Runtime-monitor dependencies and AI components for unusual behaviour
- Integrate supply-chain threat modelling into the SDLC — testing, deployment, and update flows

---

### AS04 — Cryptographic Failures

*Weak algorithms, leaked keys, data exposed at rest or in transit.*

Crypto problems rarely look like broken math — they look like shipping `MD5` for password hashing, committing a secret to git, or forgetting to rotate a key for three years. Most real failures are about how crypto is *used*, not the primitives themselves.

#### Common Patterns

- Deprecated or weak algorithms — `MD5`, `SHA-1`, AES in `ECB` mode
- Hard-coded secrets in source or configuration
- Poor key rotation or key-management practices
- Sensitive data unencrypted at rest or in transit
- Self-signed, expired, or invalid TLS certificates
- AI / ML systems mishandling secrets — model parameters or sensitive inputs exposed

#### How to Prevent It

- Modern primitives only — `AES-GCM`, `ChaCha20-Poly1305`, TLS 1.3 with valid certs
- Use managed KMS — **Azure Key Vault**, **AWS KMS**, **HashiCorp Vault**
- Rotate secrets and keys on a defined crypto-period
- Document and enforce key-lifecycle SOPs
- Keep a full inventory of certificates, keys, and owners
- Never let AI models or automation agents surface unencrypted secrets or sensitive data

> **Room note** — The web application in this room contains a weakness of this type. Find and exploit it.

---

### AS06 — Insecure Design

*Flaws baked into the architecture — no patch fixes a bad design.*

Insecure design isn't a bug — it's the absence of threat modelling. A perfectly coded feature can still be insecure if the flow itself is exploitable (weak password-reset, approval chains that can be skipped, AI agents with unchecked authority). You cannot patch your way out of a broken flow.

#### Common Insecure Designs · 2025

- Weak business-logic controls — recovery and approval flows
- Flawed assumptions about user or model behaviour
- AI components with unchecked authority or access
- Missing guardrails for LLMs and automation agents
- Test or debug bypasses left in production
- No consistent abuse-case review or AI threat modelling

#### Insecure Design in the AI Era

- **Prompt injection** — User input is blended with the system prompt, letting attackers hijack context or extract hidden instructions and data.
- **Blind trust in output** — Systems that act on AI decisions without validation or human review. One confident hallucination and a downstream action fires.
- **Poisoned models** — Models pulled from unverified sources or fine-tuned on unsafe data can embed hidden behaviours or backdoors that compromise the system from within.

#### How to Design Securely

- Treat every model as untrusted until proven otherwise
- Validate and filter all model inputs *and* outputs
- Separate system prompts from user content
- Keep sensitive data out of prompts unless strictly needed, then protect it with strict controls
- Require human review for high-risk AI actions
- Log model provenance, monitor behaviour, apply differential privacy for sensitive data
- Threat-model for prompt attacks, inference risks, agent misuse, and supply-chain compromise
- Build threat modelling into every stage, not just the start
- Define security requirements per feature, before implementation
- Apply least privilege across users, APIs, and services
- Proper authentication, authorisation, and session management across the system
- Keep dependencies and third-party components verified and current
- Continuously monitor and test for logic flaws, abuse paths, and emergent risks as features change

---

### A07 — Identification & Authentication Failures

*Authentication failure · proving identity is broken, weak, or bypassable.*

Covers every weakness in how the app confirms *who* a user is — the login, registration, password-reset, and session layers. When any of these fail, attackers don't need to escalate privilege; they simply log in as someone else.

#### Common Failure Modes

- **Username enumeration** — The app leaks which usernames exist through **different error messages** (`user not found` vs `wrong password`), response timing, or registration / password-reset endpoints. Hands attackers a pre-filtered target list for credential stuffing and brute force.
- **Weak / guessable passwords** — No minimum strength, no breach-list check, and **no lockout or rate limits** on the login endpoint. Attackers can try millions of passwords or known breach credentials with no friction.
- **Logic flaws in login / registration** — Multi-step flows that can be skipped, parameters that can be tampered with, or **server-side state that relies on client input**. Examples: registering as `role=admin`, skipping the MFA step, or resetting another account's password because the token is predictable.
- **Insecure session / cookie handling** — Session IDs exposed in URLs, not rotated after login, never expired, or stored without `Secure` / `HttpOnly` / `SameSite`. Enables **session fixation**, theft via XSS, or replay long after logout.

---

### A09 — Security Logging & Monitoring Failures

*Accountability failure · you can't see attacks, so you can't stop or investigate them.*

The defender's blind spot. An attacker who breaches an app with no logging can dwell for months; the average breach goes undetected for hundreds of days precisely because auth, access, and admin events weren't recorded — or no one was watching them.

This isn't just "turn on logging." It covers what you *capture*, how reliably you *store* it, whether it's *alerted on* in near-real-time, and whether the org has a response plan when alerts fire.

#### Typical Failures

- **Auth events not logged** — Logins, failed logins, MFA prompts, and password resets aren't recorded — so brute-force and credential-stuffing attacks are invisible.
- **High-value actions untracked** — Privilege changes, data exports, payment approvals, and admin actions go unlogged or log without context (*who*, *from where*, *what changed*).
- **No alerting / no one watching** — Logs are written but never aggregated into a SIEM, and nothing alerts on suspicious patterns (spikes in 401s, new admin role grants, impossible-travel logins).
- **Logs local, tamperable, short-lived** — Stored only on the compromised host, writable by the app user, and rotated out in days — so an attacker who lands on the box can clear their tracks.
- **Logging sensitive data** — Writing passwords, tokens, session IDs, or full PII into logs. The log pipeline itself becomes a secondary breach target.
- **No incident-response plan** — Even when alerts fire, there's no defined runbook, owner, or escalation path. Detection without response is just a louder way to miss the breach.

#### What Good Looks Like

- **Log** auth, access-control, and admin events with full context
- **Ship** centrally to an append-only store (SIEM)
- **Alert** on patterns, not just single events
- **Retain** long enough to catch slow intrusions
- **Scrub** secrets and PII before they hit the pipeline
- **Rehearse** the response runbook — detection without action is noise

---

## IAAA → OWASP Mapping

> A01 is an **Authorisation** failure (A#3 of IAAA), A07 is an **Authentication** failure (A#2), and A09 is an **Accountability** failure (A#4). Identity itself — the claim — is rarely the vulnerability; the failure is almost always in how the claim is verified, scoped, or recorded.

---

*OWASP Top 10 · Identification, Authentication, Authorisation & Accountability are the foundation every web-app security control sits on top of.*
