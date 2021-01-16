# Network configurations

## Wireless networks

### Live environment
The live environment already has the service enabled and started by default. \
To manually restart it anyway, do:
```sh
systemctl restart iwd.service
```

### New installation
Download ```iwd```, then start the systemd service. 
```sh
pacman -S iwd
systemctl start iwd.service
```

### Connect to a network
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
Name=eno1

[Network]
DHCP=yes
```

```35-wireless.network```: 
```
[Match]
Name=wlan0

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

### Link resolv.conf
```sh
ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

### Enable and start the network services
```sh
systemctl enable systemd-networkd systemd-resolved
systemctl start systemd-networkd systemd-resolved
```

## Internet test time!

### Check what works using ping
Work the commands from top to bottom, refer to description of the successful ping. 
```sh
ping archlinux.org # arch linux site - working internet
ping 1.1.1.1 # cloudflare dns - unable to resolve
ping 192.168.1.1 # router - no access to internet, only lan network
```

### Show info of network interfaces
Below commands might help. 
```sh
ip a # show all interfaces
ip a show eno1 # show status of specified interface
networkctl list # show systemd-networkd config status
```

### Check status of network services
```sh
systemctl status systemd-networkd
systemctl status systemd-resolved
systemctl status iwd
```
