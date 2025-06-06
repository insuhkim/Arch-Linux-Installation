# Arch Linux Installation & Configuration Guide

---

This guide provides step-by-step instructions for installing and configuring Arch Linux using EndeavourOS, KDE, and Wayland. It includes post-installation tasks, advanced configurations, and personal program setups.

---

## Download ISO & Make Bootable USB

1. Download EndeavourOS ISO from [here](https://endeavouros.com).
2. Create a bootable USB using:

   - [balenaEtcher](https://etcher.balena.io)
   - [Ventoy](https://ventoy.net/en/index.html)

3. Ensure **SECURE BOOT is disabled** in BIOS.
4. Boot the device with the USB plugged in, enter the boot menu, and select EndeavourOS.

> **Tip:** Before installing EndeavourOS, update mirrors in the `online installation` option to speed up the installation process.

---

## After Installation

### Create a Snapshot

Right after a successful installation, create a snapshot as a backup:

```bash
sudo pacman -S timeshift
sudo timeshift --create
```

If you installed a Desktop Environment (DE), you can use the GUI application `timeshift-launcher`.

---

### Enable Bluetooth

Enable and start the Bluetooth service:

```bash
sudo systemctl enable --now bluetooth
```

---

### TLP Battery Management for Laptops

Follow the [TLP Official Installation Guide](https://linrunner.de/tlp/installation/arch.html).

Install TLP and enable the required services:

```bash
sudo pacman -S tlp tlp-rdw
sudo systemctl enable tlp.service
sudo systemctl enable NetworkManager-dispatcher.service
sudo systemctl mask systemd-rfkill.service systemd-rfkill.socket
```

> **Note:** If TLP conflicts with `power-profiles-daemon`, remove the latter.

---

### Fix Audio Issues

There's a speaker issue with Lenovo Yoga.
[Audio is too loud when volume is set greater than 0 on Lenovo Yoga](https://github.com/alsa-project/alsa-lib/issues/366)
To solve this problem, follow these steps:

1. In `/usr/share/alsa-card-profile/mixer/paths/analog-output.conf.common`, add these three lines above the `Element PCM`:

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

2. Add the following parameters to `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub`:

```bash
snd_hda_intel.dmic_detect=0 snd_hda_intel.model=lenovo
```

3. Update GRUB and reboot:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
reboot
```

---

### Chaotic AUR

> **Note:** The key can change, please check the [Chaotic AUR documentation](https://aur.chaotic.cx/docs).

Run these commands to install Chaotic AUR:

```bash
sudo pacman-key --recv-key 3056513887B78AEB --keyserver keyserver.ubuntu.com
sudo pacman-key --lsign-key 3056513887B78AEB

sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst'
sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'
```

Append this line to `/etc/pacman.conf`:

```bash
[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
```

---

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

---

### Korean Input Configuration

- Install `fcitx5` for Hangul support:

```bash
sudo pacman -S fcitx5-im fcitx5-hangul
fcitx5-configtool
```

- Inside the config tool, add the "hangul" input method.
- (Optional) Change `Ctrl+Space` to `Shift+Space` in global options.
- Set the input method to `fcitx5` in system settings.
- (Optional) In keyboard settings, map `Right Alt` to Hangul and `Right Ctrl` to Hanja.
- Add the following line to `/etc/environment`:

```bash
XMODIFIERS=@im=fcitx
```

Refer to [this guide](https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#KDE_Plasma) for more details.

---

### Make `update-grub` Command

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

## Personal Programs

### Docker

Install Docker and enable the service:

```bash
sudo pacman -S docker
sudo systemctl enable --now docker.service
sudo usermod -aG docker $USER
```

After installing Docker, a restart is recommended.

---

### open-webui

[open-webui GitHub page](https://github.com/open-webui/open-webui)

Install with this command:

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

After installation, you can access Open WebUI at <http://localhost:3000>.

If you don't want `open-webui` to run on startup, you can type:

```bash
docker update --restart=no open-webui
```

---

### WinApps

Clone the project and follow the [installation steps](https://github.com/winapps-org/winapps?tab=readme-ov-file#installation).
The descriptions below are only a rough guide.

1. Install Docker Engine and Docker Compose:

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
   Follow this [guide](https://github.com/winapps-org/winapps/blob/main/docs/docker.md#installing-windows).

3. Install Dependencies:

```bash
sudo pacman -Syu --needed -y curl dialog freerdp git iproute2 libnotify gnu-netcat
```

4. Create a WinApps Configuration File at `~/.config/winapps/winapps.conf`.

5. Test FreeRDP:

```bash
xfreerdp3 /u:"MyWindowsUser" /p:"MyWindowsPassword" /v:192.168.122.2 /cert:tofu
```

6. Run FreeRDP:

```bash
bash <(curl https://raw.githubusercontent.com/winapps-org/winapps/main/setup.sh)
```

---

### Terminal Setup

#### ZSH

```bash
sudo pacman -S zsh
chsh -s /bin/zsh
# Log out and log back in for the changes to take effect
echo $SHELL # It should output /bin/zsh
```

#### Font

Install Nerd Font:

```bash
sudo pacman -S ttf-jetbrains-mono-nerd
```

#### Kitty

Install Kitty terminal emulator:

```bash
sudo pacman -S kitty
```

#### Neovim

Install Neovim and clipboard support:

```bash
sudo pacman -S neovim wl-clipboard
```

[How to enable clipboard support](https://askubuntu.com/questions/1486871/how-can-i-copy-and-paste-outside-of-neovim)

In `/etc/environment`, change `EDITOR` to `nvim`:

```bash
EDITOR=nvim
```

#### Useful Terminal Programs

Install useful terminal programs:

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

#### Silly Programs

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

```bash
cd
git clone --single-branch https://github.com/gpakosz/.tmux.git
ln -s -f .tmux/.tmux.conf
cp .tmux/.tmux.conf.local .
```

#### Dotfiles with [Chezmoi](https://www.chezmoi.io)

Download chezmoi first:

```bash
sudo pacman -S chezmoi
```

Initialize chezmoi with your dotfiles repository:

```bash
chezmoi init git@github.com:$GITHUB_USERNAME/dotfiles.git
```

To apply dotfiles interactively, simply run:

```bash
chezmoi -v apply
```

---

### Internet Browser

Install Zen Browser:

```bash
sudo pacman -S zen-browser
```

- Install [Nebular theme](https://github.com/JustAdumbPrsn/Nebula-A-Minimal-Theme-for-Zen-Browser?tab=readme-ov-file).
- Install [KDE force blur](https://github.com/taj-ny/kwin-effects-forceblur):

```bash
yay -S kwin-effects-forceblur
```

- Install [Transparent-zen](https://github.com/frostybiscuit/transparent-zen?tab=readme-ov-file) [here](https://addons.mozilla.org/en-US/firefox/addon/transparent-zen/).

---

### Disk Usage Analyzer

Install Filelight:

```bash
sudo pacman -S filelight
```

---

### Obsidian

Install Obsidian:

```bash
sudo pacman -S obsidian
```

- Automatically change input method in vim mode:
  - Download [plugin](https://www.obsidianstats.com/plugins/vim-im-select).
  - Set `default IM` to `keyboard-us`.
  - Set `obtaining command` to `/usr/bin/fcitx5-remote`.
  - Set `switching command` to `/usr/bin/fcitx5-remote -s {im}`.

---

### LibreOffice

Install LibreOffice:

```bash
sudo pacman -S libreoffice-still
```

- Go to View -> User Interface and select `Tabbed`, apply all.
- Go to Settings -> Load/Save -> General and change the default file format to xlsx/pptx/docx.

---

### Bottles (For Running Windows Apps)

Install Bottles:

```bash
sudo pacman -Syu
sudo pacman -S flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install flathub com.usebottles.bottles
```

For file access permissions, install Flatseal:

```bash
flatpak install flathub com.github.tchx84.Flatseal
```

Enable "All User Files" in Flatseal for Bottles.
Also, add this path to add a desktop entry for Bottles:

```bash
flatpak override com.usebottles.bottles --user --filesystem=xdg-data/applications
flatpak override com.usebottles.bottles --user --filesystem=~/.local/share/applications
mkdir -p ~/.local/share/applications
```

#### Kakaotalk

Download cjk fonts in Bottles -> dependencies.
Download the latest version from [KakaoTalk](https://www.kakaocorp.com/page/service/service/KakaoTalk) and install it using Bottles.
`Cafe` runner is better than `Soda` runner for KakaoTalk.

After installation, set KakaoTalk font to Korean font in settings.

---

### Zapret (DPI Circumvention)

#### Download from AUR (recommended)

```bash
yay -S zapret
```

#### Download Manually

1. Download the latest release from [Zapret](https://github.com/bol-van/zapret/releases).
2. Unzip and move it to `/opt/zapret`.
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

## Theme & Appearance

### GRUB Configuration

#### Change GRUB Resolution

[Reference](https://askubuntu.com/questions/54067/how-to-safely-change-grub2-screen-resolution)

1. Type `videoinfo` in the GRUB console to check available resolutions.
2. Open `/etc/default/grub` and modify `GRUB_GFXMODE` with your desired resolution.
3. Run this command to set the resolution:

```bash
sudo update-grub
```

#### Change GRUB Background

1. Prepare an image that you want to set as a background.
2. Run this command to convert the image to the correct format:

```bash
sudo magick /path/to/image.png -resize 1920x1080 -depth 24 /boot/grub/background.png
```

3. Open `/etc/default/grub` and modify `GRUB_BACKGROUND`:

```config
GRUB_BACKGROUND="/boot/grub/background.png"
```

4. Run this command to update GRUB:

```bash
sudo update-grub
```

Make sure the output of this command contains `Found background: /path/to/image.png`.

#### Advanced GRUB Theme Configuration

Install `grub-customizer`.

---

### SDDM (Login Screen)

#### Change Resolution

Run `xrandr` in your user session to find your display output name (e.g., HDMI-1, eDP-1) and supported resolutions.

Edit `/usr/share/sddm/scripts/Xsetup`:

```bash
#!/bin/sh
xrandr --output eDP-1 --mode 1920x1200
```

Restart SDDM:

```bash
sudo systemctl restart sddm
```

#### [Astronut Theme](https://github.com/Keyitdev/sddm-astronaut-theme/tree/master?tab=readme-ov-file)

Run the automatic installation script:

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/keyitdev/sddm-astronaut-theme/master/setup.sh)"
```

---

### KDE

#### KDE Sweet Theme

[YouTube Guide](https://www.youtube.com/watch?v=nmQn-JRwlo0)

#### Widgets

- [apdatifier](https://github.com/exequtic/apdatifier)
- [panel colorizer](https://store.kde.org/p/2130967)

#### Laggy on Alt-Tab

Go to Settings -> Window Management -> Task Switcher and uncheck `Show selected window`.

#### Klassy

[GitHub link](https://github.com/paulmcauley/klassy)

Install Klassy:

```bash
yay -S klassy
```

#### KDE Wallpaper Engine

See [this plugin](https://github.com/catsout/wallpaper-engine-kde-plugin).

Install Dependencies:

```bash
sudo pacman -S extra-cmake-modules plasma-framework5 gst-libav ninja \
base-devel mpv python-websockets qt5-declarative qt5-websockets qt5-webchannel vulkan-headers cmake
```

Build and Install:

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

Go to Settings -> Wallpaper and select Wallpaper Engine for `wallpaper type`.
Select Steam library, which is `~/.local/share/Steam` by default.

---

## TODO

- Hybrid GPU (NVIDIA Prime or Optimus)

---

## Deprecated

### Terminal Settings

#### Zsh & Oh-My-Zsh

Install Zsh and Oh-My-Zsh:

```bash
sudo pacman -S zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Set the default shell to Zsh by logging out and back in.

#### Powerlevel10k Theme

Install Powerlevel10k theme:

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
```

Modify `~/.zshrc`:

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

---

### Hancom Office

Install Hancom Office:

```bash
yay -S hoffice
sudo mv /opt/hnc/hoffice11/Bin/qt{,.bak}
```

---

### WebCam Setting

Check Hardware Detection:

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

Install Necessary Tools:

```bash
sudo pacman -S v4l-utils cheese mpv
```

Permission:

```bash
sudo usermod -aG video $USER
```

Test the webcam:

```bash
mpv av://v4l2:/dev/video0
```

```bash
cheese
```

---

## How to Do Things in Terminal

If you installed with KDE or another Desktop Environment, you can use the GUI.
This is for people who didn't install DE or are just curious about things.

### Mount USB

We will use `udisks2` to automatically mount USB devices.
Make sure `udisks2` exists.

- Plug in your USB drive.
- Check the device name of your USB drive by running:

```bash
lsblk
```

This will list all block devices. In most cases, your USB drive will likely be named something like `/dev/sdb1` or `/dev/sdc1` or `/dev/sda1`,
the format of `/dev/sdXN`.

- To mount a USB device:

```bash
udiskctl mount -b /dev/sdXN
```

The USB drive will be mounted in `/run/media/$USER`.

### Bluetooth Connection

You can use `bluetoothctl`, but it's a bit complicated. Use `bluetui` instead.
You can install `bluetui` with `pacman`.

In `bluetui`, `s` is for scan, `p` is for pair. Use `tab` to navigate through the menu.

### Wifi Connection

#### `iwctl`

You can configure wifi with `iwctl`, but using `nmtui` in the next line is much easier.

- Display your Wifi stations:

```bash
iwctl station list
```

Mostly, the station name is `wlan0`.

- Start looking for networks with a station:

```bash
iwctl station station_name scan
```

Note that `scan` doesn't give any output.

- Display the networks found by a station:

```bash
iwctl station station_name get-networks
```

- Connect to a network with a station, if credentials are needed they will be asked:

```bash
iwctl station station_name connect network_name
```

#### `nmtui`

`nmtui` stands for Network Manager TUI.
