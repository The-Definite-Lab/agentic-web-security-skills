---
name: project-security-kickoff
description: >-
  Use when starting ANY new Web App, SaaS, E-commerce, Mobile, or API project from scratch.
  Detects project type, assigns Risk Tier, and begins G0 Scope intake.
  Trigger phrases: "new project", "start project", "naya project", "project shuru karna hai",
  "kickoff", "initialize security", "setup security for new project".
---

# PROJECT SECURITY KICKOFF (WEB APPLICATIONS & SAAS)

## STEP 1: Identify Project Type

Ask the user:
> "**Project type kya hai?**
> 1. SaaS Web Application (Next.js / React / Node / Python)
> 2. E-commerce Website / App (Stripe / Razorpay / Payments)
> 3. Mobile App Backend / API (FastAPI / Express / Go)
> 4. Firebase / Supabase Fullstack App
> 5. AI Chatbot / Agentic Integration / n8n Automation
> 6. Marketing / Portfolio / Landing Page (No PII)
> 7. Healthcare / Regulated Platform (HIPAA / DPDP)
> 8. Other (describe)"

---

## STEP 2: Assign Risk Tier & ASVS Level

| Project Type | Risk Tier | ASVS Target |
| :--- | :---: | :---: |
| Marketing / Portfolio / Static Blog | **LITE** | ASVS L1 (Lean) |
| SaaS, Habit Tracker, Small Client Portal | **STANDARD** | ASVS L2 (L1 non-sensitive) |
| E-commerce with Payments, Healthcare | **HIGH** | ASVS L2 + Selected L3 |
| Fintech, Banking, Government Citizen Data | **CRITICAL** | ASVS L3 |

---

## STEP 3: Generate G0 Scope Document (`scope.md`)

Generate a structured `scope.md` in the project root:
```markdown
# G0 SCOPE & RISK CLASSIFICATION

- **Project Name:** [Name]
- **Repository URL:** [URL]
- **Target Tier:** LITE | STANDARD | HIGH | CRITICAL
- **ASVS Level:** L1 | L2 | L3
- **Primary Tech Stack:** [e.g. Next.js 15, Tailwind, Supabase, Stripe]
- **Data Classifications:** Public | Internal | Confidential (PII) | Restricted (Financial/Auth)
- **Applicable Regulations:** GDPR, DPDP India, PCI DSS (if e-commerce)
- **Assigned Human Reviewer:** [Name / Handle]
- **Emergency Stop Contact:** [Email / Phone]
```

Commit `scope.md` with human signature before application code is written.
