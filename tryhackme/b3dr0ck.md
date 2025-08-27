
# 🧱 b3dr0ck — TryHackMe Writeup

**Platform:** TryHackMe  
**Difficulty:** Medium  
<<<<<<< HEAD
**Date Completed:** August , 2022  
**Author:** Ali Amouz  
**Room Link:** [TryHackMe: b3dr0ck](https://tryhackme.com/room/b3dr0ck)
=======
**Date Completed:** August 2, 2025  
**Author:** Ali Amouz

>>>>>>> 903117240137412cad036ac94566e3ff37446da4

---

## 🔍 Enumeration

Initial Nmap scan revealed several open ports:

- **Port 22 (SSH):** No anonymous login.
- **Port 80 / 4040 (HTTP):** Basic web service.
- **Port 9009:** Unencrypted service, likely for certificate retrieval.
- **Port 54321:** SSL/TLS service, possibly for authentication.

---

## 🔑 Certificate-Based Access

Connected to port `9009` using netcat:

```bash
sudo nc -v 10.10.22.247 9009
```

The service allowed recovery of private keys. These were used to authenticate via port `54321` using OpenSSL. A plaintext password was revealed, which enabled SSH access as user `barney`.

---

## 🧪 Privilege Escalation (barney → fred)

Once inside, ran `linpeas` from `/tmp` to enumerate the system. Discovered `certutil` behavior that referenced JavaScript files. Reading those revealed a function `dd()` that double base64 encodes input.

Used this to decode `fred`'s password and successfully switched users:

```bash
su fred
```

---

## 🔓 Root Access via Sudo Abuse

`linpeas` showed that `fred` had passwordless sudo access to `base64` and `base32`. Found an encoded string in `pass.txt` and decoded it:

```bash
echo "<encoded>" | base32 --decode | base64 --decode
```

The result appeared to be a hash. Used an online cracking service to retrieve the plaintext password.

Switched to root:

```bash
su root
```

Retrieved `root.txt`.

---

## ✅ Summary

- Enumerated multiple services and identified certificate-based authentication.
- Exploited weak encoding logic to escalate privileges.
- Abused sudo permissions to decode and crack root credentials.

---

## 🛠️ Tools Used

- `nmap`, `nc`, `openssl`, `linpeas`, `certutil`, `base64`, `base32`, `cyberchef`

---

> _“This box was a great mix of enumeration, encoding logic, and privilege escalation. The double base64 trick and sudo abuse made it especially satisfying.”_
<<<<<<< HEAD
```
=======

>>>>>>> 903117240137412cad036ac94566e3ff37446da4
