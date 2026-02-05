

# CPENT – OSINT & Attack Surface Mapping Lab Writeup

---

## Exercise 1: Collecting Open-source Intelligence (OSINT) on Target's Domain Name

### Scenario

This exercise focuses on collecting OSINT related to the target’s domain name using public sources, in order to discover WHOIS data, DNS records, IP addresses, subdomains, and other infrastructure components.

### Target

* Domain: `certifiedhacker.com`

---

### Task 1: Performing Whois Lookups

#### Goal

Obtain WHOIS data for `certifiedhacker.com` from multiple sources and identify the reverse DNS (rDNS) entry for its IP address.

#### Tools

* `whois` (CLI)
* `nmap` NSE whois scripts
* Web: `https://www.whois.com/whois/`

#### Commands Used

```bash
# Become root
sudo su
# WHOIS via CLI
whois certifiedhacker.com

# WHOIS via Nmap NSE
nmap -sn --script whois-* www.certifiedhacker.com
```

#### Web WHOIS

1. Open a browser in Parrot.
2. Navigate to `https://www.whois.com/whois/`.
3. Search for `certifiedhacker.com`.
4. Review registrant, nameservers, IP, and rDNS.

#### Key Evidence

* IP address: `162.241.216.11`
* rDNS entry obtained from WHOIS / reverse lookup.

#### Question 3.1.1.1

On the Parrot Security machine, utilize the whois command, Nmap, and the Whois website to conduct a Whois lookup on `certifiedhacker.com`. Record the rDNS entry for `162.241.216.11`.

**Answer:** `box5331.bluehost.com`

---

### Task 2: Finding DNS Records for the Domain

#### Goal

Identify DNS records and reverse mappings for the target’s IP range and main domain.

#### Tools

* `dnsrecon`
* `dig`

#### Commands Used

```bash
# DNS records for the domain
dnsrecon -d certifiedhacker.com

# Reverse lookup over IP range
dnsrecon -r 162.241.216.0-162.241.216.255

# A / other records via dig
dig certifiedhacker.com

# Reverse DNS lookup for IP
dig -x 162.241.216.11
```

#### Key Evidence

From `dnsrecon` output:

* Name server: `ns1.bluehost.com`
* IP for `ns1.bluehost.com`: `162.159.24.80`

#### Question 3.1.2.1

On the Parrot Security machine, use the dnsrecon tool to retrieve the DNS records for `certifiedhacker.com`. Identify and enter the IP address of the name server `ns1.bluehost.com`.

**Answer:** `162.159.24.80`

---

### Task 3: Finding Domain and Sub-domains of the Target

#### Goal

Enumerate subdomains of `certifiedhacker.com` using multiple tools and sources.

#### Tools

* `subbrute`
* `nmap` (`dns-brute` script)
* `Sublist3r`
* Netcraft Site Report: `https://sitereport.netcraft.com`
* DNSDumpster: `https://dnsdumpster.com`

#### Commands Used

```bash
# Subbrute
cd subbrute
./subbrute.py certifiedhacker.com

# Nmap DNS brute-force
cd /home/pentester
nmap --script dns-brute certifiedhacker.com

# Sublist3r
cd Sublist3r
python3 sublist3r.py -d certifiedhacker.com -p 80
```

#### Web Tools

1. Netcraft Site Report → look up `https://certifiedhacker.com`.
2. Click the `certifiedhacker.com` domain link under “Network” to see subdomains.
3. DNSDumpster → enter `eccouncil.org` to view DNS and host information.

#### Key Evidence

* Nmap DNS brute-force discovered a total of **7 hostnames**.

#### Question 3.1.3.1

On the Parrot Security machine, utilize subbrute.py, Nmap, sublist3r.py, Netcraft Site Report, and DNSDumpster to identify the domain and subdomains of `certifiedhacker.com`. Record the total number of hostnames discovered after performing DNS brute-forcing with Nmap.

**Answer:** `7`

---

### Task 4: Using Scripts and AI Tools to Combine and Automate OSINT

#### Goal

Use an AI tool (ChatGPT) to generate a shell script that automates OSINT tasks on a target domain and apply it to `certifiedhacker.com`.

#### Tools

* Browser + ChatGPT
* `dos2unix`
* `bash` / shell
* `dnsmap`, `urlcrazy`, `whois`, `dnsrecon`, `dig`

#### Process

1. In browser, open ChatGPT: `https://chatgpt.com/`.

