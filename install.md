# Installation

## Pre-configuration

### Networking on archiso
On archiso, plugging in a ethernet cable is enough give you a working internet connection. However, if you are determined and crazy enough to do it on a wireless network, you will need to connect to one first. See networking [here][network]. 

### Enable ssh for easier access
Note that changes on archiso are temporary, and will not survive a reboot. \
And also you have to set a root password for ssh, duh...
```sh
systemctl start sshd
passwd root
```

### Update system clock
This will make use of systemd-timesyncd's internal ntp. 
```sh
timedatectl set-ntp true
timedatectl status
```

### Optimize mirrorlist and update package databases
This involves using reflector, a python script to help retrieving latest mirror list, and sort them by speed and location. 
```sh
reflector --threads 20 -c us -f 20 --save /etc/pacman.d/mirrorlist
pacman -Sy
```

## Disk partitioning

### Partition disk using fdisk
Some common block devices: ```/dev/sda``` ```/dev/nvme0n1``` ```/dev/mmcblk0``` \
Example: 
```sh
fdisk /dev/nvme0n1
```

### Make filesystem using mkfs
Some common block device partitions: ```/dev/sda2``` ```/dev/nvme0n1p2``` ```/dev/mmcblk0p2``` \
Examples: 
```sh
mkfs.fat -F32 /dev/nvme0n1p1 # make EFI filesystem
mkfs.ext4 /dev/nvme0n1p2
```

### Common partition descriptions

#### Boot partition
128 MiB or more, though it takes only around 50 MiB of space. \
A separate /boot partition is only required if the boot loader explicitly needs it. \
For example, UEFI boot requires an EFI system partition. 

#### Swap partition
More than 512 MiB, optional. \
A swap partition is preferred only if the device lacks memory. \
Note that swapfile can provide the same functionality with identical performance, while being much easier to be resized. 

#### Separate root/home partition
Around 32 GiB for root partition and the rest for home partition, optional. \
This configuration creates a home partition separated from the main root partition, often to make reinstalls easier, or to protect the system from out-of-space situations. 

## Installing base system

### Mount partition to /mnt
Replace with appropriate block device. 
```sh
mount /dev/nvme0n1p2 /mnt
```
If a separate boot partition exists, do something like: 
```sh
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```

### Bootstrap the base system and generate fstab
```sh
pacstrap /mnt base linux linux-firmware
genfstab -U /mnt >> /mnt/etc/fstab
```

## Configure bootloader

### UEFI boot
Basically add an uefi boot entry using efibootmgr and you should be good to go! \
You can look up partition UUID using ```ls -l /dev/disk/by-uuid```. 
```sh
LABEL=archlinux
ROOT_UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

efibootmgr --disk /dev/nvme0n1 --part 1 --create --label "$LABEL" --loader /vmlinuz-linux --unicode "root=UUID=$ROOT_UUID rw initrd=\initramfs-linux.img" --verbose
```

### Syslinux
Note that the ```syslinux-install_update``` will not do a perfect job creating the config file i.e. usually the block devices are wrong, so it is important to check for incorrect parameters. \
Additional configurations and kernel parameters can be found on the Arch Wiki page [here][syslinux-config] and [here][syslinux-kernel]. 
```sh
pacman -S syslinux
syslinux-install_update -i -a -m
nano /boot/syslinux/syslinux.cfg
```

## Post-configuration

### Change root into the new system
Do this before doing anything else below. Or changes go bye bye. 
```sh
arch-chroot /mnt
```

### Set local time zone
```sh
REGION=Hongkong

ln -sf /usr/share/zoneinfo/$REGION /etc/localtime
hwclock --systohc
```

### Generate locale
```sh
LANG=en_US.UTF-8

sed -i 's/#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
echo "LANG=$LANG" > /etc/locale.conf
locale-gen
```

### Create /etc/hostname and /etc/hosts
```sh
HOST=archlinux

echo "$HOST" > /etc/hostname
printf "127.0.0.1\tlocalhost\n::1\t\tlocalhost\n127.0.1.1\t$HOST.localdomain\t$HOST\n" >> /etc/hosts
```

### Set a root password
If you don't want to be locked out of your new system, that is. \
You can still set up and use ssh, just that the console will not let you in. 
```sh
passwd
```

[network]: network.md
[syslinux-config]: https://wiki.archlinux.org/index.php/Syslinux#Configuration
[syslinux-kernel]: https://wiki.archlinux.org/index.php/Syslinux#Kernel_parameters
