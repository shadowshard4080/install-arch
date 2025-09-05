*[View wiki pages](https://github.com/El-Wumbus/install-arch/wiki)*  

# Arch Linux install Guide
### Installing
---
- Download the latest [Arch Linux ISO](https://archlinux.org/download/) and burn it to a USB using [Rufus](https://rufus.ie/downloads/).
  
- Set Time  
```
timedatectl set-ntp true
```
- Connect to the Internet
  
You can connect with an Ethernet cable, or use [iwctl](https://joshtronic.com/2021/11/21/connecting-to-wifi-with-iwd/).

- Determine the disk you want to use with `fdisk`  
```
fdisk -l
```
- Partition with `cfdisk`

Your drive will be /dev/nvme, /dev/sda, or something similar.
```
cfdisk /dev/sda
```
|Mount Point|Partition		 |Partition Type  |Recommended Sizes |
|-----------|----------------|----------------|------------------|
|`/mnt/boot`|`/dev/efi_part` |efi_system_part |*500MB*|
|`[swap]`	|`/dev/swap_part`|Linux swap|*More than 1GB*|
|`/mnt`		|`/dev/root_part`|Linux-filesystem|*Remainder of device*|

>Swap is very optional.  
- Format the partitions
```
mkfs.fat -F 32 /dev/_efi_system_partition_
mkswap /dev/_swap_partition_
mkfs.ext4 /dev/_root_partition_
```
- Mount partitions
```
mount /dev/_root_partition_ /mnt
mount --mkdir /dev/efi_system_partition /mnt/boot
swapon /dev/swap
```

- Connect to the Internet
If you are already connected via Ethernet, you are good to go. If you'd like to connect to Wi-Fi, consult the directions on [this website.](https://joshtronic.com/2021/11/21/connecting-to-wifi-with-iwd/)

- Install needed things 
```
pacstrap /mnt base linux-lts linux-lts-headers linux-firmware base-devel nano sudo
```
- Generate fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```
- Chroot into the system
```
arch-chroot /mnt
```
- Fix Time
```
ln -sf /usr/share/zoneinfo/_Region_/_City_ /etc/localtime
hwclock --systohc
```
For Pacific Standard time, this will be:
```
ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
hwclock --systohc
```

[Edit](https://wiki.archlinux.org/title/Textedit "Textedit")  `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed [locales](https://wiki.archlinux.org/title/Locale "Locale"). After that, run
```
locale-gen
echo "LANG=en_US.UTF-8_" >> /etc/locale.conf
echo "[your_computerhostname]" > /etc/hostname
```
Set your root password with `passwd`
Edit your sudoers file with `EDITOR='nano' visudo`, find and uncomment<br>`#%sudo ALL=(ALl) All`. So it should look like this:  
```
%sudo ALL=(ALL) ALL
```
Create a user account with `sudo` permissions by using
```
groupadd sudo
useradd -m -G wheel,sudo [your_username_here]
passwd [your_username_here]
```
Install grub bootloader
```
pacman -Sy grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```
```
grub-mkconfig -o /boot/grub/grub.cfg
```
If dual-booting edit the grub config at `/etc/default/grub` and setting `GRUB_DISABLE_OS_PROBER=true` to false, `GRUB_DISABLE_OS_PROBER=false`

### Desktop environment installation
---
Install the required packages for your desktop environment, for example kde-plasma:
> If you have issues during this step, it may be system-specific. Try exiting the chroot by typing `exit` then unmount the mounted partitions using `umount /dev/sdx1` for all the mounted partitions. Then run `reboot now` to restart, mount the partitions, log in as root with either [`su`](https://wiki.archlinux.org/title/su) or by exiting the current session and using the username `root`.

**To install KDE Plasma:**
```
pacman -Sy xorg plasma-desktop sddm [terminal_emulator] [web_browser] git networkmanager packagekit-qt5 plasma-wayland-session discover plasma-nm [optional: 'kde-applications']
systemctl enable sddm
systemctl enable NetworkManager.service
reboot now
```
**To install hyprland:**

[Hyde Project](https://github.com/Hyde-project/hyde)

Now on boot, you should see a login screen, just log in with the user account you made earlier.

### Use the AUR
---
***Only install AUR packages as a regular user. DO NOT use sudo.*** It will ask for sudo password during the process.  

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

To install Steam, edit the pacman config file:

```
sudo nano /etc/pacman.conf
```
Scroll down to the multilib section and uncomment the `[multilib]` line and the `include...` line underneath it. Save the file and exit the editor.
Now run:
```
sudo pacman -Syu Steam
```

Graphics drivers should work out of the box. To install GPU-specific drivers for gaming, refer to the Arch Wiki.

[Nvidia Graphics](https://wiki.archlinux.org/title/NVIDIA)

[AMD Graphics](https://wiki.archlinux.org/title/AMDGPU)

You may also find the Lutris docs helpful.

https://github.com/lutris/docs/blob/master/InstallingDrivers.md#arch--manjaro--other-arch-linux-derivatives

[Instructions for SecureBoot and TPM, by jpetazzo](https://jpetazzo.github.io/2024/02/23/archlinux-luks-tpm-secureboot-install/).

*See more about installing Arch Linux on El-Wumbus's [wiki pages](https://github.com/El-Wumbus/install-arch/wiki), or clone them to your local system.*
```
git clone https://github.com/El-Wumbus/install-arch.wiki.git
```
