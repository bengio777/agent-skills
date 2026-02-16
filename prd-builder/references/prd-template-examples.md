# PRD Template — Worked Examples
**Parent skill:** `prd-builder/SKILL.md`
**Companion to:** `prd-template.md`

This file contains worked examples for each PRD section. Read alongside
`prd-template.md` when generating a PRD.

---

## 1.1 Problem Statement Examples

**Good:** "Freelance developers waste 3-5 hours per week manually tracking
which clients owe them money, chasing invoices across email threads, and
reconciling payments in spreadsheets."

**Bad:** "People need better tools for managing their work."

---

## 1.3 Jobs-to-be-Done Example

- **Job:** Get paid faster for completed work
- **Current approach:** Create invoice in Google Docs, email as PDF, manually
  check bank account, send follow-up emails when overdue
- **Pain:** Takes 30+ minutes per invoice, easy to forget follow-ups, no
  visibility into what's outstanding
- **Success state:** Invoice sent in < 2 minutes, payments tracked
  automatically, overdue reminders sent without manual intervention

---

## 1.4 Persona Registry Examples

**Tier 1 — v1 Persona:**
> **Sarah, Freelance Designer**
> Sarah just wrapped a branding project and sent an invoice three weeks ago.
> She's 80% sure she hasn't been paid, but confirming means digging through
> Gmail, cross-referencing her spreadsheet, and checking her bank. Last month
> she forgot to follow up on a $3,000 invoice entirely.
>
> **Pain intensity:** Hair-on-fire — directly losing money
> **Architectural implications:** None beyond core MVP

**Tier 2 — v2-v3 Persona:**
> **Marcus, Agency Owner**
> Marcus runs a 12-person design agency with three PMs each handling their own
> billing. He has no centralized view of receivables and spends a full day each
> month aggregating spreadsheets.
>
> **Pain intensity:** High — operational inefficiency at scale
> **Architectural implications:** Multi-user workspaces and role-based
> permissions. Data model in v1 should use org-scoped records even if
> single-user at launch.

**Tier 3 — v4+ Persona:**
> **DevRaj, Integration Developer**
> DevRaj builds custom automations for accounting firms. He wants an API to
> programmatically create invoices and sync payment status.
>
> **Pain intensity:** Moderate — has workarounds
> **Architectural implications:** API-first patterns and webhook infrastructure
> should be considered in v2 architecture planning.

---

## 2.1 Feature Entry Example

> **F001 — Invoice Creation**
> **Job served:** Get paid faster (1.3)
> **Persona:** Sarah (Tier 1)
> **User Stories:**
> - *As Sarah, I want to create and send an invoice in under 2 minutes, so
>   that I stop procrastinating on billing.*
> - *As Sarah, I want invoices to auto-populate my business details, so that
>   I don't retype the same info every time.*
>
> **Description:** Create a professional invoice from a template, customize
> line items, send via email with a payment link.
>
> **Acceptance Criteria:**
> - Given a logged-in user, when they click "New Invoice," then a pre-filled
>   template loads with saved business details
> - Given a completed invoice, when the user clicks "Send," then the client
>   receives an email with PDF and payment link
> - Given any invoice, the creation-to-send flow completes in ≤ 5 clicks
>
> **Failure mode:** Users create invoices but never send them. Detected by
> tracking sent-to-created ratio; if < 70%, investigate UX friction.
>
> **Effort:** L — payment link integration adds complexity

---

## 2.3 Risk-Ranked Assumption Example

| Assumption | If Wrong | Validation Cost | Validate First? |
|---|---|---|---|
| Freelancers will pay $15/mo for invoicing | Revenue model fails | Low — landing page test | YES |
| Users prefer email delivery over client portal | Wrong delivery UX | Low — user interviews | YES |
| Stripe Connect works for our payment flow | Rearchitect payments | Medium — prototype | YES |
| Users want recurring invoices in v1 | Missed P1 feature | Low — survey | No (P1, not blocking) |

---

## 6.1 Version Roadmap Example

| Version | Theme | Personas | Key Capabilities | Scaling Stage |
|---------|-------|----------|-----------------|---------------|
| v1.0 | Core MVP | Sarah (T1) | F001-F005, basic auth, core workflow | 0–500 users |
| v2.0 | Retention | Sarah + power users, Marcus (T2) | F006-F010, integrations, analytics | 500–5K users |
| v3.0 | Growth | Marcus fully served, DevRaj (T3) emerging | Team features, API access | 5K–50K users |
| v4.0+ | Platform | All tiers + ecosystem | Marketplace, enterprise | 50K+ users |

---

## 9.3 Dependency Graph Example

```
F001 Invoice Creation
  └── F002 Email Delivery (blocked by F001)
       └── F004 Payment Reminders (blocked by F002)
F003 Payment Tracking
  └── F005 Dashboard (blocked by F001 + F003)
F001 + F003 → F005 (both must exist before dashboard makes sense)

CRITICAL PATH: F001 → F002 → F004 (longest chain = timeline driver)
```

---

## 9.2 Build Order Example

```
v1.0 — Phase 1: Foundation (Week 1-2)
  Day 1-2:  Data model + database setup (User, Invoice, Client entities)
  Day 3-4:  Auth (signup, login, session management)
  Day 5-7:  Core API routes (CRUD for invoices and clients)
  Day 8-10: Base UI shell (layout, navigation, routing)

v1.0 — Phase 2: Core Features (Week 3-5)
  Day 11-13: F001 Invoice Creation (form, template, preview)
  Day 14-15: F002 Email Delivery (send engine, email template)
  Day 16-18: F003 Payment Tracking (Stripe integration, webhook handler)
  Day 19-21: F005 Dashboard (status overview, outstanding amounts)
  Day 22-24: F004 Payment Reminders (scheduling, email triggers)

v1.0 — Phase 3: Polish & Launch (Week 6)
  Day 25-26: Error handling, edge cases, input validation
  Day 27-28: Mobile responsiveness, loading states, empty states
  Day 29-30: Deploy to production, monitoring setup, launch checklist
```

---

## Implementation Handoff Header Example

This goes at the top of every generated PRD:

```markdown
## How to Use This PRD

**For human developers:** Start with Section 9 (Timeline) for build order.
Reference Section 4 (Architecture) for stack decisions. Use Feature IDs
(F001, F002...) to trace any feature back to its job, persona, and
acceptance criteria.

**For AI coding agents (Claude Code, Cursor, etc.):** This PRD is structured
for direct consumption. Feed the full document as context. Start with the
data model (4.3), then implement features in the build order (9.2). Each
feature's acceptance criteria (2.1) serve as your definition of done.

**For stakeholders/investors:** Read Sections 1 and 6 for the vision and
roadmap. Section 7 covers go-to-market and pricing.
```
