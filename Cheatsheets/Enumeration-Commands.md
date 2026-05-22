# Enumeration

## Initial nmap workflow
```bash
# fast all-ports
nmap -p- --min-rate 5000 -oN nmap/allports.txt $IP

# scripts + version on found ports
nmap -sC -sV -p<ports> -oN nmap/scripts.txt $IP

# UDP top ports (slow, run in background)
sudo nmap -sU --top-ports 50 -oN nmap/udp.txt $IP
```

## Web (port 80/443/8080/etc)

### Initial check
```bash
whatweb http://$IP
curl -I http://$IP             # headers
curl http://$IP/robots.txt
```

### Directory brute
```bash
# feroxbuster - fast, recursive
feroxbuster -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt

# gobuster
gobuster dir -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt -t 50
```

### Vhost / subdomain
```bash
ffuf -u http://$IP -H "Host: FUZZ.target.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs <size-to-filter>
```

### Parameter fuzzing
```bash
ffuf -u "http://$IP/page.php?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fs <size>
```

## SMB (445)
```bash
# null session
smbclient -L //$IP -N
smbmap -H $IP
enum4linux-ng -A $IP

# with creds
smbclient -L //$IP -U user
smbmap -H $IP -u user -p pass

# connect to share
smbclient //$IP/share -U user

# list shares + perms (impacket)
impacket-smbclient user:pass@$IP
```

## RPC (135, 139)
```bash
rpcclient -U "" -N $IP
# > enumdomusers
# > enumdomgroups
# > queryuser <rid>
```

## LDAP (389, 636)
```bash
# anonymous bind
ldapsearch -x -H ldap://$IP -s base namingcontexts
ldapsearch -x -H ldap://$IP -b "DC=domain,DC=local"

# nmap scripts
nmap -p389 --script ldap-search,ldap-rootdse $IP
```

## Kerberos (88)
```bash
# user enum (no creds needed)
kerbrute userenum --dc $IP -d domain.local /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

# AS-REP roast (users without Kerberos pre-auth)
impacket-GetNPUsers domain.local/ -no-pass -usersfile users.txt -dc-ip $IP
```

## MSSQL (1433)
```bash
impacket-mssqlclient user:pass@$IP -windows-auth
# > enable_xp_cmdshell
# > xp_cmdshell whoami
```

## MySQL (3306)
```bash
mysql -h $IP -u root -p
mysql -h $IP -u root -p'' -e "show databases;"
```

## FTP (21)
```bash
ftp $IP
# try anonymous / anonymous
# look for writable dirs
```

## SSH (22)
```bash
# version sometimes hints at OS / username convention
nc -v $IP 22
# check for weak algorithms
ssh-audit $IP
```

## SNMP (161 UDP)
```bash
snmpwalk -v 2c -c public $IP
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt $IP
```

## NFS (2049)
```bash
showmount -e $IP
mkdir /mnt/nfs && mount -t nfs $IP:/share /mnt/nfs
```
