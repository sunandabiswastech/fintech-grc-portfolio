# NovaPay Financial Technologies — GRC Portfolio Project
### Cloud-Native FinTech Payment Application Security Program
**Document Classification:** Portfolio / Educational Use
**Prepared By: Sunanda Biswas, GRC Analyst Candidate
**Date:** June 2026
**Version:** 1.0

---

## Table of Contents

1. [Fictional Company Profile](#1-fictional-company-profile)
2. [Regulatory & Framework Scope](#2-regulatory--framework-scope)
3. [Risk Register](#3-risk-register)
4. [Vendor Security Assessment](#4-vendor-security-assessment)
5. [Incident Response Plan](#5-incident-response-plan)

---

## 1. Fictional Company Profile

### NovaPay Financial Technologies, Inc.

**Headquarters:** Wilmington, Delaware (Remote-first operations)
**Founded:** 2022
**Employees:** 85 (Engineering: 40, Operations: 20, Compliance/Legal: 10, Sales/Product: 15)
**Stage:** Series A — $18M raised
**Annual Transaction Volume (Projected):** $2.4B
**Registered Money Services Business (MSB):** FinCEN Registration #31000295847

---

### Business Overview

NovaPay is a cloud-native Neo-Bank and embedded finance platform designed for Millennial and Gen Z consumers. NovaPay offers the following financial products via its mobile application (iOS/Android) and developer API:

- **NovaPay Debit Card:** Visa-network debit card linked to FDIC-insured deposit accounts (custodied via partner bank, Coastal Community Bank)
- **NovaPay Send:** Peer-to-peer (P2P) money transfers between NovaPay users and external bank accounts via ACH
- **NovaPay Checkout:** A merchant-facing payment API enabling businesses to accept card-not-present (CNP) transactions
- **NovaPay Wallet:** A multi-currency digital wallet supporting USD and select stablecoins

### Technology Stack

| Layer | Technology |
|---|---|
| Cloud Provider | Amazon Web Services (AWS) — Primary; GCP — DR |
| Core Application | Node.js microservices on Amazon EKS (Kubernetes) |
| Database | Amazon Aurora PostgreSQL (encrypted at rest, AES-256) |
| API Gateway | AWS API Gateway + Kong Enterprise |
| Card Processing | Marqeta (issuer processor) |
| KYC / Identity | Persona (third-party KYC vendor) |
| Fraud Detection | Sardine AI |
| Secrets Management | HashiCorp Vault (HCP) |
| SIEM | Datadog + AWS Security Hub |
| IaC | Terraform (AWS), Helm (K8s) |

### Regulatory Posture

NovaPay processes, transmits, and stores payment card data on behalf of cardholders and merchants. As a result, NovaPay is subject to the following regulatory and compliance frameworks:

- **PCI-DSS v4.0** (Service Provider, Level 1 — >6M transactions/year projected)
- **SOC 2 Type II** (Security and Confidentiality Trust Services Criteria)
- **Bank Secrecy Act (BSA) / Anti-Money Laundering (AML)**
- **FinCEN MSB Requirements**
- **State Money Transmitter Licenses** (currently licensed in 32 states)
- **Consumer Financial Protection Bureau (CFPB) — Regulation E** (Electronic Funds Transfer Act)

---

## 2. Regulatory & Framework Scope

### 2.1 PCI-DSS v4.0 — Applicable Requirements

NovaPay operates as both a **merchant** (for NovaPay Checkout) and a **service provider** (storing, processing, and transmitting cardholder data on behalf of merchants). As a Level 1 Service Provider, NovaPay must undergo an annual **Report on Compliance (ROC)** conducted by a Qualified Security Assessor (QSA), plus quarterly network scans by an Approved Scanning Vendor (ASV).

The following PCI-DSS v4.0 requirements are directly applicable to NovaPay's environment:

#### Cardholder Data Environment (CDE) Scope

NovaPay's CDE encompasses:
- Amazon EKS clusters hosting the payment processing microservices
- Amazon Aurora PostgreSQL databases storing tokenized PANs and encrypted CVVs (at authorization time only)
- AWS API Gateway endpoints receiving card transaction data
- Marqeta API integration handling raw PAN transmission
- HashiCorp Vault storing encryption keys for cardholder data

#### Key PCI-DSS v4.0 Requirements in Scope

| Req. # | Requirement Title | NovaPay Applicability |
|---|---|---|
| 1.1–1.3 | Install and Maintain Network Security Controls | AWS Security Groups, NACLs, and WAF rules isolating the CDE |
| 2.1–2.3 | Apply Secure Configurations | Hardened EKS node configurations; CIS Benchmark compliance enforced via AWS Config |
| 3.3–3.5 | Protect Stored Account Data | Tokenization via Marqeta (no raw PANs stored); AES-256 encryption for any stored data elements |
| 4.2 | Protect Cardholder Data with Strong Cryptography During Transmission | TLS 1.2+ enforced on all external APIs; certificate pinning on mobile apps |
| 5.2–5.3 | Protect All Systems Against Malware | AWS GuardDuty, CrowdStrike Falcon on EKS nodes |
| 6.2–6.4 | Develop and Maintain Secure Systems and Software | SDLC policy, SAST/DAST in CI/CD pipeline, dependency scanning via Snyk |
| 6.4.1 | **NEW in v4.0** — Web-facing applications protected by WAF | AWS WAF with OWASP Core Rule Set deployed in front of NovaPay Checkout API |
| 7.1–7.3 | Restrict Access by Business Need | Role-Based Access Control (RBAC) on all AWS IAM, Kubernetes, and database resources |
| 8.2–8.6 | Identify and Authenticate Access to System Components | MFA required for all CDE access; privileged access via break-glass procedures with session recording |
| 10.2–10.7 | Log and Monitor All Access to System Components and Cardholder Data | Datadog SIEM, CloudTrail, and VPC Flow Logs with 12-month retention |
| 11.3 | External and Internal Penetration Testing | Annual external pentest by QSA-approved firm; quarterly internal scans |
| 11.4.7 | **NEW in v4.0** — Multi-tenant service providers must support customers' pen test requests | NovaPay must provide segmentation confirmation reports to merchant customers |
| 12.3.2 | **NEW in v4.0** — Targeted Risk Analysis for each requirement | Annual Targeted Risk Analysis (TRA) documented and approved by CISO |
| 12.6 | Security Awareness Education | Annual security training + phishing simulation; tracked completion rates |

#### PCI-DSS v4.0 Notable Changes Affecting NovaPay

- **Requirement 6.4.1:** All payment pages served from NovaPay's domain must now use a Content Security Policy (CSP) and script integrity mechanisms to protect against e-skimming/Magecart attacks. NovaPay's frontend team must inventory all third-party JavaScript loaded on checkout flows.
- **Requirement 8.4.2:** MFA is now required for ALL access into the CDE, not just remote access — this includes internal developer access to production Kubernetes namespaces.
- **Customized Approach:** PCI-DSS v4.0 introduces an alternative to prescriptive controls, allowing NovaPay to define equivalent controls with documented rationale validated by a QSA.

---

### 2.2 SOC 2 Type II — Trust Services Criteria (TSC)

NovaPay has committed to a **SOC 2 Type II** examination covering the **Security (CC)** and **Confidentiality (C)** Trust Services Criteria. The examination period is 12 months, audited by an independent CPA firm (Sauer & Associates LLP). This report will be shared with enterprise merchant customers under NDA.

#### In-Scope Trust Services Criteria

**Security (Common Criteria — CC Series)**

| TSC Reference | Criteria | NovaPay Implementation |
|---|---|---|
| CC1.1–CC1.5 | Control Environment | Organizational structure, Board oversight, Code of Conduct, CISO role defined |
| CC2.1–CC2.3 | Communication & Information | Internal policy wiki, security awareness communications, external breach notification policy |
| CC3.1–CC3.4 | Risk Assessment | Annual enterprise risk assessment; threat modeling on all new product features |
| CC4.1–CC4.2 | Monitoring of Controls | Continuous control monitoring via Drata (GRC platform); quarterly control testing |
| CC5.1–CC5.3 | Control Activities | Control design documented; separation of duties enforced; IaC reviews required before deployment |
| CC6.1 | Logical & Physical Access | Zero-trust architecture; VPN-less access via Tailscale; SSO via Okta with SAML 2.0 |
| CC6.2 | Prior to Provisioning Access | Access request workflow in Jira Service Desk; manager approval required |
| CC6.3 | Role-Based Access / Least Privilege | IAM roles follow least privilege; quarterly access reviews via Vanta |
| CC6.6 | Security of External Transmissions | TLS 1.2+ on all external endpoints; API key rotation enforced via HashiCorp Vault |
| CC6.7 | Transmission and Removal of Information | DLP policy; encrypted data transfer for all vendor integrations; data destruction SOP |
| CC6.8 | Malicious Software Prevention | CrowdStrike EDR; runtime container security via Falco; image signing via Cosign |
| CC7.1 | Detection of Anomalies | Datadog anomaly detection; AWS GuardDuty; 24/7 SOC alerting |
| CC7.2 | Monitoring of System Components | Infrastructure health monitoring; uptime SLAs tracked; synthetic monitoring on payment APIs |
| CC7.3 | Evaluation of Security Events | Incident triage runbooks; SIEM correlation rules for payment fraud indicators |
| CC7.4–CC7.5 | Incident Response | NIST SP 800-61 aligned IRP (see Section 5); tabletop exercises bi-annually |
| CC8.1 | Change Management | All changes require PR review + JIRA ticket; production deployments gated by CI/CD pipeline |
| CC9.1 | Risk Mitigation | Cyber insurance policy ($10M coverage); business continuity plan reviewed annually |
| CC9.2 | Vendor Risk Management | Annual third-party risk assessments (see Section 4); SOC 2 reports required from Tier 1 vendors |

**Confidentiality (C Series)**

| TSC Reference | Criteria | NovaPay Implementation |
|---|---|---|
| C1.1 | Identification of Confidential Information | Data Classification Policy — four tiers: Public, Internal, Confidential, Restricted |
| C1.2 | Disposal of Confidential Information | Data retention schedule; secure deletion via AWS S3 Object Lock; database purge procedures |

---

## 3. Risk Register

**Methodology:** Risk scores use a 5×5 likelihood × impact matrix.
**Scoring Scale:** 1 (Very Low) to 5 (Very High)
**Inherent Risk Score = Likelihood × Impact (Max: 25)**
**Residual Risk Score = Post-mitigation reassessment**
**Risk Appetite Threshold:** Residual scores ≥ 12 require executive sign-off and a formal remediation plan.

---

### Risk Register Table

| Risk ID | Threat Description | Category | Likelihood (1–5) | Impact (1–5) | Inherent Risk Score | PCI-DSS v4.0 Mapping | SOC 2 TSC Mapping | Mitigation Strategy | Residual Risk Score | Risk Owner |
|---|---|---|---|---|---|---|---|---|---|---|
| **RISK-001** | **API Authorization Bypass / Broken Object-Level Authorization (BOLA):** An attacker exploits missing or misconfigured authorization checks in NovaPay's P2P Transfer API, allowing them to initiate fund transfers from another user's account by manipulating the `account_id` parameter in API requests (OWASP API Security Top 10 — API1:2023). A successful exploit could result in unauthorized fund movement, regulatory sanction, and customer trust destruction. | Application Security | 4 | 5 | **20 (Critical)** | Req. 6.2.4 (Secure coding against broken access control); Req. 6.3.2 (Bespoke/custom software inventory); Req. 11.3.1 (Penetration testing — external) | CC6.1 (Logical Access Controls); CC7.1 (Anomaly Detection); CC6.8 (Malicious Software Prevention) | (1) Enforce server-side object-level authorization on every API endpoint — never trust client-supplied account identifiers. (2) Implement API gateway-level JWT validation confirming sub claim matches requested resource owner. (3) Include BOLA-specific test cases in DAST pipeline using OWASP ZAP and StackHawk. (4) Annual external penetration test with mandatory OWASP API Top 10 coverage. (5) Anomaly detection rule: flag any account accessing >10 unique account resources within 60 seconds. | **8 (Medium)** | VP Engineering |
| **RISK-002** | **Third-Party KYC Vendor Data Breach (Persona):** NovaPay transmits Personally Identifiable Information (PII) — including SSNs, government ID images, and selfie biometrics — to its KYC vendor Persona for identity verification. A breach at Persona's infrastructure, or a misconfigured API exposing NovaPay customer records, could result in mass PII exposure affecting all onboarded NovaPay customers. Regulatory exposure includes CFPB enforcement, state AG action, and potential class-action liability. | Third-Party / Supply Chain Risk | 3 | 5 | **15 (High)** | Req. 12.8.2 (Written agreements with third-party service providers acknowledging PCI responsibility); Req. 12.8.4 (Monitor TPSP compliance status annually) | CC9.2 (Vendor Risk Management); C1.1 (Confidential Information Identification); CC6.7 (Data Transmission Security) | (1) Execute Data Processing Agreement (DPA) and BAA equivalent with Persona, specifying breach notification SLA of ≤72 hours. (2) Require Persona's current SOC 2 Type II report annually; review findings before contract renewal. (3) Minimize data transmitted — send only data elements required for the specific KYC check; do not transmit full SSN where partial suffices. (4) Implement field-level encryption on all PII transmitted to Persona API. (5) Contractual right-to-audit clause enabling NovaPay to conduct security assessments of Persona. (6) Vendor offboarding procedure ensuring data deletion within 30 days of contract termination with documented certification. | **9 (Medium)** | Chief Compliance Officer |
| **RISK-003** | **Cloud Misconfiguration Exposing Cardholder Data (S3 / Aurora Public Access):** An engineer inadvertently configures an Amazon S3 bucket containing transaction logs or an Aurora PostgreSQL snapshot as publicly accessible during a production incident response or infrastructure refactoring exercise. Given that log data may contain partial PANs or transaction metadata, public exposure would constitute a PCI-DSS reportable breach and require mandatory card brand notification. AWS misconfigurations remain the leading cause of cloud data breaches industry-wide. | Cloud Infrastructure Risk | 3 | 5 | **15 (High)** | Req. 1.3.2 (Restrict inbound/outbound traffic to only necessary communications); Req. 2.2.1 (System configuration standards); Req. 10.3.3 (Audit log protection from modification/destruction) | CC6.1 (Logical Access); CC7.1 (Anomaly/Configuration Drift Detection); CC8.1 (Change Management) | (1) AWS Organizations Service Control Policies (SCPs) blocking `s3:PutBucketAcl` with `PublicRead` for all member accounts in the CDE organizational unit — enforced at the control plane level. (2) AWS Config Rules: `s3-bucket-public-read-prohibited` and `rds-instance-public-access-check` with auto-remediation Lambdas. (3) All S3 buckets deploy via Terraform with `block_public_acls = true` enforced in the module — manual console changes trigger a CloudTrail alert and PagerDuty page. (4) Quarterly infrastructure security reviews using Prowler and AWS Security Hub CIS Benchmark score ≥ 90%. (5) Aurora snapshots encrypted with AWS KMS CMK; snapshot sharing restricted via resource-based policy requiring explicit allowlist of AWS account IDs. | **4 (Low)** | Head of Cloud Infrastructure |
| **RISK-004** | **Credential Stuffing / Account Takeover (ATO) on Consumer Mobile App:** Threat actors use large-scale automated tooling to test username/password combinations harvested from unrelated third-party breaches against NovaPay's authentication endpoint. A successful ATO enables unauthorized fund transfers, fraudulent P2P sends, and debit card provisioning to attacker-controlled Apple Pay / Google Pay wallets. At NovaPay's scale, even a 0.1% ATO success rate could compromise thousands of accounts, generating regulatory scrutiny under Regulation E and significant fraud loss reimbursement obligations. | Authentication & Identity Risk | 4 | 4 | **16 (High)** | Req. 8.3.6 (Passwords must meet minimum complexity requirements); Req. 8.4.2 (MFA required for all access into CDE); Req. 8.6.1 (System/application accounts managed by policies) | CC6.1 (Logical Access Controls); CC6.2 (Prior-to-Provisioning Access); CC7.1 (Anomaly Detection) | (1) Enforce progressive rate limiting on `/auth/login` endpoint: 5 failed attempts triggers 30-minute lockout with exponential backoff thereafter; lockout events generate SIEM alert. (2) Integrate Cloudflare Turnstile (invisible CAPTCHA) on all authentication flows to break automated tooling. (3) Implement device fingerprinting (Sardine AI) — logins from new device + new IP + high-risk geo trigger step-up authentication via SMS/email OTP before session creation. (4) Push customers toward passwordless authentication (biometric/passkey via WebAuthn) to eliminate password-based attack surface. (5) Real-time credential breach monitoring using HaveIBeenPwned Enterprise API — force password reset when customer credential detected in new breach corpus. (6) Monitor for velocity anomalies: >500 failed login attempts per minute on any single IP → automatic block + SOC alert. | **6 (Low)** | Director of Security Engineering |
| **RISK-005** | **Insider Threat — Privileged Database Access Abuse:** A NovaPay database administrator or DevOps engineer with direct access to production Aurora PostgreSQL exfiltrates customer transaction records, account balances, or PII for financial gain, competitive intelligence, or in response to coercion. Insider threats are particularly dangerous in FinTech environments because legitimate privileged users can access sensitive data without triggering standard perimeter-based controls. The risk is elevated during the current period of rapid hiring and frequent engineering rotation across teams. | Insider Threat / Privilege Abuse | 3 | 5 | **15 (High)** | Req. 7.2.1 (Access control system covers all system components); Req. 8.2.2 (Group/shared credentials prohibited); Req. 10.2.1.5 (Log all use of root/administrative privileges); Req. 10.7.2 (Failures of critical security controls detected and reported) | CC6.3 (Role-Based Access / Least Privilege); CC6.2 (Access Provisioning); CC4.1 (Monitoring of Controls); CC7.3 (Evaluation of Security Events) | (1) Eliminate all direct production database access for application engineers — all data access must occur through application-layer APIs. DBA break-glass access requires dual-approval in PagerDuty and is time-limited to 4-hour sessions, fully session-recorded via AWS Systems Manager Session Manager. (2) Database Activity Monitoring (DAM) via AWS CloudWatch + custom Aurora audit log rules — alert on any SELECT > 1,000 rows, any schema changes, or access outside business hours. (3) Enforce column-level encryption on SSN, full PAN (where applicable), and account balance fields — encryption keys in HashiCorp Vault accessible only to application service accounts, not human users. (4) Quarterly access certification reviews: managers must recertify all database access grants for their direct reports; Vanta tracks completion rate as a SOC 2 control evidence artifact. (5) Pre-employment background checks for all personnel with CDE access; NDA and Acceptable Use Policy signed on Day 1. | **7 (Medium)** | CISO |

---

**Risk Register Summary**

| Risk ID | Description (Short) | Inherent Score | Residual Score | Priority |
|---|---|---|---|---|
| RISK-001 | API Authorization Bypass (BOLA) | 20 — Critical | 8 — Medium | P1 |
| RISK-002 | KYC Vendor Data Breach | 15 — High | 9 — Medium | P1 |
| RISK-003 | Cloud Misconfiguration (S3/Aurora) | 15 — High | 4 — Low | P2 |
| RISK-004 | Credential Stuffing / ATO | 16 — High | 6 — Low | P1 |
| RISK-005 | Insider Threat — Privileged DB Access | 15 — High | 7 — Medium | P1 |

**Next Review Date:** December 2026 | **Risk Register Owner:** CISO | **Approved By:** Chief Risk Officer

---

## 4. Vendor Security Assessment

### Third-Party Cloud Infrastructure Provider Security Questionnaire

**Vendor Name:** ________________________
**Assessment Date:** ________________________
**Completed By (Vendor):** ________________________
**NovaPay Assessor:** ________________________
**Vendor Tier Classification:** Tier 1 (Critical — processes or stores NovaPay cardholder data or customer PII)

---

### Purpose and Scope

This questionnaire is used by NovaPay Financial Technologies to assess the security posture of third-party cloud infrastructure and technology vendors prior to onboarding and during annual recertification. All Tier 1 vendors (those with access to NovaPay's production environment, cardholder data, or customer PII) are required to complete this assessment. Responses are subject to validation against supporting evidence (e.g., audit reports, penetration test summaries, policy excerpts).

---

### Scoring Rubric

**Per Question:**
- **2 points — Fully Implemented:** Control is in place, documented, tested, and evidence is available
- **1 point — Partially Implemented:** Control exists but has gaps, is untested, or documentation is incomplete
- **0 points — Not Implemented / Not Applicable:** Control is absent, planned but not yet in place, or vendor cannot provide evidence

**Total Score Interpretation:**

| Score Range | Rating | NovaPay Action |
|---|---|---|
| 18–20 | Excellent | Approve; standard annual review |
| 14–17 | Acceptable | Approve with documented exceptions; re-assess in 6 months |
| 10–13 | Marginal | Conditional approval; require remediation plan within 90 days; re-assess in 3 months |
| 0–9 | Unacceptable | Do not onboard; engage vendor for remediation roadmap; re-assess in 6 months |

---

### Section 1: Data Security & Encryption

**Question 1: Data Encryption at Rest and in Transit**

*Does your organization encrypt all data at rest and in transit using industry-standard algorithms? Please specify the encryption standards used for stored data and data transmitted over networks, and identify any categories of customer data that may be excluded from encryption.*

**Vendor Response:**
_______________________________________________________________

**Evidence Requested:** Encryption policy document; sample of KMS or HSM configuration; TLS certificate scan results from Qualys SSL Labs (minimum A rating)

**Score (Circle One):** 0 / 1 / 2
**Assessor Notes:** _______________________________________________________________

---

**Question 2: Key Management Lifecycle**

*Describe your organization's cryptographic key management practices, including key generation, rotation schedules, storage mechanisms (e.g., HSM, cloud KMS), access controls on key material, and procedures for key compromise and revocation. Specifically, how are encryption keys protecting NovaPay customer data managed, and who has access to them?*

**Vendor Response:**
_______________________________________________________________

**Evidence Requested:** Key management policy; HSM certification (FIPS 140-2 Level 2 or higher); evidence of annual key rotation for symmetric keys protecting sensitive data categories

**Score (Circle One):** 0 / 1 / 2
**Assessor Notes:** _______________________________________________________________

---

### Section 2: Access Control & Identity Management

**Question 3: Privileged Access Management and Multi-Factor Authentication**

*Does your organization enforce Multi-Factor Authentication (MFA) for all access to cloud management consoles, administrative dashboards, and any systems that store or process NovaPay data? Describe your Privileged Access Management (PAM) solution, including how privileged sessions are recorded, time-limited, and reviewed.*

**Vendor Response:**
_______________________________________________________________

**Evidence Requested:** MFA enforcement policy; screenshot of PAM tool configuration (e.g., CyberArk, BeyondTrust); sample privileged access review report from the past 6 months

**Score (Circle One):** 0 / 1 / 2
**Assessor Notes:** _______________________________________________________________

---

**Question 4: Logical Separation and Multi-Tenancy Controls**

*If your platform hosts multiple customers on shared infrastructure, describe the technical controls in place to ensure logical separation between customer environments. How do you ensure that a security incident or misconfiguration in one tenant's environment cannot expose NovaPay data? Has tenant isolation been validated by a third-party penetration test?*

**Vendor Response:**
_______________________________________________________________

**Evidence Requested:** Architecture diagram showing tenant isolation model; penetration test report summary (redacted) addressing multi-tenant isolation within the past 12 months; reference to applicable certifications (e.g., CSA STAR, ISO 27001)

**Score (Circle One):** 0 / 1 / 2
**Assessor Notes:** _______________________________________________________________

---

### Section 3: Vulnerability Management & Security Testing

**Question 5: Penetration Testing Program**

*How frequently does your organization conduct external penetration tests? Who conducts the tests (internal team or third-party firm)? Describe the scope of the most recent test, including whether it covered the infrastructure and application layers hosting NovaPay data. How are critical and high-severity findings tracked and remediated?*

**Vendor Response:**
_______________________________________________________________

**Evidence Requested:** Executive summary of the most recent penetration test (redacted); remediation tracking report showing current open findings; if a third party, provide the firm's name and CREST/GPEN certification

**Score (Circle One):** 0 / 1 / 2
**Assessor Notes:** _______________________________________________________________

---

**Question 6: Patch and Vulnerability Management**

*Describe your organization's vulnerability management program, including how vulnerabilities are discovered (scanning tools, CVE feeds, bug bounty), severity classification methodology (e.g., CVSS), and patching SLAs. Specifically, what are your committed timelines for patching Critical (CVSS ≥ 9.0) and High (CVSS 7.0–8.9) vulnerabilities in systems that process NovaPay data?*

**Vendor Response:**
_______________________________________________________________

**Evidence Requested:** Vulnerability management policy; last 90-day patch compliance report; evidence of scanning cadence (e.g., Qualys, Tenable scan history)

**Score (Circle One):** 0 / 1 / 2
**Assessor Notes:** _______________________________________________________________

---

### Section 4: Incident Response & Business Continuity

**Question 7: Security Incident Detection and Notification**

*Describe your organization's Security Operations Center (SOC) capabilities, including whether monitoring is 24/7, and your mean time to detect (MTTD) and mean time to respond (MTTR) targets for security incidents. If NovaPay data is involved in a confirmed or suspected security incident, what is your contractual commitment for notifying NovaPay, and what information will be provided in the initial notification?*

**Vendor Response:**
_______________________________________________________________

**Evidence Requested:** Incident response plan (redacted); SOC coverage model; breach notification SLA from executed service agreement or MSA; sample incident notification template

**Score (Circle One):** 0 / 1 / 2
**Assessor Notes:** _______________________________________________________________

---

**Question 8: Business Continuity and Disaster Recovery**

*Describe your organization's Business Continuity Plan (BCP) and Disaster Recovery (DR) program. What are your documented Recovery Time Objective (RTO) and Recovery Point Objective (RPO) commitments for services processing NovaPay data? How frequently are BCP/DR plans tested, and is NovaPay notified of test results or planned maintenance events that could affect availability?*

**Vendor Response:**
_______________________________________________________________

**Evidence Requested:** BCP/DR plan executive summary; most recent DR test results (tabletop or live failover); SLA documentation specifying RTO/RPO; uptime history (99.9%+ required for Tier 1 vendors)

**Score (Circle One):** 0 / 1 / 2
**Assessor Notes:** _______________________________________________________________

---

### Section 5: Compliance & Governance

**Question 9: Security Certifications and Audit Reports**

*What industry security certifications and audit reports does your organization currently maintain? Please indicate whether a current SOC 2 Type II report is available (within the last 12 months), and whether your organization holds PCI-DSS certification (Attestation of Compliance or Report on Compliance). Are there any qualified opinions, exceptions, or noted deficiencies in your most recent audit reports?*

**Vendor Response:**
_______________________________________________________________

**Evidence Requested:** Current SOC 2 Type II report (shared under NDA); PCI-DSS AOC or ROC summary; ISO 27001 certificate if applicable; any management responses to audit findings

**Score (Circle One):** 0 / 1 / 2
**Assessor Notes:** _______________________________________________________________

---

**Question 10: Subprocessor and Fourth-Party Risk Management**

*Does your organization use any subprocessors, sub-contractors, or fourth-party vendors who may have access to, or process, NovaPay customer data? If so, how do you vet and monitor these subprocessors? Do you maintain a current subprocessor registry? Will NovaPay be notified of material changes to subprocessor relationships (addition or removal) with sufficient lead time to review?*

**Vendor Response:**
_______________________________________________________________

**Evidence Requested:** Current subprocessor list; subprocessor vetting policy; sample of a subprocessor security assessment; contractual notification language regarding subprocessor changes

**Score (Circle One):** 0 / 1 / 2
**Assessor Notes:** _______________________________________________________________

---

### Assessment Summary

| Section | Questions | Max Score | Vendor Score |
|---|---|---|---|
| Data Security & Encryption | 1–2 | 4 | __ |
| Access Control & Identity | 3–4 | 4 | __ |
| Vulnerability Management & Testing | 5–6 | 4 | __ |
| Incident Response & BCP | 7–8 | 4 | __ |
| Compliance & Governance | 9–10 | 4 | __ |
| **TOTAL** | **1–10** | **20** | **__** |

**Overall Rating:** ________________________
**Recommendation:** ________________________
**Conditions / Remediation Required:** ________________________
**Re-Assessment Date:** ________________________
**NovaPay Assessor Signature:** ________________________

---

## 5. Incident Response Plan

### NovaPay Financial Data Breach Incident Response Plan
**Framework Alignment:** NIST SP 800-61 Rev. 2 — Computer Security Incident Handling Guide
**Plan Owner:** Chief Information Security Officer (CISO)
**Last Tested:** Tabletop Exercise — March 2026
**Review Cycle:** Annually or after any declared incident

---

### 5.1 Plan Objectives

This Incident Response Plan (IRP) establishes NovaPay's structured approach for detecting, containing, eradicating, and recovering from cybersecurity incidents involving customer financial data, cardholder data, or regulated PII. The plan is designed to:

- Minimize harm to NovaPay customers, partners, and the organization
- Meet regulatory notification obligations (PCI-DSS, Regulation E, state breach notification laws)
- Preserve forensic evidence for legal and regulatory proceedings
- Restore normal business operations as rapidly as possible
- Support continuous improvement through post-incident review

---

### 5.2 Incident Classification

| Severity | Criteria | Example | Response SLA |
|---|---|---|---|
| **P1 — Critical** | Confirmed unauthorized access to cardholder data or customer PII; active exfiltration in progress; ransomware deployment in CDE | Attacker exfiltrating Aurora DB containing transaction records | Immediate — CISO notified within 15 minutes; IRT fully assembled within 1 hour |
| **P2 — High** | Suspected breach pending confirmation; significant system compromise; large-scale ATO campaign | Unusual API traffic patterns consistent with credential stuffing; DB query anomaly alert | CISO notified within 1 hour; investigation initiated within 2 hours |
| **P3 — Medium** | Isolated compromise of non-CDE system; confirmed policy violation; phishing attack on employee | Employee workstation malware infection; failed phishing simulation requiring remediation | Security team notified within 4 hours; investigation within 8 hours |
| **P4 — Low** | Anomaly not yet confirmed as security incident; routine policy violation | Unusual login time for non-privileged employee; failed MFA attempt | Logged and reviewed within 24 hours |

---

### 5.3 Incident Response Team (IRT)

| Role | Responsibility | Primary Contact |
|---|---|---|
| **Incident Commander (CISO)** | Overall incident leadership; executive escalation; regulatory notification decisions | CISO |
| **Security Lead** | Technical investigation; forensic analysis; containment execution | Director of Security Engineering |
| **Legal Counsel** | Legal risk assessment; attorney-client privilege over investigation; regulatory counsel | VP Legal / Outside Counsel (Debevoise LLP) |
| **Compliance Officer** | Regulatory notification obligations; PCI-DSS breach reporting to card brands | Chief Compliance Officer |
| **Communications Lead** | Internal communications; customer notifications; press statements | VP Marketing / PR Firm |
| **Engineering Lead** | Infrastructure remediation; system restoration; deployment of fixes | VP Engineering |
| **Customer Support Lead** | Customer-facing communications; fraud claim processing; account freezes | Director of Customer Operations |
| **Finance** | Fraud loss quantification; cyber insurance engagement; breach cost tracking | CFO / Finance Controller |

---

### 5.4 Incident Response Phases

This IRP follows the four phases defined in NIST SP 800-61 Rev. 2:

**Phase 1 → Preparation**
**Phase 2 → Detection & Analysis**
**Phase 3 → Containment, Eradication & Recovery**
**Phase 4 → Post-Incident Activity**

---

#### PHASE 1: PREPARATION

*Ongoing — executed before any incident occurs*

**1.1 Team Readiness**

- IRT contact roster maintained in PagerDuty with 24/7 on-call rotation; reviewed quarterly
- All IRT members complete annual NIST 800-61 aligned incident response training
- Bi-annual tabletop exercises simulating financial breach scenarios (e.g., BOLA-driven data exfiltration; ransomware in CDE)
- Dedicated incident response Slack workspace (`#ir-active`) with pre-configured channel templates

**1.2 Tooling & Evidence Preservation**

- Forensic workstations pre-imaged and accessible to Security Lead
- AWS CloudTrail and VPC Flow Logs configured with 12-month retention in a separate, write-protected S3 bucket
- Datadog SIEM correlation rules reviewed quarterly; playbooks linked directly from SIEM alerts
- Legal hold procedures documented; outside counsel on retainer for breach response

**1.3 Regulatory Contacts Pre-established**

| Regulator / Entity | Contact | Notification Trigger |
|---|---|---|
| Card Brands (Visa, Mastercard) | Visa GARM; MC GSOP hotline | Confirmed PCI breach — notify within 72 hours of confirmed breach |
| PCI-DSS QSA | Coalfire Systems | Upon confirmed or suspected CDE compromise |
| FinCEN | BSA E-Filing System | Suspicious Activity Report (SAR) within 30 days of identifying suspicious activity |
| State AGs | Per-state breach notification portals | Per applicable state law (most require 30–72 hours) |
| Acquiring Bank | Coastal Community Bank — Security Contact | Immediately upon confirmed card data compromise |
| Cyber Insurance | Coalition Insurance — Breach Coach Hotline | Within 24 hours of suspected P1/P2 incident |

---

#### PHASE 2: DETECTION & ANALYSIS

*NIST SP 800-61 § 3.2*

**2.1 Detection Sources**

Incidents may be detected through the following channels:

- Datadog SIEM alert (automated correlation rule triggering on anomalous database queries, API abuse patterns, or IAM changes)
- AWS GuardDuty finding (e.g., `UnauthorizedAccess:IAMUser/MaliciousIPCaller`)
- Sardine AI fraud detection alert (velocity anomaly on P2P send endpoint)
- Customer complaint (report of unauthorized transaction via support channel)
- External researcher disclosure (via security@novapay.com or HackerOne bug bounty program)
- Card brand alert (Visa or Mastercard notifying NovaPay of a common point of purchase — CPP — investigation)
- Internal employee report (AUP violation or anomalous behavior observed by colleague)

**2.2 Initial Triage Checklist (Security On-Call Engineer)**

Upon receiving an alert or report:

1. Log the incident in the IRT ticketing system (Jira Security project) within 15 minutes of detection
2. Assign a preliminary severity rating (P1–P4) based on the classification matrix in Section 5.2
3. For P1/P2: immediately page the CISO and Security Lead via PagerDuty; open `#ir-active` Slack channel
4. Preserve all available log data — do NOT delete, modify, or restart affected systems before forensic snapshot
5. Begin incident timeline documentation in the Jira ticket (timestamp every action taken)
6. Conduct initial scoping questions:
   - What systems are confirmed or suspected to be involved?
   - Is the threat actor still present (active intrusion)?
   - What data types are confirmed or potentially at risk (PAN, PII, account credentials)?
   - What is the estimated number of affected accounts?

**2.3 Forensic Analysis (Security Lead)**

- Acquire memory dumps and disk images from affected EC2 instances / EKS nodes before any containment that would destroy volatile data
- Collect and archive: CloudTrail logs, VPC Flow Logs, Aurora audit logs, application logs, WAF logs for the relevant time window — place in a dedicated, write-protected S3 evidence bucket
- Identify the initial attack vector (e.g., exploited vulnerability, compromised credential, misconfiguration)
- Determine the full scope of data potentially accessed or exfiltrated:
  - Query Aurora audit logs for all `SELECT` statements on sensitive tables during the incident window
  - Review S3 access logs for any unusual data egress
  - Examine API Gateway access logs for anomalous request patterns
- Document IOCs (Indicators of Compromise): malicious IPs, user agents, compromised account IDs, modified files
- Determine the attacker's lateral movement path through the environment
- Assign a confirmed severity and brief the CISO with preliminary findings within 4 hours of P1 declaration

---

#### PHASE 3: CONTAINMENT, ERADICATION & RECOVERY

*NIST SP 800-61 § 3.3*

**3.1 Short-Term Containment (Security Lead + Engineering Lead)**

*Goal: Stop the bleeding without destroying forensic evidence*

- Isolate affected EKS pods/nodes using Kubernetes NetworkPolicy to block all inbound/outbound traffic (preserve; do not terminate)
- Revoke all IAM credentials and API keys associated with compromised accounts or services immediately via AWS IAM — document each revocation in the Jira ticket
- Block identified malicious IP addresses at the AWS WAF and Security Group level
- If active exfiltration is detected: coordinate with VPC networking to null-route egress to attacker-controlled IPs
- Engage Coastal Community Bank to place fraud monitoring holds on affected card accounts
- Activate Sardine AI enhanced fraud monitoring rules for the affected customer cohort
- Freeze affected customer accounts if unauthorized transactions are confirmed (Regulation E obligation to investigate within 10 business days)

**3.2 Legal & Regulatory Notification Determination (Compliance Officer + Legal Counsel)**

Conduct a formal breach notification assessment within 24 hours of P1/P2 incident declaration:

- Quantify: number of affected customers; data elements exposed (PAN, CVV, SSN, account balance, transaction history)
- Assess PCI-DSS notification obligation: if CHD confirmed compromised → notify acquiring bank and card brands within 72 hours
- Assess state breach notification laws: if PII confirmed exposed → compile per-state notification requirements and deadlines
- Assess FinCEN SAR obligation: if suspicious activity is detected that may constitute money laundering, terrorist financing, or other financial crime → file SAR within 30 calendar days
- Open cyber insurance claim with Coalition Insurance; engage breach coach
- Attorney-client privilege: all written communications between Legal and IRT regarding breach scope and notification should be marked "PRIVILEGED AND CONFIDENTIAL — ATTORNEY-CLIENT COMMUNICATION"

**3.3 Long-Term Containment and Eradication (Security Lead + Engineering Lead)**

*Goal: Eliminate the root cause and ensure no persistence mechanisms remain*

- Deploy patched application version to resolve the exploited vulnerability (emergency change process; abbreviated CAB approval required)
- Rotate all secrets and credentials in HashiCorp Vault for affected service accounts; regenerate all API keys for affected vendor integrations
- Conduct a full malware scan of all CDE systems using CrowdStrike and AWS GuardDuty; validate clean bill of health before proceeding to recovery
- Review and harden IAM policies, Security Groups, and Kubernetes RBAC configurations implicated in the incident
- Validate that attacker persistence mechanisms (backdoors, unauthorized IAM users, modified Lambda functions) have been fully removed
- Conduct a secondary forensic review to confirm no additional compromised systems remain

**3.4 Recovery (Engineering Lead + Security Lead)**

- Restore affected systems from last verified clean snapshot (Aurora Point-in-Time Recovery to a timestamp prior to the incident)
- Deploy restored systems initially to an isolated environment; validate functionality and security configuration before returning to production
- Gradually restore production traffic using weighted routing in AWS API Gateway — monitor Datadog dashboards intensively during phased restoration
- Confirm cardholder data environment integrity with QSA before resuming card transaction processing if CDE was affected
- Issue customer notifications per regulatory requirements (see Section 5.5)
- Confirm all Regulation E fraud claims are filed and refund timelines communicated to affected customers

---

#### PHASE 4: POST-INCIDENT ACTIVITY

*NIST SP 800-61 § 3.4*

**4.1 Post-Incident Review (Full IRT — within 5 business days of containment)**

Conduct a formal Post-Incident Review (PIR) / "lessons learned" meeting. Document the following:

- Detailed incident timeline from first indicator of compromise to full recovery
- Root cause analysis (RCA): What was the underlying technical and process failure?
- What detection controls worked? Which failed or were slow?
- What containment actions were effective? What caused delays?
- Were regulatory notification timelines met?
- What is the estimated total cost of the incident (fraud losses, customer remediation, legal/regulatory, staff hours, reputational impact)?

**4.2 Remediation Action Items**

Each identified gap must have:
- A specific remediation action
- An assigned owner (name and title)
- A deadline (30/60/90 days based on severity)
- Tracked in Jira as a security remediation epic linked to the incident ticket

**4.3 Control Updates and Evidence**

- Update the Risk Register to reflect any new or elevated risks identified during the incident
- Document control updates as SOC 2 control evidence in Drata (GRC platform)
- If a PCI-DSS control was found deficient, engage QSA to assess whether interim compensating controls are required pending full remediation
- Deliver final incident report to CISO, Board Risk Committee, and (if required) regulatory bodies

**4.4 Required Regulatory Reports Post-Incident**

| Report | Recipient | Timeline |
|---|---|---|
| PCI-DSS Forensic Investigation Report | Card Brands + Acquiring Bank | As required by card brand security program |
| Suspicious Activity Report (SAR) | FinCEN | Within 30 days of detecting suspicious activity |
| State Breach Notification | Affected state AGs and residents | Per state law (typically 30–72 hours from confirmation) |
| Regulation E Resolution Notice | Affected customers | Written explanation of findings within 3 business days of completing investigation |
| Board Risk Committee Briefing | NovaPay Board | Within 30 days of incident closure |
| Post-Incident Report (Final) | CISO, Legal, Compliance | Within 15 business days of incident closure |

---

### 5.5 Customer Notification Template (P1 — Financial Data Breach)

> **Subject:** Important Security Notice Regarding Your NovaPay Account
>
> Dear [Customer Name],
>
> We are writing to inform you of a security incident that may have affected your NovaPay account. We take the security of your financial information extremely seriously, and we want to be transparent with you about what happened, what information was involved, and the steps we are taking to protect you.
>
> **What Happened:** On [Date], NovaPay identified unauthorized access to [describe system/data affected] affecting accounts including yours.
>
> **What Information Was Involved:** [Specify data elements: e.g., name, email, partial card number]
>
> **What We Are Doing:** We have [describe containment actions taken]. We have also reported this incident to the appropriate regulatory authorities.
>
> **What You Should Do:** We recommend that you [monitor your account for unauthorized transactions / place a fraud alert with the credit bureaus / change your NovaPay password immediately]. If you notice any unauthorized transactions, please contact us immediately at 1-800-XXX-XXXX or security@novapay.com.
>
> We sincerely apologize for any inconvenience or concern this may cause. Protecting your financial data is our highest priority.
>
> — The NovaPay Security Team

---

### 5.6 Incident Response Workflow Diagram

```
DETECTION
    │
    ▼
[Alert / Report Received] ──► Log in Jira ──► Assign Severity (P1-P4)
    │
    ├── P3/P4: Security Team handles; standard workflow
    │
    └── P1/P2: ▼
              Page CISO + Security Lead (PagerDuty)
              Open #ir-active Slack channel
                    │
                    ▼
              ANALYSIS
              Preserve logs + forensic snapshots
              Determine: scope | attack vector | data at risk
              Preliminary brief to CISO within 4 hours
                    │
                    ▼
              CONTAINMENT
              Isolate affected systems
              Revoke compromised credentials
              Block malicious IPs
              Legal / Regulatory notification assessment
                    │
                    ▼
              ERADICATION
              Patch vulnerability / remove malware
              Rotate all affected secrets
              Confirm no persistence mechanisms remain
                    │
                    ▼
              RECOVERY
              Restore from clean snapshots
              Phased traffic restoration
              Customer notifications issued
              Regulatory reports filed
                    │
                    ▼
              POST-INCIDENT REVIEW
              RCA documented within 5 days
              Remediation action items assigned
              Risk register + controls updated
              Final report to Board
```

---

*End of Document*

---

**Document Control**

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | June 2026 | [Your Name], GRC Analyst | Initial draft |

**Disclaimer:** This document is a fictional portfolio project created for educational and professional development purposes. All company names, regulatory contacts, and financial figures are illustrative. This document does not constitute legal or compliance advice.
