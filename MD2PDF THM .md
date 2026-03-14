MD2PDF TryHackMe CTF Writeup
The MD2PDF room on TryHackMe is an easy-level CTF challenge focused on exploiting a Markdown-to-PDF conversion service to perform Server-Side Request Forgery (SSRF). 

Initial Reconnaissance
Use nmap or rustscan to identify open ports:
nmap -sC -sV 10.10.x.x

Results show:
Port 22 (SSH) – Open, but no useful access.
Port 80 (HTTP/RTSP) – Web interface for Markdown-to-PDF conversion.
Port 5000 (RTSP) – Internal web service, accessible only from localhost. 
Web Enumeration
Run Gobuster to discover hidden directories:
gobuster dir -u http://10.10.x.x -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt

Found directories:
/admin – Returns 403 Forbidden.
/convert – Accepts POST requests to convert Markdown to PDF. 
Exploitation: SSRF via HTML Injection
The web app uses wkhtmltopdf (version 0.12.5) to render HTML/PDF. 
The /convert endpoint accepts Markdown input, which can include raw HTML.
The /admin page is only accessible from localhost:5000, but SSRF allows the server to make requests on behalf of the attacker. 
Payload:

<iframe src="http://localhost:5000/admin" width="1000" height="1000"></iframe>

Paste this into the Markdown input field and click Convert to PDF.
The server processes the iframe, making a request to http://localhost:5000/admin internally.
The resulting PDF contains the response — the flag. 
Flag
flag{1f4a2b6ffeaf4707c43885d704eaee4b}
Summary
This challenge demonstrates how SSRF can be exploited through HTML injection in a PDF generator, bypassing localhost restrictions.  Always validate and sanitize user input, especially when rendering content server-side.