2. Prompt example (given in lab):

   ```text
   Generate a shell script to perform enumeration on the target domain (e.g., microsoft.com) to extract subdomains using the dnsmap tool, parallel domains using the urlcrazy -p command, WHOIS lookup data using the whois command, DNS records using the dnsrecon tool, and check for zone transfers using the dig command. The command should organize and display the results in clearly labeled sections for each type of information.
   ```

3. Save the generated script as `Domain_OSINT.sh` under `/home/pentester/Scripts/Module 3/` or use the provided file.

4. Edit the script and replace `microsoft.com` with `certifiedhacker.com`.

#### Commands Used

```bash
cd /home/pentester/Scripts/Module\ 3
sudo su
dos2unix Domain_OSINT.sh
chmod +x Domain_OSINT.sh
./Domain_OSINT.sh
```

#### Output

* Script runs:

  * Subdomain enumeration (`dnsmap`)
  * Parallel domains (`urlcrazy`)
  * WHOIS (`whois`)
  * DNS records (`dnsrecon`)
  * Zone transfer checks (`dig`)
* Results printed in labeled sections and saved to `domain_info.txt`.

#### Question 3.1.4.1

On the Parrot Security machine, leverage scripts and AI tools to integrate and automate OSINT. Execute the `Domain_OSINT.sh` script located at `/home/pentester/Scripts/Module 3`, and record the Registrar IANA ID obtained from the Whois lookup.

**Answer:** `2`

---

## Exercise 2: Collecting OSINT about Target on the Web

### Scenario

This exercise focuses on gathering information about the target from general web sources using search operators and Google Hacking Database (GHDB), and understanding public cloud storage exposure.

---

### Task 1: Searching for Target's Information using Advanced Search Operators

#### Goal

Use Google advanced search operators and GHDB to refine search results for `certifiedhacker.com` and identify a specific GHDB entry.

#### Web Queries

In browser (Google):

```text
site:certifiedhacker.com -site:www.certifiedhacker.com
site:certifiedhacker.com inurl:Support
site:certifiedhacker.com allintitle:login
```

Then:

1. Go to `https://www.exploit-db.com/google-hacking-database`.

2. Search GHDB for the dork:

   ```text
   inurl:home.htm intitle:1766
   ```

3. Note the corresponding GHDB-ID.

#### Question 3.2.1.1

In GHDB, search for `inurl:home.htm intitle:1766` and record the GHDB-ID of the corresponding Google dork.

**Answer:** `8450`

---

### Task 2: Searching for Public Cloud Buckets, Blobs, Files, and Spaces

#### Goal

Explore `buckets.grayhatwarfare.com` to understand how public cloud buckets may expose data.

#### Steps

1. Visit `https://buckets.grayhatwarfare.com/`.
2. Review available options (e.g., AWS).
3. Note that advanced search and filtering require registration.

*(No graded question in your text for this task.)*

---

## Exercise 3: Collecting OSINT on Target's Employees

### Scenario

This exercise focuses on identifying employees’ public presence and social footprint using OSINT tools like Sherlock, Maigret, and Cylect.io.

---

### Task 1: Collecting OSINT from Social Media using Sherlock

#### Goal

Use Sherlock to find public usernames and profiles for a given name.

#### Tools

* `sherlock` (Python-based OSINT tool)

#### Commands Used

Example in lab (Elon Musk):

```bash
sudo su
sherlock "Elon Musk"
```

For the question’s target:

```bash
sherlock "Satyam Nadella"
```

#### Output

* List of social media profiles and URLs matching the searched name.

#### Question 3.3.1.1

On Parrot Security machine, utilize Sherlock to collect publicly available personal information about Elon Musk. Write the command used to gather information of Satyam Nadella.

**Answer:**

```bash
sherlock "Satyam Nadella"
```

---

### Task 2: Social Media Content Analysis using Maigret

#### Goal

Use Maigret to build a detailed OSINT profile and generate a report.

#### Tools

* `maigret` (Python-based OSINT tool)

#### Commands Used

Example (Elon Musk):

```bash
sudo su
maigret "Elon Musk" --html
```

For the question’s subject (Satyam Nadella), same pattern:

```bash
maigret "Satyam Nadella" --html
```

* Maigret generates an HTML report in:

  * `/home/pentester/reports/report_<name>_plain.html`

#### Key Evidence

* In the report for Satyam Nadella, under the “Studfile” section, one URL is listed.

#### Question 3.3.2.1

On Parrot Security machine, utilize the Maigret tool to conduct social media content analysis and investigate online profiles associated with Satyam Nadella. Record the URL displayed under “Studfile”.

