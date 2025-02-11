
# EndeavourOS System Configuration Guide

---

## System Related

### Update Mirrors & Check Updates
```bash
reflector-simple
eos-update
eos-update --aur
yay
```

### Enable Bluetooth
```bash
sudo systemctl enable --now bluetooth
```

### Battery Management with TLP
[TLP Official Installation Guide](https://linrunner.de/tlp/installation/arch.html)

```bash
sudo pacman -S tlp tlp-rdw
sudo systemctl enable tlp.service
sudo systemctl enable NetworkManager-dispatcher.service
sudo systemctl mask systemd-rfkill.service systemd-rfkill.socket
```
If it conflicts with `power-profiles-daemon`, remove it:

### Fix Audio Issues
[Audio is too loud when volume is set greater than 0 on Lenovo Yoga](https://github.com/alsa-project/alsa-lib/issues/366)

#### Solution 1: Modify `analog-input-aux.conf`
Edit `/usr/share/alsa-card-profile/mixer/paths/analog-input-aux.conf` and modify these lines:
```diff
@@ -79,8 +79,6 @@
override-map.2 = all-left,all-right

[Element Master]
-switch = mute
-volume = merge
override-map.1 = all
override-map.2 = all-left,all-right

@@ -243,4 +241,8 @@
override-map.1 = all-center
override-map.2 = all-center,lfe

+[Element Master]
+switch = mute
+volume = ignore
+
.include analog-output.conf.common
```
#### Solution 2: Modify `analog-input-aux.conf.common`
Add these three lines above the `Element PCM`:
```diff
+[Element Master]
+switch = mute
+volume = ignore

[Element PCM]
switch = mute
volume = merge
override-map.1 = all
override-map.2 = all-left,all-right
```
#### Solution 3: Modify GRUB
Add the following parameters to `GRUB_CMDLINE_LINUX_DEFAULT`:
```bash
snd_hda_intel.dmic_detect=0 snd_hda_intel.model=lenovo
```
Update GRUB:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
reboot
```

### Korean Input Configuration
1. Install `fcitx5` for Hangul support:
```bash
sudo pacman -S fcitx5-im fcitx5-hangul
fcitx5-configtool
```
2. Inside config tool, add "hangul" input method.
3. (Optional) Change `Ctrl+Space` to `Shift+Space` in global options.
4. Set input method to `fcitx5` in system settings.
5. (Optional) In keyboard settings, map `Right Alt` to Hangul and `Right Ctrl` to Hanja.
6. Add the following line to `/etc/environment`:
```bash
XMODIFIERS=@im=fcitx
```
Refer to [this guide](https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#KDE_Plasma) for more details.

### Change GRUB Resolution
[Reference](https://askubuntu.com/questions/54067/how-to-safely-change-grub2-screen-resolution)

1. Type `videoinfo` in the GRUB console to check available resolutions.
2. Open `/etc/default/grub` and modify `GRUB_GFXMODE` with your desired resolution.
3. Create an update-grub command:
```bash
#!/bin/sh
set -e
exec grub-mkconfig -o /boot/grub/grub.cfg "$@"
```
4. Ensure correct permissions:
```bash
sudo chown root:root /usr/sbin/update-grub
sudo chmod 755 /usr/sbin/update-grub
sudo update-grub
```

---

## Personal Programs

### Docker 
```bash
sudo pacman -S docker
sudo systemctl enable --now docker.service
sudo usermod -aG docker $USER
```
Restart the computer
### open-webui
Install with this command:
```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```
After installation, you can access Open WebUI at <http://localhost:3000>.

Start/Stop Container
```bash
docker stop open-webui
docker start open-webui
```

### WinApps
Clone the project and follow the [installation steps](https://github.com/winapps-org/winapps?tab=readme-ov-file#installation).
The descriptions below are only a rough guide. 
1. Install Docker Engine
```bash
# Install Docker and docker-compose
sudo pacman -S docker docker-compose

# Enable and start Docker service
sudo systemctl enable --now docker.service

# Add user to docker group
sudo usermod -aG docker $USER

# Logout and login again to apply group changes
```
2. Install Windows
Follow this [guide](https://github.com/winapps-org/winapps/blob/main/docs/docker.md#installing-windows)

3. Install Dependencies
```bash
sudo pacman -Syu --needed -y curl dialog freerdp git iproute2 libnotify gnu-netcat
```

4. Create A WinApps Configuration File at `~/.config/winapps/winapps.conf`

5. Test FreeRDP
```bash
xfreerdp3 /u:"MyWindowsUser" /p:"MyWindowsPassword" /v:192.168.122.2 /cert:tofu
```

6. Run FreeRDP
```bash
bash <(curl https://raw.githubusercontent.com/winapps-org/winapps/main/setup.sh)
```

### Terminal Setup
#### SSH Key Configuration
1. Ensure `git` and `openssh` are installed:
```bash
sudo pacman -S git openssh
```
2. Create a new SSH key pair:
```bash
ssh-keygen -t ed25519 -C "insuhkim@naver.com"
```
3. Add the SSH key to the SSH Agent:
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
4. Add the SSH Key to GitHub:
   - Copy the public key:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
   - Go to [GitHub SSH Key Settings](https://github.com/settings/ssh/new) and paste the key.
5. Verify SSH Connection (Optional):
```bash
ssh -T git@github.com
```

#### Neovim
```bash
sudo pacman -S neovim wl-clipboard
```
[How to enable clipboard support](https://askubuntu.com/questions/1486871/how-can-i-copy-and-paste-outside-of-neovim)

#### Useful Terminal Programs
```bash
sudo pacman -S btop dust bat tldr lsd zoxide
```
- `btop` - Resource monitor
- `dust` - Disk usage analyzer
- `bat` - Enhanced `cat`
- `tldr` - Simplified `man` pages
- `lsd` - Better `ls`
- `eza` - Also better `ls`
- `powertop` - Power usage analyzer
- `zoxide` - Better `cd`
- `bottom` - Also resource monitor

#### Other Tools
- `atuin`
- Terminal window managers: `tmux`, `zellij`, or `wezterm`
- KDE configuration with `konsave`:

#### Dotfiles with [Chezmoi](https://www.chezmoi.io)
```bash
chezmoi init git@github.com:$GITHUB_USERNAME/dotfiles.git
```

### KDE Sweet Theme
[YouTube Guide](https://www.youtube.com/watch?v=nmQn-JRwlo0)


### Internet Browser
```bash
sudo pacman -S vivaldi
```

### LibreOffice
```bash
sudo pacman -S libreoffice-still
```

### Bottles (For Running Windows Apps)
```bash
sudo pacman -Syu
sudo pacman -S flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install flathub com.usebottles.bottles
flatpak override com.usebottles.bottles --user --filesystem=xdg-data/applications
```
For file access permissions, install Flatseal:
```bash
flatpak install flathub com.github.tchx84.Flatseal
```
Enable "All User Files" in Flatseal for Bottles.

### Zapret (DPI Circumvention)
1. Download the latest release from [Zapret](https://github.com/bol-van/zapret/releases)
2. Unzip and move it to `/opt/zapret`
3. Run:
```bash
./blockcheck.sh
./install_easy.sh
```
4. Enable Zapret:
```bash
sudo systemctl enable zapret
sudo systemctl start zapret
```

---

## TODO & Not Completed

### Hybrid GPU (NVIDIA Prime or Optimus)
### Terminal Settings
#### Zsh & Oh-My-Zsh
```bash
sudo pacman -S zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
Set default shell to Zsh by logging out and back in.

#### Powerlevel10k Theme
```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
```
Modify `~/.zshrc`:
```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```
Install Nerd Font:
```bash
sudo pacman -S ttf-jetbrains-mono-nerd
```


## Deprecated
### Hancom Office
```bash
yay -S hoffice
sudo mv /opt/hnc/hoffice11/Bin/qt{,.bak}
```

