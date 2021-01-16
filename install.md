# Installation

## Preparation

### Networking on archiso
On the live environment, a ethernet cable is usually enough to get you started on the installation. However, if you wanted to use a wireless network, you will need to connect to one first. See networking [here][network]. 

### Enable ssh for access across network
Before you ssh to the machine, you have to set a root password first. Note that the option ```PermitRootLogin``` is set to ```yes``` by default on the live environment, so we don't have to bother with config and can log in right away. 
```sh
systemctl start sshd
passwd root
```

## Disk partitioning

### Usual setups

#### Boot partition
128 MiB or more, though my typical setup usually takes around only 50 MiB of space. \
Only necessary if the boot loader explicitly requires it. For example, UEFI direct boot (EFISTUB) will need an EFI system partition. 

#### Separate root/home partition
Around 32 GiB for root partition and the rest for home partition, optional. \
This creates a home partition separated from the main root partition, often to make reinstalls easier, or to protect the system from out-of-space situations. 

### Partition disk
We will use ```fdisk``` to do the partitioning. Determine the right block device by judging the disk model, then just do it using the commands. You might want to checkout the example layouts [here][suggested_layouts]. 
```sh
fdisk -l /dev/nvme0n1
fdisk /dev/nvme0n1
```

### Format partitions
After making the partitions, we need to format them. 
```sh
mkfs.fat -F32 /dev/nvme0n1p1 # make EFI filesystem
mkfs.ext4 /dev/nvme0n1p2
```

Now, the disk partitions should be ready for the bootstrapping process. 

## Installing base system

### Mount the partitions to /mnt
Mount them all! This will be important for ```pacstrap``` and ```genfstab``` steps below. 
```sh
# mount root
mount /dev/nvme0n1p2 /mnt
# mount boot
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
# mount home
mkdir /mnt/home
mount /dev/nvme0n1p3 /mnt/home
```

### Bootstrap the base system and generate fstab
Oh and one more thing: Download the microcode along with pacstrap. Microcode contains fixes that can be useful and is used on boot. \
We will use Intel as examples here. Change it to ```amd-ucode``` if your CPU is AMD.
```sh
pacstrap /mnt base linux linux-firmware intel-ucode
genfstab -U /mnt >> /mnt/etc/fstab
```

## Configure bootloader

### UEFI direct boot (EFISTUB)
If you already set up an EFI filesystem accordingly, the process is just adding an uefi boot entry using ```efibootmgr```, and after that you should be good to go! \
Look up root partition UUID using ```ls -l /dev/disk/by-uuid```, and remember to change the microcode image accrodingly. 
```sh
LABEL=archlinux
UUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
UCODE=intel-ucode.img
efibootmgr --disk /dev/nvme0n1 --part 1 --create --label "$LABEL" --loader /vmlinuz-linux --unicode "root=UUID=$UUID rw initrd=\\$UCODE initrd=\initramfs-linux.img" --verbose
```

### Syslinux
This involves ```pacman``` and ```chroot``` which are not ready at this stage, only proceed after the Post-configuration section. \
The live environment comes with a script called ```syslinux-install_update``` that will most of the job setting up syslinux. However the config file created is usually error-prone, so it is important to check for incorrect parameters. \
Additional configurations and kernel parameters can be found on the Arch Wiki page [here][syslinux-config] and [here][syslinux-kernel]. 
```sh
pacman -S syslinux
syslinux-install_update -i -a -m
nano /boot/syslinux/syslinux.cfg
```

## Post-configuration

### Change root into the new system
Do this before doing anything else below, or the changes will go to the live environment, which is most likely not what you'd want. 
```sh
arch-chroot /mnt
```

### Set local time zone
```sh
REGION=Hongkong
ln -sf /usr/share/zoneinfo/$REGION /etc/localtime
hwclock --systohc
```

### Set and generate locale
```sh
LANG=en_US.UTF-8
sed -i 's/#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
echo "LANG=$LANG" > /etc/locale.conf
locale-gen
```

### Set hostname and create hosts file
```sh
HOST=archlinux
echo "$HOST" > /etc/hostname
printf "127.0.0.1\tlocalhost\n::1\t\tlocalhost\n127.0.1.1\t$HOST.localdomain\t$HOST\n" >> /etc/hosts
```

### Initialize pacman
```sh
pacman-key --init
pacman-key --populate
pacman -Sy
```

### Set a root password
Set a root password for maintenance purposes, in case something breaks. 
```sh
passwd
```

[network]: network.md
[suggested_layouts]: https://wiki.archlinux.org/index.php/partitioning#Example_layouts
[syslinux-config]: https://wiki.archlinux.org/index.php/Syslinux#Configuration
[syslinux-kernel]: https://wiki.archlinux.org/index.php/Syslinux#Kernel_parameters