**Answer:**
`https://studfile.net/users/Satyam%20Nadella/`

---

### Task 3: Searching for People Information using Cylect.io

#### Goal

Leverage Cylect.io to discover people search tools and analyze public information.

#### Steps

1. Open `https://cylect.io/`.
2. Scroll down, enter a target person’s name.
3. Choose **People**.
4. Under “People Search” tools, select **WebMii**.
5. Review WebMii search result page.
6. Back in Cylect.io, click on **Spokeo** and review results.

#### Question 3.3.3.1

On Parrot Security machine, utilize `https://cylect.io` to search for and analyze publicly available information about individuals. Identify the option that is selected under People Search in this lab.

**Answer:** `Webmii`

---

## Exercise 4: Collecting OSINT using Automation Tools

### Scenario

This exercise uses automation frameworks and recon platforms to gather OSINT efficiently: Maltego, SpiderFoot, and reNgine.

---

### Task 1: Gathering OSINT on a Target using Maltego

#### Goal

Use Maltego CE to visually map relationships for `certifiedhacker.com` and extract DNS, IP, location, and WHOIS-based information.

#### Tools

* Maltego Community Edition

#### Process (High-Level)

1. Start Maltego (`maltego` as root).
2. Complete activation using Maltego ID.
3. Create a new graph.
4. Drag **Website** entity to the graph and change it to `www.certifiedhacker.com`.
5. Run transforms:

   * `To Domains [DNS]`
   * `To DNS Name [Using Name Schema dictionary]`
   * `To DNS Name - SOA`
   * `To DNS Name - MX`
   * `To DNS Name - NS`
   * `To IP Address [DNS]`
   * `To Location [city, country]`
   * `To Entities from WHOIS [IBM Watson]`

#### Key Evidence

* Number of name servers associated with `certifiedhacker.com`.
* IP address resolved for the website.

#### Question 3.4.1.1

Use Maltego to gather information about `certifiedhacker.com`. Identify the number of name servers associated with `certifiedhacker.com` using the `To DNS Name - NS (Name Server)` transform and record the result.

**Answer:** `2`

#### Question 3.4.1.2

Use Maltego to gather information about the `certifiedhacker.com` website. Record the IP address of `certifiedhacker.com` obtained from the `To IP Address [DNS]` transform.

**Answer:** `162.241.216.11`

---

### Task 2: Gathering OSINT using SpiderFoot

#### Goal

Automate OSINT collection against `eccouncil.org`.

#### Tools

* SpiderFoot (`sf.py`)

#### Commands Used

```bash
cd ~
git clone https://github.com/smicallef/spiderfoot.git
cd spiderfoot
pip install -r requirements.txt

# Run SpiderFoot web UI
python3 sf.py -l 127.0.0.1:5001
```

Then in browser:

1. Open `http://127.0.0.1:5001`.
2. Click **New Scan**.
3. Mode: **Footprint**.
4. Scan name: `ECC`.
5. Target: `eccouncil.org`.
6. Run Scan and review results.

#### Key Evidence

* Domain registrar identified for `eccouncil.org`.

#### Question 3.4.2.1

On the Parrot Security machine, use SpiderFoot to gather open-source intelligence on `eccouncil.org`. Record the Domain Registrar name identified during the scan.

**Answer:** `Network Solutions, LLC`

---

### Task 3: Performing Web Reconnaissance using reNgine

#### Goal

Perform a web recon scan of `certifiedhacker.com`, including subdomains, endpoints, and vulnerabilities.

#### Tools

* reNgine (web-based recon framework)

#### Process

1. Open browser → `http://127.0.0.1`.
2. Login: user `root`, password `toor`.
3. Complete initial setup (Project name, etc.).
4. Go to **Targets → Add Targets**, add `certifiedhacker.com`.
5. Click **Initiate Scan**.
6. Choose scan engine (e.g., “reNgine Recommended”).
7. Proceed through subdomain import and URL scope; start scan.
8. After some time, click **View Results** to see subdomains, endpoints, technologies, and vulnerabilities.

#### Key Evidence

* Detection of Nginx version on `demo.certifiedhacker.com`.

#### Question 3.4.3.1

On the Parrot Security machine, utilize reNgine to conduct a comprehensive scan to identify subdomains, endpoints, and vulnerabilities associated with `certifiedhacker.com`. Record the Nginx version detected on the `demo.certifiedhacker.com` subdomain.

**Answer:** *(Fill with the exact version shown in your reNgine scan output, e.g. `nginx/1.x.x`.)*

---

