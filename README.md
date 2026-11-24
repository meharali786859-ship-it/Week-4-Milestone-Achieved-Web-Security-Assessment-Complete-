# 🏥 Black-Box Security Assessment: Mediroza General Hospital

[![Vulnerability Assessment](https://img.shields.io/badge/Assessment-Black--Box%20VAPT-red.svg)](https://github.com)
[![Target](https://img.shields.io/badge/Target-Simulated%20Healthcare%20App-blue.svg)](https://github.com)
[![Status](https://img.shields.io/badge/Status-Findings%20Documented-orange.svg)](https://github.com)
[![Cohort](https://img.shields.io/badge/Batch-B084%20NETWORKWALKS-green.svg)](https://github.com)

A comprehensive black-box vulnerability assessment and penetration testing (VAPT) project conducted against **Mediroza General Hospital** (`https://medirozohospital.com`) as part of the **NETWORKWALKS Cybersecurity Internship (Week 4 Capstone)**.

---

## 📌 Executive Summary

The objective of this assessment was to evaluate the external security posture of a simulated healthcare web infrastructure running **Mediroza CMS 1.4.2** on a **LiteSpeed Web Server**. 

The assessment identified **9 distinct vulnerabilities**, including **3 CRITICAL** findings that allow:
1. **Unauthenticated Directory Indexing:** Direct exposure of sensitive endpoints (`/patient/`, `/staff/`, `/old/`).
2. **Confidential Database Backup Exposure (M3):** Publicly downloadable `.sql` dump containing staff salaries (in ZAR), National IDs, and shareholder equity data.
3. **SQL Injection Authentication Bypass (M1):** Full administrative login bypass on the Patient Portal using standard SQLi payloads.

---

## 📊 Findings Severity Matrix

| # | Vulnerability Finding | Severity | Milestone | Status |
|---|----------------------|:--------:|:---------:|:------:|
| 1 | Directory Indexing Enabled (`/patient/`, `/staff/`, `/old/`) | `CRITICAL` | M1 / M3 | Open |
| 2 | Sensitive Database Backup Exposed in `/old/` | `CRITICAL` | M3 | Open |
| 3 | Sensitive Directories Disclosed in `robots.txt` | `HIGH` | M1 | Open |
| 4 | Missing Modern HTTP Security Headers | `MEDIUM` | M1 | Open |
| 5 | Server & CMS Version Information Disclosure | `LOW` | M1 | Open |
| 6 | BREACH Attack Possible (HTTP Deflate Compression) | `LOW` | M1 | Open |
| 7 | Email Address Disclosed (Harvesting / Phishing Risk) | `INFO` | M1 | Open |
| 8 | SQL Injection in Patient Portal Login | `CRITICAL` | M1 | Open |
| 9 | Sensitive Medical PDFs Exposed & Weak Passwords Cracked | `HIGH` | M2 | Open |

---

## 🛠️ Methodology & Phases

### 🔍 Phase M1: Reconnaissance & Web Scanning
- **Tools Deployed:** `whois`, `dnsrecon`, `nslookup`, `whatweb`, `wafw00f`, `nikto`, `curl`.
- **Activities:** Fingerprinted LiteSpeed server, bypassed WAF User-Agent blocking, analyzed `robots.txt`, and validated SQL Injection on the Patient Portal.

<p align="center">
  <img src="evidence_screenshots/01_recon_whois_whatweb.png" width="48%" />
  <img src="evidence_screenshots/02_nikto_wafw00f_scan.png" width="48%" />
</p>

---

### 🔓 Phase M2: Password Cracking & Cryptographic Review
- **Tools Deployed:** `pdf2john`, `john the ripper`, `rockyou.txt`.
- **Impact:** Downloaded restricted medical records from `/patient/`. Extracted PDF cryptographic hashes and cracked weak passwords (`123456`, `password`), exposing confidential patient treatment plans (PHI breach).

<p align="center">
  <img src="evidence_screenshots/08_pdf_cracking_john_m2.png" width="48%" />
  <img src="evidence_screenshots/09_decrypted_patient_report.png" width="48%" />
</p>

---

### 📂 Phase M3: Sensitive Data Exposure (Complete Database Breach)
- **Vulnerability:** Unrestricted Directory Indexing on `/old/`.
- **Impact:** Downloaded `mediroza_db_backup_2019.sql` (14.8 MB) containing unencrypted staff monthly salaries, South African National IDs, and corporate shareholder equity registers.

<p align="center">
  <img src="evidence_screenshots/04_directory_indexing_old.png" width="48%" />
  <img src="evidence_screenshots/05_db_backup_sql_leak.png" width="48%" />
</p>

---

### 💉 Critical Finding Spotlight: SQL Injection (Auth Bypass)
- **Affected URL:** `https://medirozohospital.com/patient/login.php`
- **Vulnerable Logic:** Direct concatenation of user input in backend SQL authentication query.
- **Exploit Payload:** `admin' --` in username parameter.
- **Outcome:** Bypassed password check, gaining immediate access to the Patient Care Management Portal.

<p align="center">
  <img src="evidence_screenshots/06_sqli_patient_portal_bypass.png" width="85%" />
</p>

---

## 🛡️ Strategic Recommendations

1. **Disable Directory Indexing:** Add `Options -Indexes` to `.htaccess` or LiteSpeed server configuration.
2. **Secure Database Backups:** Purge all `.sql` backups from public web roots and store them in encrypted, access-restricted cloud vaults.
3. **Use Prepared Statements:** Refactor database queries using parameterized PDO queries:
   ```php
   $stmt = $pdo->prepare("SELECT id, username, password_hash FROM users WHERE username = :username");
   $stmt->execute(['username' => $_POST['username']]);
   ```
4. **Enforce Strong PDF Access Control:** Never expose raw files in public directories; enforce session-based authorization checks.
5. **Implement Security Headers:** Deploy `CSP`, `X-Frame-Options: DENY`, `HSTS`, and `X-Content-Type-Options: nosniff`.

---

## 📑 Full Assessment Report
For the complete technical breakdown, CVSS scores, and remediation code, refer to the compiled PDF report:
- 📥 **[Download Security Assessment Report (PDF)](Security_Assessment_Report_Mediroza.pdf)**

---

## ⚖️ Disclaimer
*All tests and assessments documented in this repository were conducted strictly on an authorized, simulated learning target provided by NETWORKWALKS for educational purposes. No unauthorized scanning or testing was conducted against production or non-simulated entities.*

---

## 🤝 Acknowledgements
Special thanks to **Sir Waqas Karim (CCIE)** and the **NETWORKWALKS** team for designing this hands-on, realistic cybersecurity curriculum.
