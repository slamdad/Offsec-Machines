TryHackMe — Escape the Corridor Write-Up

Room: Escape the Corridor

Platform: TryHackMe

⸻

1. Enumeration

Nmap Scan

Initial scan:

sudo nmap 10.82.189.252

Result:
	•	Port 80 open
	•	Service: HTTP

Full scan:

sudo nmap -A -p 80 10.82.189.252

Key findings:
	•	Web server: Werkzeug/2.0.3 Python/3.10.2
	•	Title: Corridor
	•	OS: Linux (4.x / 5.x)

⸻

2. Web Analysis

Browsing to the site revealed multiple “doors”.
Each door linked to a hexadecimal string.

These values resembled MD5 hashes.

Example hashes discovered:

c4ca4238a0b923820dcc509a6f75849b
c81e728d9d4c2f636f067f89cc14862c
eccbc87e4b5ce2fe28308fd9f2a7baf3
...

These were saved into a file:

nano hashes.txt


⸻

3. Hash Identification

These hashes matched:

MD5(1)
MD5(2)
MD5(3)
...
MD5(13)

This indicated:
	•	Each door corresponds to MD5 of numbers 1–13

⸻

4. Exploiting IDOR Logic

The challenge hinted at:

Examine hexadecimal values and try accessing unexpected locations.

Since doors existed for numbers 1–13, the next logical test was:

What about 0?

Generate MD5 of 0:

echo -n 0 | md5sum

Output:

cfcd208495d565ef66e7dff9f98764da


⸻

5. Direct Object Reference Manipulation

Manually navigate to:

http://<TARGET>/cfcd208495d565ef66e7dff9f98764da

This directory existed.

This confirmed an IDOR vulnerability:
	•	Server did not validate allowed IDs.
	•	It allowed access to hidden resource /0.

⸻

6. Flag Found

Accessing that path revealed the flag:

flag{2477ef02448ad9156661ac40a6b8862e}


⸻

7. Vulnerability Explanation

This is a classic Insecure Direct Object Reference (IDOR):
	•	Application exposes object identifiers directly.
	•	No access control validation.
	•	Predictable pattern (MD5 of integers).
	•	Attacker enumerates possible IDs.

⸻

8. Key Takeaways
	•	Always test logical boundaries (0, negatives, high values).
	•	Hashes are often deterministic and reversible.
	•	If you see patterns → assume enumeration is possible.
	•	IDOR vulnerabilities rely on missing authorization checks.

⸻

Tools Used
	•	nmap
	•	md5sum
	•	Browser manual testing

⸻

Attack Flow Summary
	1.	Enumerate open ports
	2.	Discover hash-based routing
	3.	Identify MD5 pattern
	4.	Generate MD5(0)
	5.	Access hidden endpoint
	6.	Retrieve flag

⸻