## Exercise 5: Identifying and Mapping Attack Surface

### Scenario

This exercise uses various network-scanning and scripting tools to discover devices, open ports, services, OS fingerprints, and to automate scanning, building a target database.

---

### Task 1: Discovering Network Devices with Netdiscover and Nmap

#### Goal

Identify live hosts and open ports on the local subnet using ARP-based discovery and TCP scans.

#### Tools

* `netdiscover`
* `nmap`

#### Commands Used

```bash
# Show netdiscover help
netdiscover -h

# Passive discovery on eth0
sudo netdiscover -i eth0 -p

# Generate traffic with ping sweep
nmap -sn 192.168.0.0/24

# TCP scan over subnet
nmap -sT 192.168.0.0/24
```

#### Key Evidence

* For host `192.168.0.51`, Nmap TCP scan shows **4** open ports.

#### Question 3.5.1.1

Use Netdiscover and Nmap to identify network devices. Perform a TCP scan with Nmap to determine the number of open ports on the machine with IP address `192.168.0.51`.

**Answer:** `4`

---

### Task 2: Network Scanning using DMitry

#### Goal

Perform port scanning and banner grabbing using DMitry.

#### Tools

* `dmitry`

#### Commands Used

(Example for 192.168.0.7 in lab; adapted for exam IP):

```bash
sudo nmap -sn 192.168.0.0/24    # Identify targets

# Port scan
dmitry -pf 192.168.0.51

# Port scan + banner grab
dmitry -pb 192.168.0.51
```

#### Key Evidence

* DMitry reported **3** open ports on `192.168.0.51`.

#### Question 3.5.2.1

Use the DMitry tool to perform a port scan on the `192.168.0.51` host and record the number of open ports detected.

**Answer:** `3`

---

### Task 3: Scanning and Scripting with hping3

#### Goal

Use hping3 to craft packets, run simple scripts, capture traffic, and perform a port scan.

#### Tools

* `hping3`
* `tcpdump`
* `wireshark`

#### Samples of Commands Used

```bash
# Start hping3 shell
sudo hping3

# Send single ICMP echo request
hping send {ip(daddr=172.19.19.7)+icmp(type=8,code=0)}

# TTL loop (script inside hping3)
foreach i [list 5 6 7 8 9 10] {hping send "ip(daddr=172.19.19.7,ttl=$i)+icmp(type=8,code=0)"}

# Port scan on target
sudo hping3 --scan known 172.19.19.7 -S
sudo hping3 --scan '0-3000' 172.19.19.7 -S

# File via ICMP
sudo hping3 127.0.0.1 --listen signature --safe --icmp
sudo hping3 127.0.0.1 --icmp -d 100 --sign signature --file /etc/passwd
```

For the graded question:

```bash
sudo hping3 --scan '0-3000' 172.19.19.51 -S
```

#### Key Evidence

* hping3 scan on `172.19.19.51` found **4** open ports.

#### Question 3.5.3.1

Use hping3 to perform various network tasks and scan `172.19.19.51` to identify open ports. Record the total number of discovered open ports.

**Answer:** `4`

---

### Task 4: Automating Penetration Testing Tasks Using Bash Scripting

#### Goal

Use a Bash script (`pentest.sh`) to automate:

* Live host discovery
* FTP port detection
* FTP brute-force with Hydra
* FTP login with cracked credentials

#### Tools

* `bash` scripting
* `nmap`
* `hydra`
* `ftp`

#### Commands Used

```bash
bash pentest.sh
# Then follow script prompts:
# 1) IP range, e.g. 172.19.19.7-50
# 2) Target IP for Hydra, e.g. 172.19.19.9
# 3) FTP login IP, e.g. 172.19.19.9
```

Script key operations:

* `nmap -sP` to identify live hosts.
* Parse output (`out.txt`, `out1.txt`, `open.txt`, `final.txt`, `ftp.txt`).
* `nmap -p 21` to detect FTP open.
* `hydra` with username & password wordlists.
* `ftp` login with cracked credentials.

#### Key Evidence

* Hydra cracked multiple FTP credentials, including user `shiela`.
* Password for `shiela` was `sushie`.

#### Question 3.5.4.1

Use a Bash script to run an Nmap scan, perform a brute-force attack with Hydra, and authenticate via FTP. Enter the cracked password for the user `shiela`.

**Answer:** `sushie`

---

### Task 5: Using Workspaces and db_nmap (Metasploit)

#### Goal

Create a workspace, run Nmap scans through Metasploit (`db_nmap`), store results in the database, and use them as input to modules.

