# Transpedite Modernization — Detailed Work Plan

> Public planning document for the modernization of the Transpedite platform.
> Maintained by **Toucan Talent** for **Med-Rok**.
> Last updated: 2026-05-18

---

## ⏱ Project duration

**Total project: approximately 2 months from project kickoff to milestone M3** (platform ready, PHI Gate certified).

- ✅ **Sprint 1 — Foundation (1 week)**: complete
- **Sprint 2 onward — this plan (8 weeks remaining)**:
  - Weeks 1–4: Application core rebuilt (M1)
  - Weeks 5–6: Demo environment ready, client UAT (M2)
  - Weeks 7–8: HIPAA controls verified, pentest closed, PHI Gate certified (M3)

See the [Milestones](#milestones), [Gantt chart](#gantt-chart) and [Week-by-week plan](#week-by-week-plan) sections below for the full breakdown.

---

## Purpose of this document

This plan covers all work from **Sprint 2 through delivery of a HIPAA-ready modernized platform with a live Demo environment**.

**Sprint 1 (foundation) is complete** — see [architecture-sprint-1.md](./architecture-sprint-1.md). 51 of 67 Sprint 1 tasks completed (76%); the foundational AWS infrastructure is operational; the AWS BAA is signed; the technical platform is ready to receive the modernized application.

---

## Executive summary

| | |
|---|---|
| **Goal** | Modernize the Transpedite platform on a HIPAA-ready stack, with a Demo environment available for Med-Rok to present the platform to potential  clients. |
| **Approach** | **Parallel rebuild**: the existing demo system stays available; the new platform is built next to it; the modernized platform becomes the Demo at the end of the project. |
| **Team** | 2 senior developers + AI-assisted development workflow. |
| **Total duration** | **~2 months from kickoff to platform ready**. Sprint 1 (1 week) already complete; this plan covers the remaining 8 weeks. |
| **What's included** | Two environments — Development (internal) and Demo (client-facing). HIPAA controls verified and documented (PHI Gate certified ready). |
| **Already delivered** | Sprint 1: HIPAA-ready AWS infrastructure, signed BAA, base application stack, project board, full documentation suite. |

---

## Scope

This contract delivers the **Transpedite platform** — the technology that future clients will use, plus a permanent **Demo environment** for sales presentations.

**Included**:
- The application rebuilt on a modern, supported stack (Rails 7.2, Ruby 3.3, PostgreSQL 16).
- All HIPAA security controls built into the code (encryption, audit logging, TLS, access controls).
- The PHI Gate certified ready — all 14 controls verified.
- A Demo environment, accessible via URL, where Med-Rok can present the platform to potential clients.



---

## Environments

| Environment | Purpose | Data | Included in this contract |
|---|---|---|---|
| **Development** | Internal — for the development team to build and test. Runs locally. | Synthetic, generated automatically | ✅ Yes |
| **Demo** | Client uses this to present the platform to potential customers via URL. | Synthetic, curated to look realistic | ✅ Yes |


---

## Milestones

| # | Milestone | Week | What the client receives |
|---|---|---|---|
| **M1** | Application core rebuilt | End of Week 4 | All 34 domain models reconstructed on Rails 7.2; state machines operational; PHI encryption configured; access audit table live. Available for technical inspection. |
| **M2** | Demo environment ready for client review | End of Week 6 | Full application running in the Demo environment with curated synthetic data. Client and sales team can use it to present to prospects. |
| **M3** | **PHI Gate certified — platform ready** | End of Week 8 | All 14 HIPAA controls verified and documented. External pentest passed. The platform is technically ready for HIPAA-regulated use. |

---

## Gantt chart

```mermaid
gantt
    title Transpedite Modernization — Sprint 2 to Platform Ready
    dateFormat YYYY-MM-DD
    axisFormat W%V

    section Sprint 1 (done)
    Infrastructure + BAA + base stack        :done, s1, 2026-05-11, 1w

    section Application build
    Models + encryption + audit log          :a1, 2026-05-25, 1w
    State machines + Pundit policies         :a2, after a1, 1w
    Search + matching + background jobs      :a3, after a2, 1w
    New modules + integration (M1)           :crit, a4, after a3, 1w
    UI complete + Demo environment           :a5, after a4, 1w
    Client UAT in Demo (M2)                  :a6, after a5, 1w

    section Migration tooling (reusable)
    Migration framework design               :b1, 2026-06-08, 2w
    Migration scripts per domain             :b2, after b1, 2w

    section HIPAA Gate
    Pentest contracted + backups built       :crit, c1, 2026-05-25, 1w
    Pentest execution (external vendor)      :c2, after c1, 3w
    Pentest remediation + 12 controls        :c3, after c2, 2w
    PHI Gate certified (M3)                  :milestone, c4, 2026-07-19, 0d

    section Demo finalization
    Demo polish + presentable data           :d1, 2026-07-13, 1w
    Platform ready (M3)                      :milestone, d2, 2026-07-19, 0d
```

*Dates assume kickoff on 2026-05-25 (the Monday following plan approval). All dates shift by the same amount if approval is delayed.*

---

## Week-by-week plan

### Week 1 — Kickoff
**Build:** Application skeleton on Rails 7.2 — model classes, migrations for the 27 base tables, Active Record Encryption keys provisioned in AWS, PHI access audit table created.
**Migration tooling:** Initial design for the migration script framework.
**HIPAA:** External pentest vendor contracted and scheduled; automated backup infrastructure built.
**Deliverable:** Schema deployed in the Demo environment.

### Week 2 — State machines and authorization
**Build:** Transfer workflow state machines (Statesman), history tables, Pundit authorization policies for every controller, security questions flow.
**Migration tooling:** First migration script templates (users, hospitals, facilities).
**HIPAA:** Backup restore drill executed and documented (control #9).
**Deliverable:** Core workflow operational with synthetic data; user authentication and authorization complete.

### Week 3 — Search, matching, background jobs
**Build:** Patient-bed matching algorithm; PostgreSQL full-text search (replaces legacy Solr); background job system for SLA tracking and notifications (replaces legacy cron battery).
**Migration tooling:** Templates for cases, beds, and matches.
**HIPAA:** Pentest executes (external vendor, in parallel with development).
**Deliverable:** Matching engine functional; SLA timers active; search across cases and beds.

### Week 4 — New modules and integration ← **M1: Core rebuilt**
**Build:** Discharge Barrier checklist module; Inpatient Psychiatric profile module; integration tests covering the full transfer workflow end-to-end.
**Migration tooling:** All 8 migration script templates complete and tested with synthetic data.
**HIPAA:** Pentest continues; preliminary findings reviewed if available.
**Deliverable:** All functional modules built and integrated. Client can review code and architecture.

### Week 5 — UI rebuild and Demo environment
**Build:** Complete UI rebuild on the modern stack; PDF generation for face sheets; attachments uploading to encrypted S3 (in the Demo environment).
**Migration tooling:** Migration framework documented for future use.
**HIPAA:** Pentest report received; remediation work begins.
**Deliverable:** Demo environment fully operational with synthetic data.

### Week 6 — Client UAT in Demo ← **M2: Demo ready for client review**
**Build:** Bug fixes from client UAT; performance tuning; load test execution.
**HIPAA:** Remaining controls (#4, #6, #7, #10, #12) verified and documented.
**Client action:** UAT testing of the application against business workflows.
**Deliverable:** Functional sign-off from the client.

### Week 7 — HIPAA controls and Demo polish
**Build:** Final bug fixes; Demo data curated for sales presentations (realistic but fully synthetic patient names, hospitals, cases).
**HIPAA:** Controls #5, #8, #11 verified; incident response policy signed off; pentest remediation complete.
**Deliverable:** Demo data presentable; HIPAA controls on track.

### Week 8 — PHI Gate certification ← **M3: Platform ready**
**Build:** Final Demo polish.
**HIPAA:** All 14 controls verified, individually documented, and signed off (control #14 included).
**Deliverable:** PHI Gate certificate. The platform is technically ready for HIPAA-regulated use.

---

## What we need from the client

These items determine whether the timeline holds:

| # | What we need | When | If missed |
|---|---|---|---|
| 1 | Confirmation of scope for the two new modules (Discharge Barrier, Inpatient Psych) | **By end of Week 1** | Modules removed from scope; client can add them in a later sprint. |
| 2 | Client availability for UAT testing in the Demo environment | **Week 6** | UAT shifts later; remaining weeks compress. |
| 3 | Decision on whether the pentest vendor is engaged through this contract or directly by the client | **By end of Week 1** | If through this contract: we book the vendor. If direct: client books it. Either way, pentest must start by end of Week 1 to finish on time. |

---

## PHI Gate — the 14 controls

Although no real PHI is loaded during this contract, the PHI Gate is certified ready so that no security blockers remain. All 14 controls signed off by Week 8:

| # | Control | When verified |
|---|---|---|
| 1 | AWS BAA signed | ✅ Done (Sprint 1) |
| 2 | EBS encryption with KMS CMK | ✅ Done (Sprint 1) |
| 3 | S3 encryption with KMS CMK | ✅ Done (Sprint 1) |
| 4 | Active Record Encryption operational | Week 6 |
| 5 | End-to-end TLS verified (SSL Labs A/A+) | Week 7 |
| 6 | PHI access audit log operational | Week 6 |
| 7 | Authorization policies reviewed (Pundit) | Week 6 |
| 8 | MFA active for all admins | Week 7 |
| 9 | Backup restore drill documented | Week 2 |
| 10 | PHI filters in logs verified | Week 6 |
| 11 | Incident response policy signed | Week 7 |
| 12 | Disclaimers operational | Week 6 |
| 13 | External pentest passed | Week 7 |
| 14 | Synthetic data isolation procedure documented | Week 8 |

---

## Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Pentest finds critical issues | Could delay HIPAA Gate by 1–2 weeks | Pentest scheduled to finish by Week 7, leaving buffer in Week 8. |
| Legacy domain logic reveals unknown complexity | Build phase extends 1–2 weeks | Built-in buffer in Week 6 (UAT) where bug fixes are scoped. |
| Scope creep on new modules | Build phase extends | Scope frozen at end of Week 1. Changes route to a follow-on engagement. |
| Pentest vendor lead time longer than 1 week | Pentest finishes later | Contract signed Week 1; alternate vendors identified as backup. |

---

## Approach: why "parallel rebuild" and not "lift and shift"

The current Transpedite demo runs on technology components that have not received security patches for years — some for more than five years. A system that handles patient data on unpatched technology **cannot be HIPAA-compliant**, regardless of the surrounding infrastructure. This is not a matter of opinion; it is a regulatory baseline.

For that reason, the project rebuilds the application on a modern, supported stack rather than simply moving the legacy code onto the new AWS infrastructure. The current demo keeps operating throughout the project, without interruption, until the new platform is ready to replace it.


---

## Acceptance criteria for sign-off

This plan is considered approved when:

- The **3 milestones (M1–M3)** are accepted as the project's measurable outcomes.
- The **3 client commitments** are accepted.
- The **8-week timeline** is approved.

---

## Related documents

- [Architecture diagram (Sprint 1)](./architecture-sprint-1.md) — what was delivered.
- Sprint 1 work and detailed ticket history is maintained by Toucan Talent in the project's ClickUp workspace; the client has access for visibility and comments.

---

> This is a working document. The plan will be updated weekly during execution. Any change to scope, timeline, or milestones requires written agreement from both parties.
