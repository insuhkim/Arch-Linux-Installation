# systems setting
## update mirrors & native/aur packages
```
reflector-simple
eos-update
eos-update --aur
yay
```

## setup bluetooth
```
sudo systemctl enable --now bluetooth
```

## install some programs I need
``` 
sudo pacman -S neovim 
sudo pacman -S wl-clipboard

```
https://askubuntu.com/questions/1486871/how-can-i-copy-and-paste-outside-of-neovim

## setup fcitx5-hangul
```
sudo pacman -S fcitx5-im fcitx5-hangul
```

run 
```
fcitx5-configtool
```
and add hangul
in global option change ctrl-space to shift-space
system settings -> keyboard -> virtual keyboard 
select fcitx5 
Fcitx should be launched by KWin under KDE Wayland in order to use Wayland input method frontend. This can improve the experience when using Fcitx on Wayland. To configure this, you need to go to "System Settings" -> "Virtual keyboard" and select "Fcitx 5" from it. You may also need to disable tools that launches input method, such as imsettings on Fedora, or im-config on Debian/Ubuntu. For more details see https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#KDE_Plasma 

in keyboard setting goto 'key bindings' and select 'make right alt a hangul key' and 'make right ctrl a hanja key' if you want

https://blog.litehell.info/post/fcitx5_for_101_key_keyboard_kde_laptop

log out and it will work


## kde configuration with konsave
```
yay -S konsave
```

## change grub resolution
https://askubuntu.com/questions/54067/how-to-safely-change-grub2-screen-resolution
https://askubuntu.com/questions/418666/update-grub-command-not-found

1. type videoinfo in grub console and check what resolutions are available
2. open /etc/default/grub file with sudo
3. change GRUB_GFXMODE with the resolution you want
4. make update-grub command in /usr/sbin/update-grub
```
#!/bin/sh
set -e
exec grub-mkconfig -o /boot/grub/grub.cfg "$@"
```

5. run 
```
sudo chown root:root /usr/sbin/update-grub
sudo chmod 755 /usr/sbin/update-grub
sudo update-grub
```


## python pip setup
```
sudo pacman -S python3-pip
```



## internet browser
```
sudo pacman -S vivaldi
```


# office tools 
## libre office
```
sudo pacman -S libreoffice-fresh
```

## hancom office
```
yay -S hoffice
```
korean input
```
cd /opt/hnc/hoffice11/Bin/
sudo mv qt qt.bak
```


# zsh
``` 
sudo pacman -S zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
log out and zsh will be default shell

powerlevel10k theme
```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
```
change the value of ZSH_THEME in ~/.zshrc to "powerlevel10k/powerlevel10k"


```
sudo pacman -S ttf-jetbrains-mono-nerd
```



# bottles
1. flatpak 
```
sudo pacman -Syu
sudo pacman -S flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```
restart
```
flatpak install flathub com.usebottles.bottles
```


https://velog.io/@duboo/Ubuntu-20.04-%EC%B9%B4%EC%B9%B4%EC%98%A4%ED%86%A1-%EC%84%A4%EC%B9%98%EB%B0%A9%EB%B2%95-%ED%95%9C%EA%B8%80%EA%B9%A8%EC%A7%90-%EB%AC%B8%EC%A0%9C

2. flatseal(enables bottles to access user file)
```
flatpak install flathub com.github.tchx84.Flatseal
```
execute flatseal and allow "All User Files" in bottles



# zapret
download newest release
https://github.com/bol-van/zapret

unzip it and run 
```
./blockcheck.sh
./install_easy.sh
```

you might need to move folder to /opt/zapret

and execute 
```
sudo systemctl enable zapret
sudo systemctl start zapret
sudo systemctl status zapret
```

# winapps
https://nowsci.com/winapps/kvm/

https://forum.manjaro.org/t/unable-to-connect-to-libvirt-lxc/65177/1

https://www.youtube.com/watch?v=p8cT57Tyckc

https://svrforum.com/software/2054977
