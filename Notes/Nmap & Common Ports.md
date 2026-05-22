> [Ultimate lookup cheatsheet ](https://www.stationx.net/common-ports-cheat-sheet/)

## 1. Common Ports Lookup Table

|**Port(s)**|**Protocol**|**Default Service / Usage**|**OS Hint / Note**|
|---|---|---|---|
|**20/21 (TCP)**|FTP|File Transfer Protocol|Check for Anonymous login (`get file_name`)|
|**22 (TCP)**|SSH|Secure Shell|**Most likely Linux** (but can be Windows)|
|**23 (TCP)**|Telnet|Unencrypted Text Communications|Legacy/Insecure|
|**25 (TCP)**|SMTP|Simple Mail Transfer Protocol|Mail routing|
|**80 (TCP)**|HTTP|Hypertext Transfer Protocol|Unencrypted web traffic|
|**88 (TCP/UDP)**|Kerberos|Authentication Protocol|Indicates an Active Directory environment|
|**161 (TCP/UDP)**|SNMP|Simple Network Management Protocol|_Fun fact: tool `onesixtyone` is named after this port!_|
|**389 (TCP/UDP)**|LDAP|Lightweight Directory Access Protocol|Often tied to Active Directory|
|**443 (TCP)**|HTTPS|HTTP over SSL/TLS|Encrypted web traffic|
|**445 (TCP)**|SMB|Server Message Block|Fileshares. **Indicates a Windows host** (or Samba)|
|**3389 (TCP)**|RDP|Remote Desktop Protocol|**Indicates a Windows host**|

> 📌 **Port Range Note:** Ports **0 - 1023** are Well-Known/Common system ports. Others are registered or dynamic.
> 
> ⚠️ **Service Assumption Warning:** Until you instruct Nmap to interact with a service and tease out identifying information, a port could be running something else entirely. Use `netcat` or `socat` (stronger) for manual exact service scanning.

---

## 2. Nmap Scanning Commands

### workflow 
1. scan top 1000 ports with sV first
2. Full scan with -p-, at the same time enum the found top ports with **sC** and **banner grabbing** 



### Good Practise (keep record)

```bash
nmap -sC -p 22,80 -oA nibbles_script_scan 10.129.42.190
```

### Full Comprehensive Scan

```
nmap -sV -sC -p- 10.129.42.253
```

- `-sV`: Service version detection.
    
- `-sC`: Runs default enumeration scripts.
    
- `-p-`: Scans **all 65,535 ports** (instead of just the top 1000).
    

### Fast Banner Grabbing


``` bash
nmap -sV --script=banner <IP>
```

even faster
```bash
nc -nv 10.129.42.190 22
```

---

## 3. Service Enumeration & Exploitation

### FTP (File Transfer Protocol)

If anonymous login is allowed, log in and download interesting files:

Bash

```
get file_name
```

### SMB (Server Message Block - Port 445)

**OS Discovery via Nmap:**

Bash

```
# Aggressive scan (includes OS detection, script scanning, traceroute)
nmap -A -p445 10.10.10.40

# Specific OS discovery script
nmap --script smb-os-discovery.nse -p445 10.10.10.40
```

**Connecting to Shares:**

Bash

```
# Null Session (No password, List shares)
smbclient -N -L \\\\10.129.42.253

# Connect to a specific share anonymously
smbclient \\\\10.129.42.253\\users

# Connect as a specific user
smbclient -U bob \\\\10.129.42.253\\users
```

> 💡 **Common Default Password Try:** `Welcome1`

### SNMP (Port 161 UDP)

Used to check server stats and system details. Sometimes the credentials match the community string exactly.

**Using snmpwalk:**

Bash

```
# Query specific MIB (e.g., system name) using community string 'public'
snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0

# General walk using community string 'private'
snmpwalk -v 2c -c private 10.129.42.253
```

**Brute Forcing Community Strings:**

Bash

```
onesixtyone -c dict.txt 10.129.42.254
```