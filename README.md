# Transpedite v2 — Architecture Overview

> Public architecture documentation for the Transpedite modernization project.
> Maintained by **Toucan Talent** for **Med-Rok**.

---

## What is Transpedite?

Transpedite is a clinical platform that coordinates patient transfers and discharges between hospitals and specialized care facilities. It manages:

- Clinical cases (patients needing transfer)
- Bed availability across discharge facilities
- Automatic matching between patients and beds based on clinical needs
- The full acceptance, transfer, and follow-up workflow
- HIPAA-compliant audit trail of all PHI access

The original system (Rails 5.0.2 / Ruby 2.4.1) is being **modernized in parallel** to a new HIPAA-compliant stack on AWS, with a controlled cutover at the end of the project.

---

## Documents

- **[Architecture diagram (Sprint 1)](./architecture-sprint-1.md)** — visual overview of the deployed infrastructure
- (More architecture snapshots will be added as the project evolves through Sprints 2-7)

---

## Project status

| | |
|---|---|
| **Current sprint** | Sprint 1 of 7 |
| **Stack target** | Rails 7.2 / Ruby 3.3 / PostgreSQL 16 / Redis 7 / AWS |
| **HIPAA compliance** | AWS BAA signed (Organization-level); PHI Gate of 14 controls before real patient data is loaded |
| **Maintained by** | Toucan Talent |

---

## Why this is public

This repository contains the **architecture overview** intended to be shared with:
- The client's stakeholders, board, partners
- External auditors (HIPAA, security)
- Toucan Talent portfolio and case studies

Sensitive operational details (AWS account IDs, IPs, secrets, code) live in a **private repository** maintained by Toucan Talent. Nothing in this public repo can be used to attack the live system.

---

## Contact

For questions, contact Toucan Talent through the established communication channels of the project.

---

> **Note**: This repository is read-only for external audiences. Architecture diagrams are updated by Toucan Talent at the end of each sprint.
