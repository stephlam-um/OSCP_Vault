```bash 
sudo openvpn user.ovpn
```
the .ovpn file specifies which addr to connect to

``` bash
ifconfig
netstat -rn
```

ifconfig shows the current network connection, `tun` adapter is shown if vpn is successfully connected.

netstat to show which networks can we access to via the vpn.

