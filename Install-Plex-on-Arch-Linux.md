# Install Plex on arch
***DO NOT install AUR packages as ROOT, Don't use sudo.** It will ask for a sudo password during the process.*  

<ins>*Requires **git***</ins>

### With AUR helper
---
>Depending on the AUR helper installed the syntax may vary.
- Install and Enable the Package
```
yay -S plex-media-server
sudo systemctl enable --now plexmediaserver
```
### Without AUR helper 
---
- Clone and [*cd*](https://man.archlinux.org/man/cd.1p) into the Git Repo
```
git clone https://aur.archlinux.org/plex-media-server.git
cd plex-media-server
```
- Build and Enable Plex
```
makepkg -si
sudo systemctl enable --now plexmediaserver
```
>`makepkg` is part of the [Pacman](https://wiki.archlinux.org/title/Pacman) package manager.
- Check Status with
```
sudo systemctl status plexmediaserver
```
>exit this with `q` or *`ctrl + c`* 

Then go to <ins>https://localhost:34200/web</ins> and you should see the web-UI.  
If you don't, you can check the status with