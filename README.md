insuhkim's Endeavour OS setup
========



# System Related
## Update mirrors & Check Update
```bash
reflector-simple
eos-update
eos-update --aur
yay
```

## Enable Bluetooth 
```bash
sudo systemctl enable --now bluetooth
```


## [TLP for laptop battery](https://linrunner.de/tlp/installation/arch.html)
```bash
sudo pacman -S tlp tlp-rdw
sudo systemctl enable tlp.service
sudo systemctl enable NetworkManager-dispatcher.service
systemctl mask systemd-rfkill.service systemd-rfkill.socket
```
If it conflicts with power-profiles-daemon, remove it
```bash
sudo pacman -R power-profiles-daemon
```


## Korean Input
1. Install fcitx5 for hangul
```bash
sudo pacman -S fcitx5-im fcitx5-hangul
fcitx5-configtool
```

2. Inside configtool, add "hangul" input method   

3. (optional) in global option change ctrl-space to shift-space

4. Set input method to fcitx5
select fcitx5 in system settings -> keyboard -> virtual keyboard 

5. (optional) in keyboard setting goto 'key bindings' and select 'make right alt a hangul key' and 'make right ctrl a hanja key' if you want

6. Add next line to /etc/environment 
```bash
XMODIFIERS=@im=fcitx
```
<https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#KDE_Plasma>   
log out and it will work


## [Change grub resolution](https://askubuntu.com/questions/54067/how-to-safely-change-grub2-screen-resolution)

1. type videoinfo in grub console and check what resolutions are available
2. open /etc/default/grub file with sudo
3. change GRUB_GFXMODE with the resolution you want
4. make update-grub command in /usr/sbin/update-grub
```bash
#!/bin/sh
set -e
exec grub-mkconfig -o /boot/grub/grub.cfg "$@"
```

5. [Make update-grub command](https://askubuntu.com/questions/418666/update-grub-command-not-found)

```bash
sudo chown root:root /usr/sbin/update-grub
sudo chmod 755 /usr/sbin/update-grub
sudo update-grub
```


# personal programs I need

## Terminal Setup
### neovim
``` bash
sudo pacman -S neovim 
sudo pacman -S wl-clipboard
```

<https://askubuntu.com/questions/1486871/how-can-i-copy-and-paste-outside-of-neovim>


## kde sweet theme
see <https://www.youtube.com/watch?v=nmQn-JRwlo0>

## internet browser
```
sudo pacman -S vivaldi
```


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





## bottles
1. flatpak 
```
sudo pacman -Syu
sudo pacman -S flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```
restart
```
flatpak install flathub com.usebottles.bottles
flatpak override com.usebottles.bottles --user --filesystem=xdg-data/applications
```


https://velog.io/@duboo/Ubuntu-20.04-%EC%B9%B4%EC%B9%B4%EC%98%A4%ED%86%A1-%EC%84%A4%EC%B9%98%EB%B0%A9%EB%B2%95-%ED%95%9C%EA%B8%80%EA%B9%A8%EC%A7%90-%EB%AC%B8%EC%A0%9C

2. flatseal(enables bottles to access user file)
```
flatpak install flathub com.github.tchx84.Flatseal
```
execute flatseal and allow "All User Files" in bottles


## [winapps](https://github.com/winapps-org/winapps?tab=readme-ov-file)
clone the project

## Zapret(DPI)
1. download newest release of [Zapret](https://github.com/bol-van/zapret/releases)

2. unzip it and move folder to /opt/zapret
run
```bash
./blockcheck.sh
./install_easy.sh
```
3. Enable zapret
```
sudo systemctl enable zapret
sudo systemctl start zapret
```

# TODO & not completed

## Hybrid GPU(nvidia-prime or optimus)


## terminal setting
### zsh & oh-my-zsh
``` 
sudo pacman -S zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
log out and zsh will be default shell

powerlevel10k theme
```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
```
change the value of ZSH_THEME in ~/.zshrc to "powerlevel10k/powerlevel10k"


```
sudo pacman -S ttf-jetbrains-mono-nerd
```
### atuin
### chezmoi
### btop
### powertop
### dust
### bat
### tldr
### terminal window manager(tmux or zellij or wezterm)
### eza or lsd

### kde configuration with konsave
```
yay -S konsave
```
