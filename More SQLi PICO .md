More SQLi – picoCTF Writeup

Challenge Overview

Name: More SQLi
Author: Mubarak Mikail
Category: Web Exploitation

This challenge focuses on exploiting a login form vulnerable to SQL Injection (SQLi) to bypass authentication and retrieve the flag.

⸻

Initial Analysis

After launching the challenge instance, a basic login page is presented with:
	•	Username field
	•	Password field

No additional functionality is exposed, indicating the objective is likely authentication bypass.

⸻

Identifying the Vulnerability

Login forms commonly construct backend SQL queries like:

SELECT * FROM users 
WHERE username = 'INPUT_USERNAME' AND password = 'INPUT_PASSWORD';

If user input is not sanitized, it becomes vulnerable to SQL Injection.

⸻

Exploitation

Payload Used

' OR 1=1-- -

Injection Process
	•	Username: admin (or any value)
	•	Password: ' OR 1=1-- -

⸻

How the Injection Works

The query becomes:

SELECT * FROM users 
WHERE username = 'admin' AND password = '' OR 1=1-- -';

Breakdown:
	•	' → closes the password string
	•	OR 1=1 → always evaluates to TRUE
	•	-- → comments out the rest of the query

Result:

The WHERE clause becomes TRUE → authentication bypassed → login succeeds without valid credentials.

⸻

Using Burp Suite

To analyze the request/response:
	1.	Intercept the login request using Burp Suite Proxy
	2.	Send the request to Repeater
	3.	Modify the password field with the payload
	4.	Forward the request

⸻

Flag Discovery

After successful login:
	•	The server response contains additional data
	•	The flag is visible in the HTTP response body

This is often not rendered on the webpage but can be seen in:
	•	Burp Repeater response
	•	Raw HTTP response

⸻

Key Observations
	•	The application does not sanitize user input
	•	SQL query is directly concatenated with user input
	•	Authentication logic relies entirely on query result

⸻

Mitigation (Real-world context)

To prevent this vulnerability:
	•	Use prepared statements / parameterized queries
	•	Avoid string concatenation in SQL queries
	•	Implement input validation and escaping
	•	Use ORM frameworks where possible

⸻

Conclusion

This challenge demonstrates a fundamental SQL Injection technique:
	•	Authentication bypass using logical conditions (OR 1=1)
	•	Importance of intercepting traffic with Burp Suite
	•	Extracting hidden data from HTTP responses

⸻

Final Answer

Flag was obtained from the server response after successful SQL Injection login bypass.
:::
