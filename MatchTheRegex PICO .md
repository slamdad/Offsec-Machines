MatchTheRegex – picoCTF 2023 Writeup

Challenge Type
Web Exploitation (Client-Side Validation)

⸻

Objective

Identify the hidden regex pattern in the client-side JavaScript and provide an input that satisfies it to obtain the flag.

⸻

Step 1: Analyze the Web Page

After launching the challenge instance, the page presents:
	•	A simple input field
	•	A submit button

No visible hints are provided, so the next step is to inspect the source code.

⸻

Step 2: Inspect Source Code

View the page source or open Developer Tools → Sources.

Locate the JavaScript function responsible for handling input submission (commonly named send_request()).

Inside the code, a commented regex appears:

^p.....F!?


⸻

Step 3: Understand the Regex

Breakdown of the pattern:
	•	^ → Start of string
	•	p → Must begin with lowercase p
	•	..... → Exactly 5 arbitrary characters
	•	F → Must end with uppercase F
	•	!? → Optional ! or ?

⸻

Step 4: Craft Valid Input

The total structure:

p + 5 characters + F + optional (! or ?)

Examples of valid inputs:
	•	picoCTF
	•	p12345F!
	•	pabcdeF?

The simplest valid input:

picoCTF


⸻

Step 5: Submit Input

Enter the valid string into the input field and submit.

If it matches the regex, the JavaScript logic triggers a response (usually an alert or API call).

⸻

Step 6: Retrieve the Flag

Upon successful submission, the flag is displayed.

Example:

picoCTF{succ3ssfully_matchtheregex_f89ea585}

Note: The hash portion may vary per instance.

⸻

Key Takeaways
	•	Client-side validation is not secure and can be bypassed or analyzed easily
	•	Always inspect JavaScript for hidden logic in web challenges
	•	Regex understanding is essential for web exploitation tasks

⸻

Final Answer Format

picoCTF{succ3ssfully_matchtheregex_<dynamic_hash>}