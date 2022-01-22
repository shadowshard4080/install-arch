# How to use Pacman 
###### *The basics of Pacman*  

- ### Installing Packages  
*Before Installing Packages you should refresh the package databases with either `pacman -Sy` or `pacman -Syy` to update all databases even if they are up to date.*  

To install packages with pacman use
```
sudo pacman -S [package_name]
```
or to update databases and install packages at the same time use,
```
sudo pacman -Sy [package_name]
```  

To install multiple packages use a space seperated list, for example, to install *terminator* and *nano* do the following,
```
sudo pacman -S nano terminator
```

To skip confirmation, for use in scripts, use the following
```
sudo pacman -S [package_name] --noconfirm
```
> The `--noconfirm` flag tells pacman to use the defualt options instead of asking.  
 
or pipe the output of `yes`, which is just *y* repeating, into the command. ***Avoid Using This to Prevent Undesireable Results***.
```
yes | sudo pacman -S [package_name]
```

>When installing packages in a script, it's a good idea to use the `--needed` flag to prevent re-installation of already existing, up-to-date packages.

To search for a package to install you can use
```
sudo pacman -sS [package name]
```

- ### Updating Packages
To update all out-of-date packages use
```
sudo pacman -Syu
```
> The `-y` flag updates databases and the `-u` flag performs a system upgrade

- ### Removing Packages
Removing a packages is a easy as
```
sudo pacman -R [package_name]
```

For both flags `-S` and `-R` the addional flag `-p` that, instead of running the operation, prints the targets of the operation.

>[*Learn more about pacman*](https://archlinux.org/pacman/pacman.8.html)