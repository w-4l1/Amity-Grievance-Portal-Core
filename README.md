# Public Grievance Redressal Portal (ServiceNow Scoped App)

A **Time-Bound, Cryptographically Tamper-Evident Public Grievance Redressal Portal** built as a scoped enterprise application (`x_1850267_public_0`) on **ServiceNow** using the Fluent SDK, App Engine Studio, and Service Portal.

## Key Capabilities

- **Cryptographic Tamper-Evidence (SHA-256):** `GrievanceHashEngine` computes an uppercase 64-character SHA-256 digest on every grievance insert, locks `cryptographic_hash` as strictly read-only via ACLs, issues an instant **Tamper-Evident Receipt** to the citizen, and enables one-click live integrity verification in the Agent Workspace.
- **Hardened Public Citizen Self-Onboarding:**
  - Custom public Sign In page (`/grievance` → `grievance_login`) with a direct **Create an Account** callout.
  - Public registration page (`/grievance?id=citizen_register`) protected by a **Stateless SHA-256 Math CAPTCHA** (`H(answer|nonce)`), a CSS-hidden **DOM honeypot** (`website_url`), authoritative server-side validation, and strict **zero-role** `sys_user` provisioning with `$s$` salted password hashing (`setDisplayValue()`).
- **Anti-Spam Rate Limiting:** Rolling 24-hour server-side `GlideAggregate` cap of **5 grievances per citizen** inside `Grievance Citizen Intake`.
- **RBAC & Privacy Isolation:**
  - `/grievance` (`grievance_home`) and `/grievance?id=agent_workspace` remain strictly private (`public: false`).
  - Citizens view only their own records (`opened_by = gs.getUserID()` via `GlideRecordSecure`).
  - `Agent Workspace` is gated to `x_1850267_public_0.grievance_agent` and `x_1850267_public_0.grievance_manager`.
- **Automated Routing & 48-Hour SLA Governance:**
  - Before-insert Business Rules stamp a **48-hour Target Resolution** (`Set Grievance Target Resolution`) and dynamically assign tickets to active municipal groups by name (`Route Grievance to Department`).
  - Scheduled SLA breach monitor (`Monitor SLA Breaches`) escalates overdue tickets and fires breach events.

## Architecture Overview

| Layer | Component | Identifier / Route |
| :--- | :--- | :--- |
| **Public Onboarding** | Custom Sign In & Self-Registration | `/grievance` (`grievance_login`), `/grievance?id=citizen_register` |
| **Citizen Portal** | Intake, SHA-256 Receipt & Tracker | `/grievance?id=grievance_home` (`x_1850267_public_0_citizen_intake`) |
| **Agent Workspace** | Queue, SLA Badges & Hash Verifier | `/grievance?id=agent_workspace` |
| **Core Data Model** | Grievance Table (extends `task`) | `x_1850267_public_0_grievance` |
| **Security Engine** | SHA-256 Hashing & CAPTCHA Signing | `GrievanceHashEngine` (`sys_script_include`) |

## Operational Prerequisite (Fresh Instance Setup)

Because `sys_user_group` is a non-scoped global table, the four municipal assignment groups are managed as **operational foundation data** rather than scoped XML metadata. On a fresh ServiceNow instance, create and activate these four exact group names:

1. `Water Supply Dept` (`grievance_category = water`)
2. `Electricity Board Dept` (`grievance_category = electricity`)
3. `Sanitation & Waste Dept` (`grievance_category = sanitation`)
4. `Roads & Infrastructure Dept` (`grievance_category = roads`)

## Verification & UAT Summary

- **Automated Security Suite (`GlideScopedEvaluator`):** **10/10 PASS** (CAPTCHA generation, wrong-answer rejection, silent honeypot trap, token tampering, field validation, password mismatch, zero-role salted provisioning, duplicate blocking, 24h rate limit, and anonymous HTTP boundary checks).
- **End-to-End UAT Matrix (`TC-01` – `TC-10`):** **10/10 PASS** after remediating 4 Cycle-1 defects in PR #11 (zero-role `GlideRecordSecure` insert write-drop, portal homepage redirect, 48-hour SLA rule, and `close_notes` form layout).

## Release History (Pull Requests)

- **PR #1 – PR #4:** Scoped app foundation, `x_1850267_public_0_grievance` schema, `GrievanceHashEngine` (SHA-256), RBAC roles, and hash-immutability ACLs.
- **PR #5 – PR #9:** Citizen Portal, Agent Workspace with live hash verification, dynamic department routing, and SLA breach monitoring.
- **PR #10:** Public Citizen Self-Registration, stateless SHA-256 Math CAPTCHA, CSS honeypot, custom `grievance_login` page, and 24-hour rolling rate limit.
- **PR #11:** UAT defect remediation (`GlideRecord` validated insert, `grievance_home` post-login redirect, 48-hour `Set Grievance Target Resolution` Business Rule, and `close_notes` form layout).
