# 🛡️ Web Application Security Assessment & Penetration Testing

A comprehensive grey-box web application penetration testing project targeting **Damn Vulnerable Web Application (DVWA)** and **OWASP Juice Shop**. This repository documents discovery, vulnerability scanning, manual exploitation, proof-of-concept (PoC) steps, and source-code level remediation strategies for PHP and Node.js/Express environments.

## 📌 Table of Contents

* [Executive Overview](#-executive-overview)

* [Assessment Methodology & Tools](#-assessment-methodology--tools)

* [Key Findings & Vulnerability Matrix](#-key-findings--vulnerability-matrix)

* [Repository Structure](#-repository-structure)

* [Environment Setup](#-environment-setup)

* [Code Remediation Highlights](#-code-remediation-highlights)

* [Documentation & Reports](#-documentation--reports)

* [Disclaimer](#-disclaimer)

## 🧐 Executive Overview

This assessment evaluates the security posture of two distinct web architecture stacks:

1. **DVWA:** Traditional PHP/MySQL monolithic web application.

2. **OWASP Juice Shop:** Modern Single Page Application (SPA) built with Angular, Node.js, Express, and SQLite.

### **Summary of Vulnerability Severity Counts**

| 

| **Critical 🔴** | **High 🟠** | **Medium 🟡** | **Low/Info 🔵** | 
| **4** | **6** | **6** | **4** | 

## 🛠️ Assessment Methodology & Tools

The testing was conducted following the **OWASP Web Security Testing Guide (WSTG v4.2)** across four primary phases:

1. **Reconnaissance & Fingerprinting:** Service enumeration and web application component discovery.

2. **Automated Vulnerability Scanning:** Dynamic Application Security Testing (DAST) scanning for low-hanging vulnerabilities.

3. **Manual Exploitation & Validation:** Business logic testing, access control bypass, and request manipulation.

4. **Remediation Engineering:** Writing secure coding patches for PHP and Node.js.

### **Tools Used**

* **Network & Port Scanning:** `Nmap`

* **Web Server Fingerprinting:** `Nikto`

* **Dynamic Application Security Testing (DAST):** `OWASP ZAP`

* **Manual Interception & Exploitation:** `Burp Suite Professional` / `Community`

* **Database Exfiltration:** `SQLMap`

## 🚨 Key Findings & Vulnerability Matrix

| **Vulnerability ID** | **Vulnerability Class** | **CVSS v3.1** | **Risk Level** | **Target Scope** | 
| **SEC-001** | SQL Injection (SQLi) - Auth Bypass | 9.8 | **CRITICAL** | DVWA / Juice Shop Login | 
| **SEC-002** | Broken Authentication & Weak Hashing | 9.1 | **CRITICAL** | Login Endpoints & Session Tokens | 
| **SEC-003** | Server-Side Request Forgery (SSRF) | 8.6 | **HIGH** | Remote Asset / Image Imports | 
| **SEC-004** | Cross-Site Scripting (Reflected & Stored) | 8.2 | **HIGH** | Comment & Search Forms | 
| **SEC-005** | Insecure Direct Object Reference (IDOR) | 7.5 | **HIGH** | User Profile & Document APIs | 
| **SEC-006** | Security Misconfigurations / Missing Headers | 6.1 | **MEDIUM** | HTTP Server Responses | 

## 📂 Repository Structure

```
.
├── docs/
│   ├── DVWA_JuiceShop_Security_Assessment_Report.pdf
│   └── Executive_Summary.pdf
├── exploits/
│   ├── sqli_payloads.txt
│   ├── xss_payloads.txt
│   └── ssrf_targets.txt
├── remediations/
│   ├── php/
│   │   ├── secure_login.php
│   │   └── secure_ssrf.php
│   └── nodejs/
│       ├── secure_login.js
│       └── secure_ssrf.js
├── scans/
│   ├── nmap_recon.txt
│   ├── nikto_results.json
│   └── zap_report.html
└── README.md

```

## ⚙️ Environment Setup

### **1. Target Setup via Docker Compose**

Run both target environments locally using Docker:

```
# Clone this repository
git clone https://github.com/your-username/web-app-security-assessment.git
cd web-app-security-assessment

# Start DVWA container
docker run --rm -it -p 80:80 vulnerables/web-dvwa

# Start OWASP Juice Shop container
docker run --rm -it -p 3000:3000 bkimminich/juice-shop

```

* **DVWA:** Accessible at `http://localhost:80` (Default Credentials: `admin` / `password`)

* **Juice Shop:** Accessible at `http://localhost:3000`

## 💻 Code Remediation Highlights

### **1. SQL Injection Remediation (PHP)**

❌ **Vulnerable Code:**

```
$username = $_POST['username'];
$password = $_POST['password'];
$query  = "SELECT * FROM users WHERE user = '$username' AND password = '$password'";
$result = mysqli_query($conn, $query);

```

✅ **Secure Remediation (Prepared Statements):**

```
$stmt = $pdo->prepare('SELECT id, username, password_hash FROM users WHERE username = :username');
$stmt->execute(['username' => $_POST['username']]);
$user = $stmt->fetch();

if ($user && password_verify($_POST['password'], $user['password_hash'])) {
    // Authenticate session safely
}

```

### **2. SSRF Prevention (Node.js)**

❌ **Vulnerable Code:**

```
app.get('/fetch-image', async (req, res) => {
    const response = await axios.get(req.query.imageUrl);
    res.send(response.data);
});

```

✅ **Secure Remediation (DNS Lookup & Private IP Validation):**

```
const ip = require('ip');
const dns = require('dns').promises;

app.get('/fetch-image', async (req, res) => {
    try {
        const targetUrl = new URL(req.query.imageUrl);
        const addresses = await dns.lookup(targetUrl.hostname);

        // Block loopback and local network access (e.g., 127.0.0.1, 169.254.169.254)
        if (ip.isPrivate(addresses.address) || ip.isLoopback(addresses.address)) {
            return res.status(403).send("Access Denied: Restricted IP Range.");
        }

        const response = await axios.get(targetUrl.href);
        res.send(response.data);
    } catch (err) {
        res.status(400).send("Invalid Request");
    }
});

```

## 📄 Documentation & Reports

* 📄 **Full Master Report:** Refer to [`docs/DVWA_JuiceShop_Security_Assessment_Report.pdf`](./docs/) for detailed step-by-step reproduction steps and CVSS scoring breakdown.

* 📄 **Google Doc Version:** [Master Web Security Assessment Report](https://docs.google.com/document/d/1flB5CKvM-mdgUZATGegXvXma9fKjiBpkTGMkrAAauyA/edit?utm_source=gemini)

## ⚠️ Disclaimer

This project is created strictly for **educational and authorized security testing purposes only**. All security testing was executed against intentionally vulnerable web applications (`DVWA` and `OWASP Juice Shop`) hosted in a isolated local environment. Unauthorized testing against targets without prior authorization is illegal and against cybercrime laws.