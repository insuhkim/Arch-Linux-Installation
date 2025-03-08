
# EndeavourOS System Configuration Guide

---

## Download ISO & Make bootable USB

Download EndeavourOS ISO from [here](https://endeavouros.com)

Boot usb with [balenaEtcher](https://etcher.balena.io) or [ventoy](https://ventoy.net/en/index.html)

Make sure **SECURE BOOT is disabled** in BIOS

Boot the device with usb plugged in and enter boot menu, select EndeavourOS

Before installing EndeavourOS, it is recommended to update mirrors in `online installation` to make installation faster.

## After installation

### Make snapshot

Right after successful installation, make a snapshot just in case.

```bash
sudo pacman -S timeshift
sudo timeshift --create
```

Or you installed with DE, you can use GUI application `timeshift-launcher`

### Enable Bluetooth

```bash
sudo systemctl enable --now bluetooth
```

### TLP battery management in Laptops

[TLP Official Installation Guide](https://linrunner.de/tlp/installation/arch.html)

```bash
sudo pacman -S tlp tlp-rdw
sudo systemctl enable tlp.service
sudo systemctl enable NetworkManager-dispatcher.service
sudo systemctl mask systemd-rfkill.service systemd-rfkill.socket
```

If it conflicts with `power-profiles-daemon`, remove it.

### Fix Audio Issues

There's speaker issue with lenovo yoga.
[Audio is too loud when volume is set greater than 0 on Lenovo Yoga](https://github.com/alsa-project/alsa-lib/issues/366)
To solve this problem, you should follow these steps.

First, in `/usr/share/alsa-card-profile/mixer/paths/analog-output.conf.common`,
add these three lines above the `Element PCM`:

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

Add the following parameters to `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`

```
snd_hda_intel.dmic_detect=0 snd_hda_intel.model=lenovo
```

After that, just update GRUB and reboot:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
reboot
```

### Chaotic AUR

NOTE: the key can change, please check [Chaotic AUR document](https://aur.chaotic.cx/docs)

Run this command to install Chaotic AUR:

```bash
sudo pacman-key --recv-key 3056513887B78AEB --keyserver keyserver.ubuntu.com
sudo pacman-key --lsign-key 3056513887B78AEB

sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst'
sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'
```

Append this line to `/etc/pacman.conf`

```bash
[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
```

### SSH Key

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

### Korean Input Configuration

- Install `fcitx5` for Hangul support:

```bash
sudo pacman -S fcitx5-im fcitx5-hangul
fcitx5-configtool
```

- Inside config tool, add "hangul" input method.
- (Optional) Change `Ctrl+Space` to `Shift+Space` in global options.
- Set input method to `fcitx5` in system settings.
- (Optional) In keyboard settings, map `Right Alt` to Hangul and `Right Ctrl` to Hanja.
- Add the following line to `/etc/environment`:

```bash
XMODIFIERS=@im=fcitx
```

Refer to [this guide](https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#KDE_Plasma) for more details.

### Make `update-grub` command

Create `/usr/sbin/update-grub` with the following content:

```bash
#!/bin/sh
set -e
exec grub-mkconfig -o /boot/grub/grub.cfg "$@"
```

Give it executable permissions:

```bash
sudo chown root:root /usr/sbin/update-grub
sudo chmod 755 /usr/sbin/update-grub
```
---

## How to do things in terminal

If you installed with KDE or other Desktop Environment, you can use GUI.
This is for people who didn't install DE, or just curious about things.

### Mount USB

We will use `udisks2` to automatically mount USB devices.
Make sure `udisks2` exists.

- Plug in your USB drive
- Check the device name of your USB drive by running:

```bash
lsblk
```

This will list all block devices. In most case, your USB drive will likely be named something like `/dev/sdb1` or `/dev/sdc1` or `/dev/sda1`,
the format of `/dev/sdXN`

- To mount a USB device:

```bash
udiskctl mount -b /dev/sdXN
```

USB drive will be mounted in `/run/media/$USER`

### Bluetooth Connection

You can use `bluetoothctl`, but it's a bit complicated. Use `bluetui` instead.
You can install `bluetui` with `pacman`

In `bluetui`, `s` is for scan, `p` is for `pair`. Use `tab` to navigate throught menu.

### Wifi Connection

#### `iwctl`

you can configure wifi with `iwctl`, but using `nmtui` in next line is much easier.

- Display your Wifi stations:

```bash
iwctl station list
```

mostly, station name is `wlan0`.

- Start looking for networks with a station:

```bash
iwctl station station_name scan
```

note that `scan` doesn't give any output

- Display the networks found by a station:

```bash
iwctl station station_name get-networks
```

- Connect to a network with a station, if credentials are needed they will be asked:

```bash
iwctl station station_name connect network_name
```

#### `nmtui`

`nmtui` stands for Network Manager TUI

---

## Personal Programs

### Docker

```bash
sudo pacman -S docker
sudo systemctl enable --now docker.service
sudo usermod -aG docker $USER
```

After installing docker, restart is recommended.

### open-webui

[open-webui github page](https://github.com/open-webui/open-webui)
Install with this command:

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

After installation, you can access Open WebUI at <http://localhost:3000>.

If you don't want `open-webui` run on startup, you can type

```bash
docker update --restart=no open-webui
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

#### Font

Install Nerd Font:

```bash
sudo pacman -S ttf-jetbrains-mono-nerd
```

#### Kitty

```bash
sudo pacman -S kitty
```

#### Neovim

```bash
sudo pacman -S neovim wl-clipboard
```

[How to enable clipboard support](https://askubuntu.com/questions/1486871/how-can-i-copy-and-paste-outside-of-neovim)

in `/etc/environment`, change `EDITOR` to `nvim`

```bash
EDITOR=nvim
```

#### Useful Terminal Programs

```bash
sudo pacman -S btop dust bat tldr lsd zoxide fastfetch tmux yazi
```

- `btop` - Resource monitor
- `dust` - Disk usage analyzer
- `bat` - Enhanced `cat`
- `tldr` - Simplified `man` pages
- `lsd` - Better `ls`
- `powertop` - Power usage analyzer
- `zoxide` - Better `cd`
- `fastfetch` - System information tool
- `tmux` - Terminal multiplexer
- `yazi` - Terminal file manager

#### Other Tools

- `bottom` - Also a resource monitor
- `eza` - Also better `ls`
- `atuin` - Command history manager

#### Looking cool

- pipes.sh
- cava
- cmatrix
- lolcat
- fortune
- cowsay
- cbonsai
- onefetch

#### TMUX Configuration

[oh-my-tmux](https://github.com/gpakosz/.tmux?tab=readme-ov-file)

#### Dotfiles with [Chezmoi](https://www.chezmoi.io)

Download chezmoi first

```bash
sudo pacman -S chezmoi
```

```bash
chezmoi init git@github.com:$GITHUB_USERNAME/dotfiles.git
```

To apply dotfiles interactively, simply run

```bash
chezmoi -v apply
```

### Internet Browser

```bash
sudo pacman -S vivaldi
```

### Disk Usage analyzer

```bash
sudo pacman -S filelight
```

### Obsidian

```bash
sudo pacman -S obsidian
```

- automatically change input method in vim mode
download [plugin](https://www.obsidianstats.com/plugins/vim-im-select)
set `default IM` to `keyboard-us`
`obtaining command` to `/usr/bin/fcitx5-remote`
`switching command` to `/usr/bin/fcitx5-remote -s {im}`

### LibreOffice

```bash
sudo pacman -S libreoffice-still
```

Goto View -> User interface and Select `Tabbed`, apply all.
Goto Settings -> Load/Save -> General and change default file format to xlsx/pptx/docx.

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

#### Kakaotalk

Download cjk fonts in Bottles -> dependencies.
Download the latest version from [KakaoTalk](https://www.kakaocorp.com/page/service/service/KakaoTalk) and install it using Bottles.
`Cafe` runner is better than `Soda` runner for KakaoTalk.

### Zapret (DPI Circumvention)

#### Download from AUR(recommended)

```bash
yay -S zapret
```

#### Download Manually

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

## Theme & Appearance

### GRUB Configuration


#### Change GRUB Resolution

[Reference](https://askubuntu.com/questions/54067/how-to-safely-change-grub2-screen-resolution)

1. Type `videoinfo` in the GRUB console to check available resolutions.
2. Open `/etc/default/grub` and modify `GRUB_GFXMODE` with your desired resolution.
3. run this command to set the resolution:

```bash
sudo update-grub
```

#### Change GRUB Background

1. open `/etc/default/grub` and modify `GRUB_BACKGROUND` with your desired image path.
2. then run this command to set the background:

```bash
sudo update-grub
```

Make sure the output of this command contains `Found background: /path/to/image.png`

#### Advanced GRUB theme configuration

Install `grub-customizer`

### SDDM(login screen)

#### Change Resolution

Run `xrandr` in your user session to find your display output name (e.g., HDMI-1, eDP-1) and supported resolutions.

Edit `/usr/share/sddm/scripts/Xsetup`

```bash
#!/bin/sh
xrandr --output eDP-1 --mode 1920x1200
```

Simply restart sddm

```bash
sudo systemctl restart sddm
```

#### [Astronut theme](https://github.com/Keyitdev/sddm-astronaut-theme/tree/master?tab=readme-ov-file)

Run automatic installation script

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/keyitdev/sddm-astronaut-theme/master/setup.sh)"
```

### KDE

#### KDE Sweet Theme

[YouTube Guide](https://www.youtube.com/watch?v=nmQn-JRwlo0)

#### laggy on Alt-Tab

goto settings -> window management -> task switcher
uncheck `show selected window`

#### Klassy

[Github link](https://github.com/paulmcauley/klassy)

```bash
yay -S klassy
```

#### KDE Wallpaper Engine

See [this plugin](https://github.com/catsout/wallpaper-engine-kde-plugin)

Install Dependencies

```bash
sudo pacman -S extra-cmake-modules plasma-framework5 gst-libav ninja \
base-devel mpv python-websockets qt5-declarative qt5-websockets qt5-webchannel vulkan-headers cmake
```

Build and Install

```bash
# Download source
git clone https://github.com/catsout/wallpaper-engine-kde-plugin.git
cd wallpaper-engine-kde-plugin

# Download submodule
git submodule update --init --force --recursive

# Configure, build and install
# 'USE_PLASMAPKG=ON': using kpackagetool tool to install plugin
cmake -B build -S . -GNinja -DUSE_PLASMAPKG=ON
cmake --build build
cmake --install build

# Install package (ignore if USE_PLASMAPKG=OFF for system-wide installation)
cmake --build build --target install_pkg
```

Goto Settings -> Wallpaper and Select Wallpaper Engine for `wallpaper type`.
Select Steamlibrary which is `~/.local/share/Steam` by default.

---

## TODO

- pinch to zoom
- Hybrid GPU (NVIDIA Prime or Optimus)
- hyprland
  
---

## Deprecated

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

### Hancom Office

```bash
yay -S hoffice
sudo mv /opt/hnc/hoffice11/Bin/qt{,.bak}
```

### WebCam Setting

Check Hardware Detection

```bash
lsusb
```

Most WebCams use the `uvcvideo` driver. Ensure it's loaded:

```bash
lsmod | grep uvcvideo
```

If not, load it:

```bash
sudo modprobe uvcvideo
```

Check kernel messages for errors:

```bash
dmesg | tail
```

```bash

Install Necessary Tools

```bash
sudo pacman -S v4l-utils cheese mpv
```

Permission

```bash
sudo usermod -aG video $USER
```

Test the webcam

```bash
mpv av://v4l2:/dev/video0
```

```bash
cheese
```
