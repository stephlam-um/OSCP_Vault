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

PASS:Buck3tH4TF0RM3!

## Recon

### Nmap
```
nmap -Pn -p- --min-rate 1000 -T4 $IP -oN allports.txt
nmap -sC -sV -p<open ports> $IP
```

### Notes
- 

## Enumeration
_Per-service findings. Web dirs, SMB shares, versions, default creds tried._

### Port XX — service
- 

## Foothold
_How you got the initial shell. Exploit used, payload, command run._

```
# command
```

**Key insight:** 

## Privilege Escalation
_What user → what user. Tool used (linpeas/winpeas/etc), vector found, exploit._

```
# command
```

**Key insight:** 

## Loot
- user.txt: 
- root.txt: 
- creds found: 
- hashes: 

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

2. 
## Links to techniques
_Link to `Techniques/` notes for anything reusable._

- [[]]

## References
_Writeups, IppSec video, exploit-db links._

- IppSec video
