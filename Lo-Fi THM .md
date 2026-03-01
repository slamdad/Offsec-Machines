TryHackMe — Lo-Fi Write-Up

Platform: TryHackMe

Reference terminal session:  ￼

⸻

1. Enumeration

Nmap Scan

Initial scan:

sudo nmap 10.82.176.17

Result:
	•	22/tcp — SSH
	•	80/tcp — HTTP

Only a web service and SSH were exposed.

⸻

2. Directory Enumeration

Used Gobuster:

gobuster dir -u http://10.82.176.17 -w /usr/share/wordlists/dirb/common.txt

Findings:
	•	index.php
	•	server-status (403)
	•	No obvious admin panels

This suggested the vulnerability was likely in index.php.

⸻

3. Manual Web Testing

Upon browsing the website, parameters were identified.

The key discovery:

A GET parameter (e.g., page=) appeared to control file inclusion.

Example testing:

http://10.82.176.17/index.php?page=...


⸻

4. Testing for Path Traversal

Tested basic traversal:

?page=../../../../etc/passwd

It successfully displayed /etc/passwd.

This confirmed a Local File Inclusion (LFI) vulnerability via path traversal.

⸻

5. Reading Sensitive Files

Since /etc/passwd worked, the application was directly including file paths without sanitization.

The flag was retrieved by reading the appropriate file using traversal.

Flag obtained:

flag{e4478e0eab69bd642b8238765dcb7d18}


⸻

6. Vulnerability Explanation

This is a classic:

Local File Inclusion (LFI) / Path Traversal

Root cause:
	•	User-controlled input passed into file include function.
	•	No sanitization.
	•	No directory restriction.

Example vulnerable PHP logic:

include($_GET['page']);

Without validation, attackers can traverse directories:

../../../../etc/passwd


⸻

7. Attack Flow Summary
	1.	Scan open ports
	2.	Enumerate directories
	3.	Identify page= parameter
	4.	Test path traversal
	5.	Read /etc/passwd
	6.	Read flag file
	7.	Capture flag

⸻

Key Takeaways
	•	Always test parameters for traversal (../)
	•	If /etc/passwd loads → immediate LFI
	•	Simple boxes often rely on input validation flaws
	•	Gobuster helps confirm limited surface area

⸻
