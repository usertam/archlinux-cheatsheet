# Using AUR packages

## Prerequisite

### Set up a non-root user
Since ```makepkg``` refuses to work with root privileges, we have to create a normal user with sudo capabilities to work with AUR packages. 

Some requirments: 
* A non-root user is created
* The user is able to use ```sudo```
    * User belongs in ```wheel``` group
    * User has own password set

Below are some commands to quickly do that: 
```sh
useradd -m -U user -G wheel
passwd user
pacman -S sudo
sed -i 's/# %wheel ALL=(ALL) ALL/%wheel ALL=(ALL) ALL/' /etc/sudoers
```

### Install packages
```sh
pacman -S --needed base-devel git
```

## Set up an AUR helper

### Using yay
```sh
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### Test it out!
```sh
yay
```
