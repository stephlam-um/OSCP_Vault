---
aliases:
  - learning notes
---
Domain joined = NTLM hash = AD attack 

OS Kernel difference? 
ramifications

running services: Flaws have been discovered in many common services such as Nagios, Exim, Samba, ProFTPd, etc. Public exploit PoCs exist for many of them, such as CVE-2016-9566, a local privilege escalation flaw in Nagios Core < 4.2.4.

why do we list current processes during recon? to see what root controlled processes are misconfigured so we can exploit? 
ps aux | grep root 

whgat does a user's ssh key look like
t the minimum, check the ARP cache to see what other hosts are being accessed and cross-reference these against any useable SSH private keys.