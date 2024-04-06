
# Arch Linux install Guide

## Pre-Install

---

- [Download Arch ISO](https://archlinux.org/download/)
- Write to USB With [Balena Etcher](https://www.balena.io/etcher/), [Rufus](https://rufus.ie/en/#), or [Ventoy(*Recommended*)](https://www.ventoy.net/en/download.html)  
- Boot to the [Live USB](https://en.wikipedia.org/wiki/Live_USB)

## Installing

---

- Set Time  

``` bash
timedatectl set-ntp true
```

- Partition with `cfdisk`  

```bash
cfdisk /dev/sdx
```

|Mount Point|   Partition    | Partition Type |  Recommended Sizes  |
|-----------|----------------|----------------|---------------------|
|`/mnt/boot`|`/dev/efi_part` |efi_system_part |*500MB*              |
|`[swap]`   |`/dev/swap_part`|Linux swap      |*More than 1GB*      |
|`/mnt`    |`/dev/root_part`|Linux-filesystem|*Remainder of device*|

>Swap is very optional on systems with more RAM [(Learn More)](https://wiki.archlinux.org/title/Swap)

- Format the partitions (*you don't need to mkswap if you don't make a swap partition*)

``` bash
mkfs.ext4 /dev/_root_partition_
mkswap /dev/_swap_partition_
mkfs.fat -F 32 /dev/_efi_system_partition_
```

- Mount partitions

``` bash
mount /dev/_root_partition_ /mnt
mount /dev/_efi_system_partition_ /mnt/boot
swapon /dev/swap
```

- Install needed things

> the linux LTS package means long-time-support, you may replace these kernel packages with the kernel of your choice.

``` bash
pacstrap /mnt base base-devel linux-lts linux-lts-headers linux-firmware nano sudo
```

>You may replace nano with your terminal editor of choice.

- Generate fstab

``` bash
genfstab -U /mnt >> /mnt/etc/fstab
```

- Chroot into the system

``` bash
arch-chroot /mnt
```

- Set Time and Locales

``` bash
ln -sf /usr/share/zoneinfo/[Region]/[City] /etc/localtime
hwclock --systohc
```

> Region/City example: America/New_York [US/East]  

[Edit](https://wiki.archlinux.org/title/Textedit "Textedit")  `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed [locales](https://wiki.archlinux.org/title/Locale "Locale"). After that, run

``` bash
locale-gen
echo "LANG=_en_US.UTF-8_" >> /etc/locale.conf
echo "[your_computerhostname]" > /etc/hostname
```

- Setup Users  

Set root password with `passwd`
Edit sudoers file with `EDITOR='nano' visudo`, find and uncomment  
`#%sudo ALL=(ALl) All`. So it should look like this:  

``` bash
%sudo ALL=(ALL) ALL
```

Create a user account with `sudo` permissions by using

```bash
useradd -m -G wheel,sudo [your_username_here]
passwd [your_username_here]
```

> If it returns an error, try running `groupadd sudo`
  
- Install grub bootloader  

``` bash
pacman -Sy grub
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

If dual-booting run `pacman -S os-prober` and edit the grub config at `/etc/default/grub` and setting `GRUB_DISABLE_OS_PROBER=true` to false, `GRUB_DISABLE_OS_PROBER=false`

- Finishing up  

For internet connection you may want to install and enable dhcp. You can do this with the following

```bash
pacman -S dhcpcd
systemctl enable dhcpcd 
```

Exit the chroot by typing `exit` then unmount the mounted partitions using `umount /dev/sdx1` for all the mounted partitions.  

Do a `reboot now` to and restart into the new arch system.  

### Desktop environment installation

---
>You can do all of this while in the `arch-chroot`

Log in as root with either [`su`](https://wiki.archlinux.org/title/su) or by exiting the current session and using the username `root`.
now install the required packages for a desktop environment, for example kde-plasma.

``` bash
pacman -Sy xorg plasma-desktop sddm [terminal_emulator] [web_browser]
systemctl enable sddm
```

You'll want to install video drivers next. If you are running an modern nvidia graphics card then follow the instructions below. Otherwise, go to the [Arch wiki](https://wiki.archlinux.org) and search for your drivers.

```bash
sudo pacman -S nvidia nvidia-dkms nvidia-settings nvidia-utils
```

Now, on boot you should see a login screen, just log in with the user account you made earlier.

### Use the AUR

---

***only install AUR packages as a regular user, without using sudo.*** It will ask for sudo password during the process.  

There are two ways to use the arch user repository, manually and using an AUR helper. Do not do either as root, or with sudo!

#### Manually

***git** must be installed*

``` bash
git clone [aur package git link]
cd ./[aur package name]
makepkg -si
```

#### With an AUR helper

***git** must be installed*

```bash
git clone https://aur.archlinux.org/yay.git 
cd yay
makepkg -si
cd ..
rm -rf yay
```

Then to use it, use  

```bash
yay -S [package]
```

yay uses the same syntax as pacman does.
