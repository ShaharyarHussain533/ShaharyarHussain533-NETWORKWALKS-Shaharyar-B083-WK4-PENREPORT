# Mediroza General Hospital — Penetration Testing & Assessment Report

**Prepared By:** Syed Shaharyar Hussain  
![Project Status](https://img.shields.io/badge/Status-Completed-success)
![Type](https://img.shields.io/badge/Type-Black--box%20Pentest-blue)
![Target](https://img.shields.io/badge/Target-https%3A%2F%2Fmedirozahospital.com-lightgrey)

---

## 📄 Executive Summary

This document compiles the findings, evidence, and remediation guidance for a 5-day black-box penetration testing assessment conducted against **Mediroza General Hospital** (`https://medirozahospital.com`)[cite: 10, 11, 17].

The primary objective of the assessment was to evaluate the application's overall security controls, test authentication mechanisms, evaluate sensitive file protections, and assess the risk of unauthorized access to Protected Health Information (PHI) and Personally Identifiable Information (PII)[cite: 11, 12].

### Key Findings Summary
* **Authentication Bypass via SQL Injection:** Unsanitized user input handling on the patient login portal allowed an unauthenticated attacker to bypass login validation and access patient records[cite: 12, 13].
* **Exfiltration of Confidential Patient Reports:** Three pathology laboratory reports containing sensitive medical details were retrieved[cite: 2, 4, 6, 8, 12, 13].
* **Weak PDF Protection:** Password protection on all three patient reports relied on trivial dictionary passwords, enabling offline password recovery[cite: 5, 7, 9, 12, 14].
* **Exposure of Database Backup & Staff PII:** Examination of document metadata led to the discovery of an internal SQL backup file containing complete staff HR records, including national ID numbers and salary details[cite: 1, 3, 12, 15].

---

## 🎯 Assessment Scope & Milestones

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Milestone 1   │ ──► │   Milestone 2   │ ──► │   Milestone 3   │ ──► │   Milestone 4   │
│ Initial Access  │     │ Data Extraction │     │ Server Exposure │     │ Pentest Report  │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘


* **Milestone 1: Initial Access & Authentication Bypass** — Identify entry points, bypass authentication on `/patient/login.php`, and retrieve lab reports[cite: 12, 13].
* **Milestone 2: Data Extraction & PDF Access** — Extract file hashes and recover cleartext contents for all 3 patient PDF files[cite: 12, 14].
* **Milestone 3: Critical Data Exposure Analysis** — Analyze document properties, correlate metadata (`j.malik`), and extract internal employee/shareholder records from server backup files[cite: 1, 3, 12, 15].
* **Milestone 4: Reporting & Remediation** — Document findings, assign risk ratings, and outline actionable security remediation steps[cite: 16].

---

## 🔍 Comprehensive Findings & Technical Analysis

### Finding M1: Authentication Bypass via SQL Injection
* **Target Endpoint:** `/patient/login.php`[cite: 13]
* **Vulnerable Parameter:** `Username`[cite: 13]
* **Payload Used:** `admin' -- `[cite: 13]
* **Mechanism:** The backend query constructed SQL statements using direct string concatenation without parameterization[cite: 13]. Submitting a single quote terminated the string literal, and the trailing comment sequence (`-- `) instructed the database engine to ignore the remainder of the query (including password validation)[cite: 13].
* **Impact:** Unauthenticated access to the patient portal and exfiltration of 3 protected lab reports[cite: 12, 13].

---

### Finding M2: Weak PDF Document Password Protection
All three retrieved PDF files were protected using standard PDF password security[cite: 7, 9, 14]. Offline dictionary attacks successfully recovered all passwords[cite: 5, 7, 9, 14]:

1. **`patient_report_1.pdf`**
   * **Password:** `123456`[cite: 9]
   * **Patient Name:** Sipho Dlamini | **ID:** MG-P-10231 | **DOB:** 1984-06-12[cite: 8]
   * **Referring Doctor:** Dr. Anita Naicker[cite: 8]
   * **Key Finding:** White Cell Count elevated at **11.8 x10^9/L** (FLAG: **HIGH**)[cite: 8].

2. **`patient_report_2.pdf`**
   * **Password:** `password`[cite: 7]
   * **Patient Name:** Priya Reddy | **ID:** MG-P-10244 | **DOB:** 1991-02-28[cite: 6]
   * **Referring Doctor:** Dr. Johan van der Merwe[cite: 6]
   * **Key Finding:** Elevated Lipid Profile (Total Cholesterol **6.4 mmol/L**, LDL **4.1 mmol/L**, Triglycerides **1.8 mmol/L**)[cite: 6].

3. **`patient_report_3.pdf`**
   * **Password:** `!@#$%^&`[cite: 5]
   * **Patient Name:** Emily Thompson | **ID:** MG-P-10258 | **DOB:** 1978-09-03[cite: 4]
   * **Referring Doctor:** Dr. Ahmed Kara[cite: 4]
   * **Key Finding:** Low Haemoglobin (**11.4 g/dL**), Ferritin (**9 ug/L**), and Vitamin D (**42 nmol/L**)[cite: 4].

---

### Finding M3: Database Backup & Internal Staff Exposure
* **Source Artifact:** `mediroza_db_backup_2019.sql` (Database: `mediroza_hr`, Table: `staff`)[cite: 1]
* **Discovery Method:** Metadata analysis of `patient_report_3.pdf` revealed the author attribute set to `j.malik` (IT Systems Administrator Jameel Malik)[cite: 1, 3, 15].
* **Impact:** Exposed personal identities, salary details, and South African National ID numbers for 30 staff members[cite: 1, 12, 15].

#### Sample Extract of Exposed HR Data

| ID | Full Name | Job Title | Department | Email Address | Phone Number | National ID | Monthly Salary (ZAR) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | Dr. Rajesh Naidoo[cite: 1] | Chief Pathologist[cite: 1] | Diagnostics Lab[cite: 1] | `r.naidoo@medirozahospital.com`[cite: 1] | +27 82 101 2007[cite: 1] | 85021013011081[cite: 1] | R 138,000[cite: 1] |
| 2 | Sarah Botha[cite: 1] | Chief Financial Officer[cite: 1] | Finance[cite: 1] | `s.botha@medirozahospital.com`[cite: 1] | +27 82 102 2014[cite: 1] | 85031023022082[cite: 1] | R 152,000[cite: 1] |
| 3 | Dr. Johan van der Merwe[cite: 1] | Medical Director[cite: 1] | Management[cite: 1] | `j.merwe@medirozahospital.com`[cite: 1] | +27 82 103 2021[cite: 1] | 85041033033083[cite: 1] | R 160,000[cite: 1] |
| 4 | Dr. Anita Naicker[cite: 1] | Consultant Cardiologist[cite: 1] | Cardiology[cite: 1] | `a.naicker@medirozahospital.com`[cite: 1] | +27 82 104 2028[cite: 1] | 85051043044084[cite: 1] | R 132,000[cite: 1] |
| 5 | Dr. Ahmed Kara[cite: 1] | Consultant Physician[cite: 1] | Internal Medicine[cite: 1] | `a.kara@medirozahospital.com`[cite: 1] | +27 82 105 2035[cite: 1] | 85061053055085[cite: 1] | R 128,000[cite: 1] |
| 9 | Jameel Malik[cite: 1] | IT Systems Administrator[cite: 1] | IT[cite: 1] | `j.malik@medirozahospital.com`[cite: 1] | +27 82 109 2063[cite: 1] | 85101093099080[cite: 1] | R 58,000[cite: 1] |

---

## 📊 Summary Table of Exfiltrated Assets

| File / Asset | Patient / Subject | Referring Doctor / Role | Password | Summary of Findings |
| :--- | :--- | :--- | :--- | :--- |
| `patient_report_1.pdf`[cite: 8] | Sipho Dlamini[cite: 8] | Dr. Anita Naicker[cite: 8] | `123456`[cite: 9] | Elevated White Cell Count (**11.8 x10^9/L**)[cite: 8] |
| `patient_report_2.pdf`[cite: 2, 6] | Priya Reddy[cite: 6] | Dr. Johan van der Merwe[cite: 6] | `password`[cite: 7] | Elevated Lipid Profile (Cholesterol **6.4 mmol/L**)[cite: 6] |
| `patient_report_3.pdf`[cite: 3, 4] | Emily Thompson[cite: 4] | Dr. Ahmed Kara[cite: 4] | `!@#$%^&`[cite: 5] | Low Haemoglobin (**11.4 g/dL**), Low Ferritin (**9 ug/L**)[cite: 4] |
| `mediroza_db_backup_2019.sql`[cite: 1] | 30 Staff Members[cite: 1] | Hospital Personnel[cite: 1] | None (Unprotected)[cite: 1] | Complete HR directory, payroll, and National IDs[cite: 1] |

---

## 🛡️ Risk Rating & Remediation Roadmap

### Vulnerability Severity Breakdown
1. **SQL Injection (`/patient/login.php`):** **CRITICAL** — Allows complete authentication bypass and access to patient medical records[cite: 12, 13, 16].
2. **Database Backup Exposure (`mediroza_db_backup_2019.sql`):** **CRITICAL** — Unprotected storage of full staff identities and financial data[cite: 1, 15, 16].
3. **Weak Document Passwords:** **HIGH** — Trivial passwords allow rapid offline access to protected medical PDF reports[cite: 5, 7, 9, 14, 16].
4. **Information Disclosure via Metadata:** **MEDIUM** — Internal system user accounts (`j.malik`) leaked in document metadata[cite: 1, 3, 15, 16].

### Remediation Guidance
1. **Implement Prepared Statements:** Use parameterized SQL queries across all database handlers to neutralize SQL injection flaws[cite: 13].
2. **Remove Exposed Backups:** Store database dumps (`.sql`) in secured, off-site environments with restricted access controls[cite: 1].
3. **Strengthen Document Protection:** Enforce strong, randomly generated passwords for all exported medical reports[cite: 5, 7, 9, 14].
4. **Sanitize Document Metadata:** Configure PDF export utilities to strip internal system usernames and metadata attributes prior to publishing documents[cite: 2, 3].

---

## ⚠️ Disclaimer

> This project was conducted in a controlled testing environment for educational and
