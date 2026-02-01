# 1️⃣ Introduction

**Tester(s):**  
- Name:  Dan Luchin

**Purpose:**  
- The purpose of this penetration test was to identify common security vulnerabilities in a locally deployed web application, with a focus on the registration functionality and HTTP security configuration, using an automated vulnerability scanning approach with OWASP ZAP.

**Scope:**  
- Tested components:
    * Web application running at ```http://localhost:8001```
    * Registration endpoint (**/register**)
    * Root endpoint (**/**)
    * Static resources (**/static/*.js, /static/*.css**)
- Exclusions:
    * No authenticated user areas
    * No external services
    * No denial-of-service or brute-force testing
    * No manual exploitation attempts
- Test approach: **Black-box**
    * The test approach is **Black-Box** because I did not inspected the source code, I tested the web application only via HTTP requests, and ZAP had no credentials or internal knowledge

**Test environment & dates:**  
- Start:  01.02.2026
- End:  01.02.2026
- Test environment details (OS, runtime, DB, browsers):
    * OS: Windows 11
    * Runtime: Docker Desktop with WSL2 backend
    * Web application: Docker-based lab application
    * Security tool: OWASP ZAP 2.17.0
    * Browser: Microsoft Edge

**Assumptions & constraints:**  
- The assessment was conducted under time constraints and relied solely on automated scanning. No user credentials were provided, and no authenticated testing was performed.

---

# 2️⃣ Executive Summary

**Short summary (1-2 sentences):**  
An automated vulnerability assessment using OWASP ZAP identified multiple medium- and low-severity security issues in the target web application, primarily related to missing CSRF protection and absent HTTP security headers.

ZAP results:
* High: 0
* Medium: 3
* Low: 1
**Overall risk level:** (Medium)

**Top 5 immediate actions:**  
1.  Implement anti-CSRF protection for all state-changing forms
2.  Add anti-clickjacking protection using X-Frame-Options or CSP
3.  Configure a Content-Security-Policy header
4.  Set X-Content-Type-Options to nosniff
5.  Perform follow-up testing after security headers are implemented

---

# 3️⃣ Severity scale & definitions

|  **Severity Level**  | **Description**                                                                                                              | **Recommended Action**           |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
|      🔴 **High**     | A serious vulnerability that can lead to full system compromise or data breach (e.g., SQL Injection, Remote Code Execution). | *Immediate fix required*         |
|     🟠 **Medium**    | A significant issue that may require specific conditions or user interaction (e.g., XSS, CSRF).                              | *Fix ASAP*                       |
|      🟡 **Low**      | A minor issue or configuration weakness (e.g., server version disclosure).                                                   | *Fix soon*                       |
| 🔵 **Info** | No direct risk, but useful for system hardening (e.g., missing security headers).                                            | *Monitor and fix in maintenance* |


---

# 4️⃣ Findings (filled with examples → replace)

> Fill in one row per finding. Focus on clarity and the most important issues.

| ID | Severity | Finding | Description | Evidence / Proof |
|------|-----------|----------|--------------|------------------|
| F-01 | 🟠 Medium | Absence of Anti-CSRF Tokens | The registration form does not include an anti-CSRF token, making it vulnerable to Cross-Site Request Forgery attacks where a victim could be forced to submit unintended requests. | ZAP alert for **/register**, evidence: <form action="/register" method="POST"> |
| F-02 | 🟠 Medium | Content Security Policy Header Not Set | The application does not define a Content-Security-Policy header, reducing protection against cross-site scripting and content injection attacks. | ZAP alerts for **/** and **/register** |
| F-03 | 🟠 Medium | Missing Anti-clickjacking Header | The application does not set X-Frame-Options or CSP frame-ancestors, allowing the pages to be embedded in iframes and exposed to clickjacking attacks. | ZAP alerts for / and /register |
| F-04 | 🟡 Low | X-Content-Type-Options Header Missing | The application does not set the X-Content-Type-Options header to nosniff, allowing browsers to perform MIME-type sniffing. | ZAP alerts affecting /, /register, and static resources |

---

# 5️⃣ OWASP ZAP Test Report (Attachment)

**Purpose:**  
- [Link to the OWASP ZAP scan results.](zap_report_round1.md)
