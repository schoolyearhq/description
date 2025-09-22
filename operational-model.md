# Partnering with Existing NGOs & Proof-of-Impact Incentives

This document describes how Schoolyear relies on **existing, credible NGOs** to onboard schools, and how the **incentive to upload proof** (anonymised enrollment/completion certificates) is created by tying **future disbursements** to **verified proof from the previous academic year**.

> **Principle:** Plug into reality, not rebuild it. Use trusted local operators to validate schools and students; release funds only against verifiable proof.

---

## Why Existing NGOs?

- **Local legitimacy:** Established NGOs already work with government-recognised schools and understand fee structures, calendars, and risks.
- **Operational reach:** They have staff/partners on the ground to collect and verify documents without heavy new structures.
- **Speed & cost:** Onboarding via known actors avoids duplicating monitoring systems and preserves the 98% efficiency target.

---

## Roles at a Glance

- **Schoolyear (platform/infrastructure):**
  - Defines verification standards and data formats (certificates).
  - Runs the pooled fund, treasury buffer policy, and risk/expansion rules.
  - Publishes anonymised proof and allocation reports.

- **Partner NGO (local operator):**
  - Identifies government-recognised schools; validates fees and term dates.
  - Collects **anonymised** enrollment/completion certificates from schools.
  - Submits proof packs according to the schema & timeline.
  - Flags anomalies and supports occasional spot checks.

- **School (beneficiary institution):**
  - Receives fee payments for sponsored students.
  - Issues official enrollment/completion certificates (or attendance registers stamped by admin).
  - Cooperates with audits and data minimisation requirements.

---

## Onboarding Workflow (via NGOs)

```mermaid
flowchart LR
    A[NGO proposes school] --> B[Eligibility check\n(govt recognition, fee schedule)]
    B -->|Pass| C[MoU & data-protection addendum]
    B -->|Fail| X[Reject / remediate]
    C --> D[Create school profile in registry\n(fee=€200/unit or local equivalent)]
    D --> E[Provisional allocation set\n(based on fund/buffer rules)]
    E --> F[Initial disbursement at academic year start]
    F --> G[School teaches; NGO collects proof]
    G --> H[Upload proof pack\n(anonymised certificates)]
    H --> I[Automated checks + random audits]
    I -->|Pass| J[Next year unlocked]
    I -->|Fail/Delay| Y[Hold/reduce allocation\n+tickets for remediation]
