# Arch Linux install Guide
### Pre-Install
---
- [Download Arch ISO](https://archlinux.org/download/)
- Write to USB With [Balena Etcher](https://www.balena.io/etcher/), [Rufus](https://rufus.ie/en/#), or [Ventoy(*Recomended*)](https://www.ventoy.net/en/download.html)
- Boot to the [Live USB](https://en.wikipedia.org/wiki/Live_USB)
### Installing
---
- Set Time  
```
timedatectl set-ntp true
```
- Partition with `cfdisk`  
```
cfdisk /dev/sdx
```

|Mount Point|   Partition	   | Partition Type |  Recommended Sizes  |
|-----------|----------------|----------------|---------------------|
|`/mnt/boot`|`/dev/efi_part` |efi_system_part |*500MB*              |
|`[swap]`	  |`/dev/swap_part`|Linux swap      |*More than 1GB*      |
|`/mnt`		  |`/dev/root_part`|Linux-filesystem|*Remainder of device*|

>Swap is very optional on systems with more RAM [(Learn More)](https://wiki.archlinux.org/title/Swap)

- Format the partitions (*you dont need to mkswap if you dont make a swap partition*)
```
mkfs.ext4 /dev/_root_partition_
mkswap /dev/_swap_partition_
mkfs.fat -F 32 /dev/_efi_system_partition_
```

- Mount partitions
```
mount /dev/_root_partition_ /mnt
mount /dev/_efi_system_partition_ /mnt/boot
swapon /dev/swap
```

- Install needed things 
```
pacstrap /mnt base linux-lts linux-lts-headers linux-firmware nano
```

- Generate fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```

- Chroot into the system
```
arch-chroot /mnt
```

- Set Time and Locales
```
ln -sf /usr/share/zoneinfo/[Region]/[City] /etc/localtime
hwclock --systohc
```
> Region/City example: America/New_York [US/East]  

[Edit](https://wiki.archlinux.org/title/Textedit "Textedit")  `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed [locales](https://wiki.archlinux.org/title/Locale "Locale"). After that, run
```
locale-gen
echo "LANG=_en_US.UTF-8_" >> /etc/locale.conf
echo "[your_computerhostname]" > /etc/hostname
```
- Setup Users  

Set root password with `passwd`
Edit sudoers file with `EDITOR='nano' visudo`, find and uncomment<br>`#%sudo ALL=(ALl) All`. So it should look like this:  
```
%sudo ALL=(ALL) ALL
```
Create a user account with `sudo` permissions by using
```
useradd -m -G wheel,sudo [your_username_here]
passwd [your_username_here]
```
> If it returns an error, try running `groupadd sudo` 
- Install grub bootloader  

```
pacman -Sy grub
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
If dual-booting run `pacman -S os-prober` and edit the grub config at `/etc/default/grub` and setting `GRUB_DISABLE_OS_PROBER=true` to false, `GRUB_DISABLE_OS_PROBER=false`
- Finishing up  

type exit the chroot by typing `exit` then unmount the mounted partitions using `umount /dev/sdx1` for all the mounted partitions. 

do a `reboot now` to and restart into the new arch system.  

### Desktop environment installation
>You can do all of this while in the `arch-chroot`
---
log in as root with either [`su`](https://wiki.archlinux.org/title/su) or by exiting the current session and using the username `root`.
now install the required packages for a desktop environment, for example kde-plasma.
```
pacman -Sy xorg plasma-desktop sddm [terminal_emulator] [web_browser]
systemctl enable sddm
reboot now
```
now on boot you should see a login screen, just log in with the user account you made earlier.
### Use the AUR
---
***only install AUR packages as a regular user, without using sudo.*** It will ask for sudo password during the process.  

There are two ways to use the arch user repository, manually and using an AUR helper. Do not do either as root, or with sudo!
#### Manually
***git** must be installed*
```
git clone [aur package git link]
cd ./[aur package name]
makepkg -si
``` 
#### With an AUR helper
***git** must be installed*
```
git clone https://aur.archlinux.org/yay.git 
cd yay
makepkg -si
cd ..
rm -rf yay
```
Then to use it, use 
```
yay -S [package]
```
yay uses the same syntax as pacman does.