#### Tools

* `postgresql`
* Metasploit Framework (`msfconsole`)
* `db_nmap`

#### Commands Used

```bash
sudo service postgresql start
sudo msfdb init
sudo msfconsole

# Check DB
db_status

# Workspaces
workspace -h
workspace -a LPT

# Scans
db_nmap -sP 192.168.0.0/24
db_nmap -sS 192.168.0.2-70
db_nmap -sV 192.168.0.2-70
db_nmap -A 192.168.0.2-70

# View data
services
hosts
hosts -c address,os_flavor
hosts -c address,os_flavor -S Linux

# Use hosts as RHOSTS
use auxiliary/scanner/portscan/tcp
hosts -c address,os_flavor -S Linux -R
show options
run
```

#### Key Evidence

* Apache version on target `192.168.0.51` identified as `2.4.52`.

#### Question 3.5.5.1

Create workspaces and use `db_nmap` in Metasploit for network scanning. Identify and document the version of Apache running on the target machine at `192.168.0.51`.

**Answer:** `2.4.52`

---

### Task 6: Scanning and Building a Target Database

#### Goal

Use Nmap output (XML) to build an initial target database including host, OS, ports, services, vulnerabilities, and priorities.

#### Tools

* `nmap`
* `xsltproc`

#### Commands Used

Example flow:

```bash
# Full scan with output to XML
nmap -sP 192.168.0.0/24 -oX scan.xml
# or combine -sS, -sV, -A as required, also with -oX scan.xml

# Convert XML to HTML
xsltproc -o ~/scanresults.html /usr/share/nmap/nmap.xsl scan.xml
```

#### Database Fields

* Host/IP
* OS
* Ports
* Services
* Vulnerabilities
* Exploit
* Notes
* Priority

From scan analysis (per lab):

* Host `192.168.0.20` had **10** open ports.

#### Question 3.5.6.1

Use Nmap to scan `192.168.0.0/24`, analyze the output, and compile an initial target database. Determine and document the number of open ports on the host `192.168.0.20`.

**Answer:** `10`

---

### Task 7: OS Fingerprinting with Nmap

#### Goal

Identify operating systems of target hosts via Nmap OS detection.

#### Tools

* `nmap`
* `wireshark` (optional packet capture)

#### Commands Used

```bash
sudo nmap -O 192.168.0.51
```

#### Key Evidence

* OS fingerprint for `192.168.0.51` indicates **Linux**.

#### Question 3.5.7.1

Conduct OS fingerprinting on the target machine at IP `192.168.0.51` and determine the operating system name.

**Answer:** `Linux`

---

### Task 8: Automating Network Scanning using AI-Powered Tools

#### Goal

Use an AI tool (Microsoft Copilot) to generate a Python script that performs multi-technique host discovery and port scanning, with OS and version detection.

#### Tools

* Microsoft Copilot (web)
* Python 3
* Script: `Attack_Surface_Mapping.py`

#### Process

1. Open `https://copilot.microsoft.com`.
2. Prompt Copilot to generate a Python script that:

   * Prompts for target URL / IP.
   * Performs ICMP, ARP, UDP, TCP pings.
   * Runs TCP connect, UDP, half-open, Xmas, SCTP scans.
   * Performs service/version detection.
   * Saves results to a text file.
3. Save the script as `Attack_Surface_Mapping.py` under `/home/pentester/Scripts/Module 3/`.

#### Commands Used

```bash
sudo su
cd /home/pentester/Scripts/Module\ 3
python3 Attack_Surface_Mapping.py
# When prompted, enter:
192.168.0.0/24
```

After completion, open the generated results file and review hosts, ports, services, OS banners.

#### Key Evidence

* For `192.168.0.20`, OS identified as **Windows**.

#### Question 3.5.8.1

Utilize AI-powered tools to scan `192.168.0.0/24` and determine the OS of the target machine `192.168.0.20`.

**Answer:** `Windows`

---

## How to Convert This into a Pentest Report Section

For a real penetration test report:

* Use **Exercises 1–4** as your “External OSINT & Web Footprinting” methodology and evidence.
* Use **Exercise 5** as your “Network Reconnaissance & Attack Surface Mapping” section.
* Move all **Questions & Answers** into:

  * An **Appendix: Lab Validation / Evidence** section, or
  * An internal “trainer notes” section.
* Turn the “Key Evidence” parts into:

  * **Findings / Observations**
  * With severity levels if appropriate (e.g., weak FTP credentials, exposed services, etc.).

