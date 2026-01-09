*[View wiki pages](https://github.com/El-Wumbus/install-arch/wiki)*

# Arch Install Guide
### Updated January 2026
---
This repository provides a comprehensive, easy-to-follow guide for installing vanilla Arch Linux on a UEFI machine. This edition of the install guide emphasizes:

- Better stability (LTS kernel by default)
- Essential packages for a "ready-to-go" experience
- Secure Boot support for seamless Windows dual-booting
- Shared NVMe drive setup for Steam libraries across Windows and Linux
- Gaming-focused configuration (Steam, OpenRGB, Hyprland + HyDE, Xbox controller, NVIDIA drivers, Wine/Bottles)
- Post-install enhancements (OneDrive sync, EasyEffects audio stack)

The core installation remains vanilla Arch—no automated scripts or third-party installers.

> **Warning:** Always double-check disk identifiers with `lsblk` or `fdisk -l`, back up important data, and consider testing in a VM before making permanent changes.

### Tips Before You Begin
---
- Capture quick screenshots or diagrams of the target partition layout and your firmware's Secure Boot pages so you can revert settings confidently.
- Keep a copy of `lsblk`/`fdisk` output in a notes app or on paper while installing to avoid mixing up disks.
- Verify ISO checksums and signatures before flashing the installer media (commands shown in Step 1).
- If you restart after partitioning, ensure you remount partitions before proceeding.
- The official `archinstall` TUI (run from the live media) can walk through partitioning and locale selection and then hand back control, but this guide assumes manual steps for clarity.

### Installation Steps
---
1. **Prepare the Bootable USB**
   - Download the latest Arch Linux ISO from [archlinux.org/download](https://archlinux.org/download/).
   - Verify the download:
     ```bash
     sha256sum archlinux-*.iso
     gpg --keyserver hkps://keys.openpgp.org --recv-keys 0xE8FBA9E3E46C4B17
     gpg --verify archlinux-*.iso.sig archlinux-*.iso
     ```
   - Flash it to a USB drive with Rufus (Windows) or `dd` (Linux/macOS).
     https://rufus.ie/en/
     

2. **Boot into the Live Environment**
   - Enter BIOS/UEFI (F2/Del/etc.), ensure UEFI mode is enabled, and boot from the USB.
   - Once the live shell loads, synchronize time:
     ```bash
     timedatectl set-ntp true
     ```

3. **Connect to the Internet**
   - **Ethernet:** Usually connects automatically.
   - **Wi-Fi (with `iwctl`):**
     ```bash
     iwctl
     device list
     station wlan0 scan
     station wlan0 get-networks
     station wlan0 connect "YourSSID"
     # Enter the password if prompted
     exit
     ```
   - Test connectivity:
     ```bash
     ping archlinux.org
     ```

4. **Partition the Disks**
   - You may skip doing this manually by using `archinstall`, but otherwise:
   - Use `lsblk` to identify disks (ideally, Arch on `/dev/nvme1n1`, shared games drive on `/dev/nvme2n1`, Windows on `/dev/nvme0n1`).
   - For the Arch drive:
     ```bash
     cfdisk /dev/nvme1n1
     # Create:
     #  - EFI partition: 512MB, type EFI System
     #  - Optional swap: 8–16GB (or match RAM for hibernation)
     #  - Root: Remaining space, type Linux filesystem
     ```
   - Shared games drive (format as NTFS if needed for Windows/Linux sharing):
     ```bash
     mkfs.ntfs -Q /dev/nvme2n1  # Quick format; wipes all data!
     ```
   > Prefer a guided partitioner while staying close to vanilla? Launch `archinstall` from the live shell to step through disk prep interactively, then return here.

5. **Format and Mount Partitions (Arch Drive)**
   ```bash
   mkfs.fat -F32 /dev/nvme1n1p1  # EFI
   mkswap /dev/nvme1n1p2         # Swap (if created)
   swapon /dev/nvme1n1p2
   mkfs.ext4 /dev/nvme1n1p3      # Root

   mount /dev/nvme1n1p3 /mnt
   mount --mkdir /dev/nvme1n1p1 /mnt/boot
   ```

6. **Install Base System (Stability + Essentials)**
   ```bash
   pacstrap /mnt base linux-lts linux-lts-headers linux-firmware base-devel nano sudo git networkmanager man-db man-pages efibootmgr
   ```

7. **Generate `fstab`**
   ```bash
   genfstab -U /mnt >> /mnt/etc/fstab
   ```

8. **Chroot into the New System**
   ```bash
   arch-chroot /mnt
   ```

9. **Basic Configuration (Inside Chroot)**
   - **Timezone (example: Seattle, see the [Arch timezone list](https://wiki.archlinux.org/title/Time_zone)):**
     ```bash
     ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
     hwclock --systohc
     ```
   - **Locales:** Edit `/etc/locale.gen` (uncomment `en_US.UTF-8 UTF-8` and desired locales), then:
     ```bash
     locale-gen
     echo "LANG=en_US.UTF-8" > /etc/locale.conf
     ```
   - **Hostname:**
     ```bash
     echo myhostname > /etc/hostname
     ```
   - **Root Password:**
     ```bash
     passwd
     ```
   - **Sudo + User:** Edit `/etc/sudoers` (uncomment `%wheel ALL=(ALL:ALL) ALL`), then create a user:
     ```bash
     useradd -m -G wheel,sudo yourusername
     passwd yourusername
     ```
   - You'll need to quit archinstall here if you used it to skip these steps.

10. **Bootloader with Secure Boot (systemd-boot Recommended)**
    - Install and configure Secure Boot tooling:
      ```bash
      bootctl --path=/boot install
      pacman -S sbctl sbsigntools
      sbctl create-keys
      sbctl enroll-keys --microsoft  # Keeps Windows bootable
      ```
    - Enable UKI support: edit `/etc/mkinitcpio.conf` HOOKS to include `systemd` and `kms`, then:
      ```bash
      mkinitcpio -P
      sbctl sign -s /boot/EFI/BOOT/BOOTX64.EFI
      sbctl sign -s /boot/EFI/systemd/systemd-bootx64.efi
      sbctl sign -s /boot/EFI/Linux/*.efi
      sbctl verify
      ```
    - Create `/boot/loader/entries/arch.conf`:
      ```
      title   Arch Linux LTS
      linux   /vmlinuz-linux-lts
      initrd  /initramfs-linux-lts.img
      options root=PARTUUID=xxxx-xxxx rw  # Replace with your root PARTUUID (via blkid)
      ```
    - Edit `/boot/loader/loader.conf` to set `default arch.conf` and `timeout 5`.
    - *Tip:* Snap a photo of your Secure Boot firmware page before changing keys so you can restore the prior state if needed.

11. **Enable Services and Reboot**
    ```bash
    systemctl enable NetworkManager
    exit
    umount -a
    reboot
    ```
    After reboot, re-enter BIOS to enable Secure Boot. Windows should remain selectable in the boot menu.

### Post-Installation Setup
---
#### Shared NVMe Drive for Steam Games, or add NVMe drives
Consider using GParted: 
```bash
sudo pacman -S gparted
```
Otherwise, manually:
```bash
sudo pacman -S ntfs-3g
sudo mkdir /mnt/games
# Add to /etc/fstab (replace UUID with blkid output):
UUID=XXXX-XXXX /mnt/games ntfs-3g uid=1000,gid=1000,umask=0022 0 0
sudo mount -a
```
Set `/mnt/games/SteamLibrary` as a Steam library directory (D:\SteamLibrary will still work in Windows).

#### Sample Btrfs `fstab` Entry (if you prefer snapshots)
```bash
UUID=YYYY-YYYY / btrfs subvol=@,compress=zstd,ssd,space_cache=v2,noatime 0 0
UUID=YYYY-YYYY /home btrfs subvol=@home,compress=zstd,ssd,space_cache=v2,noatime 0 0
```
Adjust UUIDs/subvol names to match `btrfs subvolume list /mnt` output.

#### Desktop Environment Options (Wayland Focused)
Choose the environment that fits your workflow; both run well on gaming rigs and support Secure Boot setups.

**KDE Plasma (Wayland Session)**
```bash
sudo pacman -S xorg plasma-desktop sddm konsole firefox dolphin \
  plasma-wayland-session kde-gtk-config kvantum-qt5 packagekit-qt5 discover
sudo systemctl enable sddm
sudo systemctl enable NetworkManager
```
After reboot, pick the Wayland session on the SDDM login screen.

**Hyprland + HyDE (Tiling Wayland WM)**
```bash
sudo pacman -S hyprland xdg-desktop-portal-hyprland
```
Then install HyDE:
```bash
git clone https://github.com/HyDE-Project/HyDE.git
cd HyDE
./install.sh
```
Follow the HyDE prompts for theming and plugins.

#### Using the Arch User Repository (AUR)
There are two ways to use the Arch User Repository—manual builds or an AUR helper. Do **not** run either method as `root` or with `sudo` (the tooling will prompt for elevated access when needed).

**Manual install (requires `git`):**
```bash
git clone [aur-package-git-url]
cd [aur-package-name]
makepkg -si
```

**With an AUR helper (`yay`, requires `git`):**
```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf yay
```
To install packages afterward:
```bash
yay -S [package]
```
`yay` mirrors `pacman` syntax, so `yay -Syu` refreshes the system plus AUR packages.

#### Gaming Essentials
- Enable `[multilib]` in `/etc/pacman.conf`, then run `sudo pacman -Syu`.
- NVIDIA drivers:
  ```bash
  sudo pacman -S nvidia nvidia-utils nvidia-settings
  # Legacy example:
  yay -S nvidia-470xx-dkms
  ```
- Steam and friends:
  ```bash
  sudo pacman -S steam vulkan-icd-loader lib32-vulkan-icd-loader wine bottles
  yay -S openrgb xone-dkms
  ```
Xone is for easy Xbox controller support.

#### OneDrive Support
```bash
yay -S onedrive-abraunegg
# Optional GUI:
yay -S onedrivegui-git
```
If you choose to use the GUI, follow the steps in the app. Otherwise, set up abraunegg OneDrive in terminal:
```bash
onedrive  # Authenticate via browser
systemctl --user enable --now onedrive
```

#### Improved Audio (PipeWire + EasyEffects)
```bash
sudo pacman -S pipewire pipewire-audio pipewire-pulse pipewire-alsa wireplumber easyeffects
systemctl --user enable --now pipewire pipewire-pulse wireplumber
```
Launch EasyEffects for EQ presets, bass boost, and filters.

#### Config Backup with `rsync`
```bash
sudo rsync -aAXHv --delete /etc /home /var/lib/NetworkManager/system-connections \
  /mnt/backup/$(hostname)-$(date +%F)
```
Capture `/etc`, dotfiles (`~/.config`), and custom Hyprland/HyDE assets before big tweaks so you can roll back quickly.

### Troubleshooting Appendix (Quick Hits)
---
- **Wi-Fi fails in live ISO:** ensure `rfkill list` shows devices unblocked, then reload drivers with `modprobe -r iwlwifi && modprobe iwlwifi`.
- **Secure Boot errors on boot:** boot into BIOS, clear keys, re-run `sbctl verify`, and resign the EFI binaries (`sbctl sign -s ...`).
- **NVIDIA + Wayland glitches:** switch to the latest `nvidia` + `nvidia-settings`, set `nvidia_drm.modeset=1` in the loader options, and ensure `vrr=1` is only enabled where supported.


### Additional Resources
---
- [Arch Wiki: NVIDIA](https://wiki.archlinux.org/title/NVIDIA)
- [Secure Boot Guide by jpetazzo](https://jpetazzo.github.io/2024/02/23/archlinux-luks-tpm-secureboot-install/)
- [HyDE Project](https://github.com/HyDE-project/hyde)
- [Hyprland Community Themes & Configs](https://github.com/hyprland-community/awesome-hyprland)
- [Archinstall Reference](https://wiki.archlinux.org/title/Archinstall)
- [Clone El-Wumbus's Arch Wiki](`git clone https://github.com/El-Wumbus/install-arch.wiki.git`)

