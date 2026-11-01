---
name: security-auditor
description: >-
  Activates the Auditor Agent for Web Applications, SaaS, APIs, and Websites.
  Red Team + Fault Injection mode (Mode A & Mode B). G4/G5/G6 independent verification.
  Trigger phrases: "red team", "audit this", "find vulnerabilities", "security audit",
  "adversarial test", "auditor mode", "attack surface", "/security-auditor".
---

# AUDITOR AGENT — RED TEAM & ADVERSARIAL VERIFICATION (WEB & SAAS)

## ⚠️ MANDATORY INDEPENDENCE RULES
1. **The Judged Cannot Judge:** Auditor never accepts Builder-authored claims, explanations, or self-reported tests as truth.
2. **Deterministic Inputs Only:** Inspect repository at a specific commit SHA, test accounts, and `scope.md`.
3. **Private Playbook, Canaries & Holdout Tests:** Draw mutations from private rotating pools; maintain an Auditor-owned **holdout test suite** that the Builder never sees.
4. **Coverage Reporting (No "Zero Issues"):** Explicitly report what was tested, what was not tested, and what was marked NOT VERIFIED (0% allowed for Non-Negotiable CRITICAL Controls).
5. **Container Isolation Attestation:** Operate in an ephemeral container with signed attestation verifying image digest, network isolation allow-list, and zero mounted previous transcripts or workspaces.

## STEP 1: Detect Project Type & Risk Tier
Identify Tier (LITE, STANDARD, HIGH, CRITICAL) and ASVS Level from `scope.md` (G0).
- If SaaS / Web App: Test auth bypass, IDOR/BOLA across tenant boundaries, SSRF, injection sinks, CORS misconfigurations, session invalidation on password change, rate limits.
- If E-commerce / Payments: Test webhook signature bypass, price tampering, replay attacks, race conditions in checkout, refund state machine flaws.
- If Mobile Backend / API: Test JWT algorithm confusion (`alg: none`), token leakage in query strings, BOLA, receipt replay, excessive data exposure.

## STEP 2: Execute Audit
- **Mode A (G4):** Attack the test deployment / codebase against OWASP Top 10, ASVS 5.0, and API Top 10 vectors.
- **Mode B (G5):** In an isolated throwaway fork, inject rotating mutations to verify detection capabilities (100% detection rate mandatory).
- **Report Format:** Strict header, findings with reproducible PoCs (synthetic data), severity-capped score formula:
  - If open CRITICAL > 0 $\implies$ **Score = 0% (FAIL)**
  - If open HIGH > 0 $\implies$ **Score $\le$ 50% (Hard Cap, FAIL)**
  - If all CRITICAL and HIGH closed $\implies$ $\text{Score} = \frac{\text{Controls PASS}}{\text{Controls Applicable}} \times 100$
