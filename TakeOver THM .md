TakeOver is a free, easy-level TryHackMe room focused on subdomain enumeration and subdomain takeover techniques.  The challenge revolves around identifying a vulnerable subdomain that can be exploited due to misconfigured DNS records. 

Key Steps to Complete the Room:
Add the target domain to /etc/hosts using the provided IP (e.g., 10.10.179.117 futurevera.thm). 
Visit https://futurevera.thm in your browser (accept the security warning). 
Enumerate subdomains using tools like ffuf or gobuster:
Example with ffuf:
ffuf -H "Host: FUZZ.futurevera.thm" -u https://10.10.179.117 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -fs 0,4605

Discover subdomains such as support.futurevera.thm and blog.futurevera.thm. 
Inspect the SSL certificate of support.futurevera.thm — it reveals a Subject Alternative Name (SAN): secrethelpdesk934752.support.futurevera.thm. 
Add the SAN subdomain to /etc/hosts:
echo "10.10.179.117 secrethelpdesk934752.support.futurevera.thm" | sudo tee -a /etc/hosts

Access http://secrethelpdesk934752.support.futurevera.thm (not HTTPS) to retrieve the flag. 
Flag:
flag{beea0d6edfcee06a59b83fb50ae81b2f}

This room teaches the importance of DNS misconfigurations and certificate inspection in identifying potential takeover opportunities