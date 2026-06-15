# Controls and Compliance Checklist — Botium Toys Internal Audit

**Auditor:** Kevin  
**Date:** June 2026  
**Framework:** NIST CSF | Regulations: PCI DSS, GDPR, SOC 1/2

---

## Controls Assessment Checklist

> Does Botium Toys currently have this control in place?

### Administrative / Managerial Controls

| Yes | No | Control | Control Type | Notes |
|---|---|---|---|---|
| | ✅ | Least Privilege | Preventative | All employees access all internal data — not implemented |
| | ✅ | Disaster Recovery Plans | Corrective | No DRP or backups exist |
| ⚠️ | | Password Policies | Preventative | Exists but below minimum complexity standards |
| | ✅ | Separation of Duties | Preventative | Not implemented |
| | ✅ | Access Control Policies | Preventative | No formal framework in place |
| | ✅ | Account Management Policies | Preventative | No lifecycle management |

### Technical Controls

| Yes | No | Control | Control Type | Notes |
|---|---|---|---|---|
| ✅ | | Firewall | Preventative | Active with defined ruleset |
| | ✅ | Intrusion Detection System (IDS) | Detective | Not installed |
| | ✅ | Backups | Corrective | No backups of critical data |
| ✅ | | Antivirus (AV) Software | Preventative | Installed and monitored regularly |
| ⚠️ | | Manual Monitoring / Maintenance for Legacy Systems | Preventative | Monitored but no formal schedule or clear methods |
| | ✅ | Encryption | Deterrent | Not in place for card data or PII |
| | ✅ | Password Management System | Preventative | No centralized system |

### Physical / Operational Controls

| Yes | No | Control | Control Type | Notes |
|---|---|---|---|---|
| ✅ | | Locks (offices, storefront, warehouse) | Deterrent/Preventative | Sufficient |
| ✅ | | Closed-Circuit Television (CCTV) Surveillance | Preventative/Detective | Up-to-date; all locations covered |
| ✅ | | Fire Detection / Prevention Systems | Detective/Preventative | Fire alarms and sprinklers in place |
| ✅ | | Adequate Lighting | Deterrent | In place |
| ❓ | | Locking Cabinets (network gear) | Preventative | Not confirmed in source materials |
| ❓ | | Time-Controlled Safe | Deterrent | Not confirmed in source materials |
| ❓ | | Signage — Alarm Service Provider | Deterrent | Not confirmed in source materials |

---

## Compliance Checklist

> Does Botium Toys currently adhere to this compliance best practice?

### Payment Card Industry Data Security Standard (PCI DSS)

| Yes | No | Best Practice |
|---|---|---|
| | ✅ | Only authorized users have access to customers' credit card information |
| | ✅ | Credit card data is stored, accepted, processed, and transmitted in a secure environment |
| | ✅ | Data encryption procedures are implemented to secure credit card transaction touchpoints and data |
| | ✅ | Secure password management policies are adopted |

### General Data Protection Regulation (GDPR)

| Yes | No | Best Practice |
|---|---|---|
| | ✅ | E.U. customers' data is kept private/secured |
| ✅ | | There is a plan in place to notify E.U. customers within 72 hours if their data is compromised/there is a breach |
| | ✅ | Data is properly classified and inventoried |
| ⚠️ | | Privacy policies, procedures, and processes are enforced to properly document and maintain data |

### System and Organizations Controls (SOC Type 1 / SOC Type 2)

| Yes | No | Best Practice |
|---|---|---|
| | ✅ | User access policies are established |
| | ✅ | Sensitive data (PII/SPII) is confidential/private |
| ✅ | | Data integrity ensures data is consistent, complete, accurate, and validated |
| | ✅ | Data is available only to individuals authorized to access it |

---

## Recommendations Summary

| Priority | Control / Practice | Reason |
|---|---|---|
| 🔴 Critical | Encryption | PCI DSS & GDPR violation; card and PII data exposed |
| 🔴 Critical | Least Privilege + Access Controls | Entire employee base has access to sensitive data |
| 🔴 Critical | IDS | No detection capability for active intrusions |
| 🔴 Critical | Disaster Recovery Plan + Backups | No recovery path after data loss event |
| 🟠 High | Password Management System | Policy exists but is unenforceable without tooling |
| 🟠 High | Separation of Duties | Required for PCI DSS and SOC compliance |
| 🟠 High | Legacy System Formal Schedule | Undefined intervention methods create vulnerability window |
| 🟡 Medium | Account Management Policies | Reduce stale account risk and attack surface |
| 🟡 Medium | Data Classification | Required for GDPR and SOC compliance |
