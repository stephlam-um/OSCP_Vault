

### 🌀 Gobuster (Directory & DNS Brute-Forcing)

Bash

```
# Directory scan (Standard)
gobuster dir -u http://10.129.3.6/ -w /usr/share/wordlists/dirb/common.txt

# Directory scan + extensions + hide specific status codes (Essential)
gobuster dir -u http://10.10.10.121/ -w [wordlist] -x php,txt,html -b 403,404

# Subdomain / DNS brute-forcing
gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

> **Status Codes:** `2xx` Success | `3xx` Redirection | `4xx` Client Error (`403` Forbidden / `404` Not Found)

---

### 🌐 WhatWeb (Web Tech Fingerprinting)

Bash

```
whatweb 10.10.10.0            # Quick scan
whatweb --no-errors 10.10.10.0/24  # Stealth network scan
whatweb -a 3 http://10.10.10.121   # Aggressive fingerprinting
```

- **Certificates:** Check SSL/TLS certs via WhatWeb for internal emails, subdomains, and contact info (crucial for internal networks where OSINT/Google fails).
    

---

### 💻 cUrl (Web Requests & Interaction)

Bash

```
# Send POST login data (Follows redirects)
curl -L -X POST https://example.com/login -d "username=admin" -d "password=123"

# Inspect Response (Headers only / Headers + Body)
curl -I http://10.10.10.121
curl -i http://10.10.10.121

# Connection tweaks (Ignore self-signed SSL / Send Session Cookie)
curl -k https://10.10.10.121
curl -H "Cookie: session=123" http://10.10.10.121/dashboard
```

---

### ➕ Missed Essentials (Quick Vuln Scans)

Bash

```
nikto -h http://10.10.10.121                     # Quick web vulnerability scan
nmap -p80,443 --script="http-*" 10.10.10.121     # Nmap HTTP recon scripts
```

## Fuzz
```bash
ffuf -u http://<target-ip>/FUZZ -w /usr/share/wordlists/dirb/common.txt
```