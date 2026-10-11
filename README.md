# Task 04: OWASP Web Testing Lab Report

## 1. Executive Summary
This document records verified security findings identified during web application testing on the local OWASP Juice Shop instance (`http://localhost:3000`). All vulnerability testing was conducted strictly within the authorized local lab boundary.

---

## 2. OWASP Findings Summary Table

| ID | Vulnerability Category | Risk Level | Affected Endpoint | Root Cause |
| :--- | :--- | :--- | :--- | :--- |
| **VULN-01** | Injection (SQL Injection) | **High** | `/#/login` (`/rest/user/login`) | Unsanitized SQL queries allowing authentication bypass. |
| **VULN-02** | Broken Access Control (BOLA/IDOR) | **High** | `/api/BasketItems/` | Missing authorization checks on object identifier parameter. |
| **VULN-03** | Injection (Cross-Site Scripting - XSS) | **Medium** | `/#/search` (`?q=`) | Reflected user input without proper HTML output encoding. |
| **VULN-04** | Security Misconfiguration | **Low** | HTTP Response Headers | Missing security headers (`X-Frame-Options`, `Content-Security-Policy`). |

---

## 3. Detailed Vulnerability Analyses

### Finding 01: Authentication Bypass via SQL Injection (SQLi)
* **Severity:** High
* **Affected Endpoint:** `http://localhost:3000/#/login`
* **Proof of Concept (PoC):**
  * Input payload into Email field: `' OR 1=1 --`
  * Password: `password123`
  * **Result:** Successfully logged in as the admin user (`admin@juice-sh.op`) without valid credentials.
* **Root Cause:** Backend database queries directly concatenate user input without using parameterized queries or prepared statements.
* **Business Impact:** Unauthorized access to administrator privileges, data leak, and complete database compromise.
* **Remediation:** Implement parameterized queries (Prepared Statements) or use ORM frameworks (e.g., Sequelize) to sanitize user inputs.

---

### Finding 02: Broken Object Level Authorization (IDOR)
* **Severity:** High
* **Affected Endpoint:** `http://localhost:3000/api/BasketItems/1`
* **Proof of Concept (PoC):**
  * Intercepted HTTP Request using proxy tool.
  * Modified payload parameter `BasketId` from `1` to `2`.
  * **Result:** Accessed and modified shopping cart items belonging to another user.
* **Root Cause:** Application relies on client-side parameters without server-side validation of user ownership over the requested resource.
* **Business Impact:** Unauthorized viewing and modification of sensitive user data across accounts.
* **Remediation:** Enforce server-side access control validation based on the authenticated user's session token/JWT.

---

### Finding 03: DOM-based / Reflected Cross-Site Scripting (XSS)
* **Severity:** Medium
* **Affected Endpoint:** `http://localhost:3000/#/search?q=`
* **Proof of Concept (PoC):**
  * Injected payload into search parameter: `<iframe src="javascript:alert(`XSS`)">`
  * **Result:** Execution of arbitrary JavaScript code inside the client's browser session.
* **Root Cause:** Frontend renders raw search queries directly into DOM without proper sanitization.
* **Business Impact:** Session hijacking, cookie theft, and malicious client-side redirection.
* **Remediation:** Contextually encode dynamic output and implement a strict `Content-Security-Policy` (CSP) header.

---

## 4. Remediation Action Plan
1. **Input Validation:** Enforce strict allow-lists and sanitize all input vectors on frontend and backend.
2. **Parameterized Queries:** Refactor legacy raw queries to ORM/prepared queries.
3. **Session & Auth Rules:** Validate JWT claims server-side for every API endpoint request.
4. **Security Headers:** Configure security headers including CSP, HSTS, and X-Content-Type-Options.
