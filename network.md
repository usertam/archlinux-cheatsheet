# Configuring network

## Setup wireless network in archiso
The archiso has iwd.service enabled by default, and should be auto-started on boot. \
If you want to manually restart it anyway, do:
```sh
systemctl restart iwd.service
```

### Connect to a network using iwctl
Most iwd commands are self-explanatory, here are a few you might need connecting to a new network: 
```
$ iwctl
[iwd]# station list
[iwd]# station wlan0 show
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
[iwd]# station wlan0 connect NETWORK
```

## Setup systemd network services

### Create systemd-networkd configs
User-created config files belong in ```/etc/systemd/network/```. \
Below are some examples. 

```30-wired.network```: 
```
[Match]
Type=ether
Name=en*

[Network]
DHCP=yes
```

```35-wireless.network```: 
```
[Match]
Type=wlan

[Network]
DHCP=yes
```

```20-static-wireless.network```: 
```
[Match]
Name=wlan0
SSID=NETWORK

[Network]
Address=192.168.1.111/24
Gateway=192.168.1.1
DNS=192.168.1.1
```

### Start and enable the services and link resolv.conf
```sh
systemctl start systemd-networkd systemd-resolved
systemctl enable systemd-networkd systemd-resolved
ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

## It's internet test time!

### Check what works using ping
Work the commands from up to down, refer to the description of the successful ping. 
```sh
ping archlinux.org # archlinux website - prefectly working internet
ping 1.1.1.1 # cloudflare dns server ip - internet working, but not able to resolve
ping 192.168.1.1 # router ip - internet is down, only lan network
ping 127.0.0.1 # loopback device - you sure you are connected to a network?
```

### List available network interfaces
```sh
ip a # show all interfaces
ip a show eno1 # show status of specified interface
networkctl list # systemd-networkd tool to show all and configuration status
```

### Check status of systemd services
```sh
systemctl status systemd-networkd
systemctl status systemd-resolved
systemctl status iwd
```
