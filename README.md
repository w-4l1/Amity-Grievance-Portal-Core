# Time-Bound Public Grievance Redressal Portal

A transparent, civic-first public grievance redressal platform engineered on ServiceNow PaaS (`x_1850267_public_0`) using the **ServiceNow Fluent SDK**. Citizens submit and track municipal complaints with instant tamper-evident **SHA-256 cryptographic receipts**, while municipal officers and escalation supervisors operate behind the scenes through an SLA-driven, role-segmented governance engine.

---

## Core Architecture & Engineering Highlights

| Layer | Implementation | Key Capabilities |
| :--- | :--- | :--- |
| **Data Model** | `x_1850267_public_0_grievance` (extends `task`) | Inherits enterprise task lifecycle, audit history, and assignment; adds `GRV` auto-numbering, `citizen_name`, `citizen_phone`, `grievance_category`, `target_resolution`, and read-only `cryptographic_hash`. |
| **Citizen Public Portal** | `/grievance` (`grievance_home`, OOTB Coral Theme) | Dual client/server 10-digit mobile validation, `GlideRecordSecure` intake, instant **Tamper-Evident Receipt** (Ticket #, SLA Deadline, 64-char SHA-256 digest), and server-backed **My Grievances** tracker. |
| **Agent Operations Workspace** | `/grievance?id=agent_workspace` | Real-time `GlideAggregate` SLA KPI tiles (Open vs. Breached %), urgency-sorted **Grievance Agent Queue**, category filters, inline **Manage / Resolve** modal, and event-driven live KPI auto-refresh (`$rootScope`). |
| **Cryptographic Integrity** | `GrievanceHashEngine` (`GlideDigest` SHA-256) | Generates a 64-character SHA-256 fingerprint (`citizen_name\|citizen_phone\|description`) on insert (`order: 200`), hard-locked via field-level ACL (`adminOverrides: false`), with an on-demand **Verify** action in the Agent Queue (`VERIFIED` / `TAMPER ALERT`). |
| **Departmental Auto-Routing** | `Route Grievance to Department` (`order: 150`) | Automatically routes incoming grievances by category to `Water Supply Dept`, `Electricity Board Dept`, `Sanitation & Waste Dept`, or `Roads & Infrastructure Dept` without overwriting manual assignments. |
| **SLA & Escalation Engine** | `Calculate SLA Deadline` (`order: 100`) + `Monitor Grievance SLA Breaches` | Assigns 48-hour (Water/Electricity) or 72-hour (Sanitation/Roads) deadlines; 15-minute scheduled monitor flags overdue tickets (`escalation = 3`), logs audit work notes, and fires `x_1850267_public_0.sla.breached` with a re-fire guard. |
| **Lifecycle Notifications** | 3 Active `sysevent_email_action` Rules | Automated **Grievance Received Confirmation** (with SHA-256 receipt), **Grievance Resolved Notification** (on deactivation), and high-importance **SLA Breach Escalation** to assigned agents and active `grievance_manager` users. |

---

## Role-Based Access Control (RBAC) Matrix

| Persona / Role | Table Create | Table Read | Table Write | Table Delete | `cryptographic_hash` Write | `agent_workspace` Access |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Authenticated Citizen** | Yes | Own tickets only (`opened_by`) | No | No | Denied | Blocked (403 + hidden in nav) |
| **`x_1850267_public_0.grievance_agent`** | Yes | All tickets | All tickets | No | Denied | Allowed |
| **`x_1850267_public_0.grievance_manager`** *(inherits `grievance_agent`)* | Yes | All tickets | All tickets | Yes | Denied | Allowed |
| **System Administrator** | Yes | All tickets | All tickets | Yes | **Denied (`adminOverrides: false`)** | Allowed |

---

## Release History (9 Atomic Pull Requests)

* **Phase 1 — Core Platform & Integrity**
  * **PR #1** `feat(data-model)`: Initialize `x_1850267_public_0_grievance` table and `GRV` auto-numbering.
  * **PR #2** `feat(portal)`: Deploy citizen-facing Public Grievance Service Portal (`/grievance`).
  * **PR #3** `feat(sla)`: Implement automated 48h/72h target resolution SLA calculation.
  * **PR #4** `feat(security)`: Add `GrievanceHashEngine` SHA-256 cryptographic hashing on insert.
* **Phase 2 — Agent Operations & Security**
  * **PR #5** `feat(workspace)`: Build Agent Workspace SLA dashboard on Service Portal framework.
  * **PR #6** `feat(rbac)`: Enforce scoped roles, table/field ACLs, hash immutability, and page security.
* **Phase 3 — Escalations, Portal UI & Governance**
  * **PR #7** `feat(notifications)`: Build custom breach event, 15-minute SLA monitor, and 3 email notifications.
  * **PR #8** `feat(portal-ui)`: Build Citizen Intake widget, role-aware navigation menu, and interactive Agent Queue.
  * **PR #9** `feat(governance-engine)`: Add 4 municipal assignment groups, departmental auto-routing, live SHA-256 integrity verifier, and live KPI auto-refresh.

---

## End-to-End Verification Summary

* **SLA Calculation & Breach Guard:** Verified 48h/72h deadlines across `GRV0001005`–`GRV0001007`; simulated a 2-hour overdue breach on `GRV0001006`, confirming `escalation = 3`, audit work note generation, single-fire event guard (`sysevent` count = 1 across 2 runs), and 5/5 `sys_email` records queued.
* **Tamper Resistance:** Verified `GlideRecordSecure.cryptographic_hash.canWrite() === false` as `admin` and confirmed `verifyIntegrity()` flips from `VERIFIED` to `TAMPER ALERT` upon payload mutation.
* **Departmental Routing:** Verified automatic assignment across all 4 civic categories (`Water`, `Electricity`, `Sanitation`, `Roads`).
