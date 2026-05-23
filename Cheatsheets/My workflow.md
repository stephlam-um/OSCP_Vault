start my custom recon x start up script
```bash
newbox lame 10.10.10.3
tmux attach -t lame
```
during work
```bash
curl http://$IP
gobuster dir -u http://$IP -w /usr/share/wordlists/dirb/common.txt

# scans
nmap $IP  | tee file.txt

# exploits you downloaded
cd exploits && wget https://...

# loot (hashes, creds, flags)
echo "user.txt: abc123..." >> loot/flags.txt

# screenshots
win + S
```
Finishes
```bash
tmux kill-session -t lame
or 
tmux kill-server
```