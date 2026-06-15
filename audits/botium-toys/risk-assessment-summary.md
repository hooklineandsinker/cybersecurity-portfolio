# Risk Assessment Summary — Botium Toys

**Auditor:** Kevin  
**Date:** June 2026  
**Source:** Botium Toys Scope, Goals, and Risk Assessment Report (Google Cybersecurity Certificate)

---

## Risk Score

**8 / 10** — Fairly High

This score reflects a lack of controls and poor adherence to compliance best practices across administrative, technical, and regulatory domains.

---

## Risk Description

Botium Toys currently has inadequate asset management and does not have all proper controls in place. The company may not be fully compliant with U.S. and international regulations and standards, particularly PCI DSS (payment card processing) and GDPR (EU customer data).

---

## Impact Assessment

| Category | Rating | Rationale |
|---|---|---|
| Asset Loss Impact | Medium | IT department is uncertain which assets are at risk — incomplete asset inventory |
| Regulatory / Fine Risk | High | Missing controls and non-adherence to PCI DSS and GDPR compliance practices |
| Business Continuity Risk | High | No DRP, no backups — single event could halt operations |
| Confidentiality Risk | High | No encryption; all employees have access to PII/SPII and cardholder data |

---

## Control Best Practices Gap

Per the NIST CSF **Identify** function, Botium Toys must:

- Dedicate resources to identify and appropriately manage all IT assets
- Classify existing assets and determine the business impact of their loss
- Implement controls to protect those assets aligned to their risk level

---

## Detailed Findings

| # | Finding | Risk Category | Severity |
|---|---|---|---|
| 1 | All employees have access to cardholder data and PII/SPII | Confidentiality / Access Control | 🔴 Critical |
| 2 | No encryption on credit card data at rest or in transit | Confidentiality / PCI DSS | 🔴 Critical |
| 3 | No least privilege or separation of duties | Access Control / Insider Threat | 🔴 Critical |
| 4 | No IDS installed | Detection / Incident Response | 🔴 Critical |
| 5 | No disaster recovery plan; no data backups | Business Continuity | 🔴 Critical |
| 6 | Password policy below minimum complexity standards | Authentication | 🟠 High |
| 7 | No centralized password management system | Authentication / Operations | 🟠 High |
| 8 | Legacy systems monitored without formal schedule | Vulnerability Management | 🟠 High |
| 9 | Firewall with defined ruleset in place | — | ✅ Adequate |
| 10 | Antivirus installed and monitored | — | ✅ Adequate |
| 11 | 72-hour EU breach notification plan exists | GDPR | ✅ Adequate |
| 12 | Physical security: locks, CCTV, fire detection in place | Physical | ✅ Adequate |

---

## NIST CSF Mapping

| NIST CSF Function | Status |
|---|---|
| **Identify** | ❌ Deficient — asset inventory incomplete; risk management ad hoc |
| **Protect** | ❌ Deficient — no encryption, no access controls, weak passwords |
| **Detect** | ❌ Deficient — no IDS; legacy monitoring unscheduled |
| **Respond** | ⚠️ Partial — GDPR notification plan exists; no formal IR plan documented |
| **Recover** | ❌ Deficient — no DRP, no backups |
