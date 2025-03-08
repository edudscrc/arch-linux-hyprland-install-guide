# Arch Linux + Hyprland Installation Guide
Learn how to install Arch Linux with Hyprland. Minimal installation.

<ol>
<li>Download Arch Linux ISO</li>
<li>Burn it to a USB flash</li>
<li>Boot it (make sure <b>secure boot</b> is disabled).</li>
</ol>

### [Optional] Connect to Wi-Fi (on Ethernet, you don't have to do this):
<pre>
  $ iwctl
  [iwd]# station wlan0 get-networks
  [iwd]# station wlan0 connect <Network's name>
  [iwd]# exit
  $ ping archlinux.org
</pre>

### Synchronize packages:
<pre>
  $ pacman -Syy
</pre>

### Disk partitioning (using fdisk):
<pre>
  $ fdisk /dev/nvme0n1
  <i>[repeat this command until existing partitions are deleted]</i>
  Command (m for help): <b>d</b>
  Command (m for help): <b>d</b>
  Command (m for help): <b>d</b>

  <i>[create partition 1: <b>efi</b>]</i>
  Command (m for help): <b>n</b>
  Partition number (1-128, default 1): <b>Enter</b>
  First sector (..., default 2048): <b>Enter</b>
  Last sector ...: <b>+1024M</b>

  <i>[create partition 2: <b>swap</b>]</i>
  Command (m for help): <b>n</b>
  Partition number (2-128, default 2): <b>Enter</b>
  First sector (..., default ...): <b>Enter</b>
  Last sector ...: <b>+16G</b>

  <i>[create partition 3: <b>/</b>]</i>
  Command (m for help): <b>n</b>
  Partition number (3-128, default 3): <b>Enter</b>
  First sector (..., default ...): <b>Enter</b>
  Last sector ...: <b>Enter</b>

  <i>[write partitioning to disk]</i>
  Command (m for help): <b>w</b>
</pre>

### Create filesystems on created disk partitions:
<pre>
  $ mkfs.fat -F 32 /dev/nvme0n1p1
  $ mkswap /dev/nvme0n1p2
  $ mkfs -t ext4 /dev/nvme0n1p3
</pre>

### Mount all filesystems to /mnt:
<pre>
  $ mount /dev/nvme0n1p3 /mnt
  $ mkdir -p /mnt/boot/efi
  $ mount /dev/nvme0n1p1 /mnt/boot/efi
  $ swapon /dev/nvme0n1p2
</pre>

### Install essential packages into new filesystem and generate fstab:
<pre>
  <i>[install amd-ucode for AMD chipset or intel-ucode for INTEL chipset]</i>
  $ pacstrap /mnt base linux linux-firmware amd-ucode base-devel grub efibootmgr nano networkmanager
  $ genfstab -U /mnt > /mnt/etc/fstab
</pre>

### Basic configuration of new system:
<pre>
  $ arch-chroot /mnt
  <i>[uncomment your locales, i.e. 'en_US.UTF-8' or 'pt_BR.UTF-8']</i>
  $ nano /etc/locale.gen
  $ locale-gen
  $ echo "LANG=en_US.UTF-8" > /etc/locale.conf
  $ ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
  $ hwclock --systohc
</pre>

### Setup hostname and usernames:
<pre>
  $ echo <i>yourhostname</i> > /etc/hostname
  $ nano /etc/hosts
      <i>127.0.0.1 localhost</i>
      <i>::1 localhost</i>
      <i>127.0.1.1 yourhostname</i>
  $ useradd -m -G wheel,storage,power,audio,video -s /bin/bash yourusername
  $ passwd root
  $ passwd yourusername
  $ EDITOR=nano visudo
      <i>[uncomment following line]</i>
      <i>%wheel ALL=(ALL) ALL</i>
</pre>

### Enable networking:
<pre>
  $ systemctl enable NetworkManager
</pre>

### Install and setup GRUB:
<pre>
  $ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB --removable
  $ grub-mkconfig -o /boot/grub/grub.cfg
</pre>

### Exit chroot, unmount all disks and reboot:
<pre>
  $ exit
  $ umount -R /mnt
  $ reboot
</pre>

<h1 align="center">
    Setup Userspace and Hyprland
</h1>

### Activate time synchronization using NTP:
<pre>
  $ timedatectl set-ntp true
</pre>

### [Optional] Connect to Wi-Fi using nmcli:
<pre>
  $ nmcli device wifi connect &lt;Network's name&gt; password &lt;Network's password&gt;
</pre>

### Enable multilib (32-bit packages):
<pre>
  $ sudo nano /etc/pacman.conf
  <i>[uncomment [multilib] section]</i>
  $ sudo pacman -Sy
</pre>

### Install GPU drivers (and packages for gaming):
<b>AMD</b>
<pre>
  $ sudo pacman -S xf86-video-amdgpu mesa lib32-mesa
  $ sudo pacman -S vulkan-radeon lib32-vulkan-radeon
</pre>

<b>NVIDIA</b>
<pre>
  $ sudo pacman -S nvidia nvidia-utils lib32-nvidia-utils
</pre>

### [Optional] Run service that will discard unused blocks on mounted filesystems. This is useful for SSDs and thinly-provisioned storage:
<pre>
  $ sudo systemctl enable fstrim.timer
</pre>

### Install useful packages:
<pre>
  $ sudo pacman -S git btop wget fd curl unzip
  $ sudo pacman -S bash-completion openssh eza
  $ sudo pacman -S python python-gobject
  $ sudo pacman -S ripgrep fuse2
  curl -fsSL https://pyenv.run | bash
</pre>

### Install yay:
<pre>
  $ git clone https://aur.archlinux.org/yay.git
  $ cd yay
  $ makepkg -si
  $ cd ..
  $ rm -r yay
</pre>

### Install Hyprland and some useful packages:
<pre>
  $ sudo pacman -S hyprland kitty
  $ sudo pacman -S xdg-desktop-portal-hyprland
  $ sudo pacman -S dunst hyprpolkitagent
  $ sudo pacman -S qt5-wayland qt6-wayland
  $ sudo pacman -S waybar hyprpaper
  $ sudo pacman -S thunar mpv rofi
  $ sudo pacman -S gvfs tumbler ffmpegthumbnailer
  $ yay -S qimgv-git wlogout hyprshot
</pre>

### Install necessary fonts:
<pre>
  $ sudo pacman -S noto-fonts noto-fonts-emoji
  $ sudo pacman -S noto-fonts-cjk ttf-jetbrains-mono
  $ sudo pacman -S ttf-jetbrains-mono-nerd
  $ sudo pacman -S ttf-liberation otf-font-awesome
</pre>

### Install sound drivers and sound support:
<pre>
  $ sudo pacman -S pipewire wireplumber pipewire-audio
  $ sudo pacman -S pipewire-alsa pipewire-pulse
  $ sudo pacman -S lib32-pipewire pavucontrol
</pre>

### Enable bluetooth support:
<pre>
  $ sudo pacman -S bluez bluez-utils
  $ sudo systemctl enable bluetooth
</pre>

### Install additional softwares:
<pre>
  $ sudo pacman -S steam discord
  $ sudo pacman -S fastfetch qbittorrent
  $ yay -S spotify visual-studio-code-bin
  $ yay -S google-chrome
</pre>

### Reboot:
<pre>
  $ reboot
</pre>

### Enable Wayland support on Google Chrome:
<pre>
  <i>[In Google Chrome's address bar, type the following command]</i>
  chrome://flags/

  <b>Search for the flag "Preferred Ozone platform" and set it to "Wayland"</b>
</pre>

### Install HDR utilities and how to use it:
<pre>
  $ sudo pacman -S gamescope
  $ yay -S vk-hdr-layer-kwin6-git
  
  <i>[In Steam, to enable HDR for a single game, set the following Launch options]</i>
  DXVK_HDR=1 gamescope -f -W 2560 -H 1440 --force-grab-cursor --hdr-enabled -- %command%
  
  <i>[To play a video with HDR using MPV, do the following]</i>
  <i>[1. Add the following environment variable]</i>
  ENABLE_HDR_WSI=1
  <i>[2. Play the video using the command below]</i>
  $ mpv --vo=gpu-next --target-colorspace-hint --gpu-api=vulkan --gpu-context=waylandvk "path/to/video"
</pre>

### Fix cedilla on us-intl with dead keys:
<pre>
  $ sudo pacman -S fcitx5
  <i>[Add it to exec-once on hyprland.conf]</i>
  exec-once = fcitx5

  On chrome://flags, set these values:
  Preferred Ozone platform - Wayland
  Wayland text-input-v3 - Enabled
</pre>

### Add permission to serial ports:
<pre>
  $ usermod -a -G uucp $USER
  $ reboot
</pre>

### How to change DNS with NetworkManager:
<pre>
  $ sudo nano /etc/NetworkManager/conf.d/dns.conf
  <i>[Add the following content]</i>
  [main]
  dns=none

  $ nmcli connection modify "Wired connection 1" ipv4.dns "1.1.1.1, 1.0.0.1"
  $ nmcli connection modify "Wired connection 1" ipv6.dns "2606:4700:4700::1111, 2606:4700:4700::1001"

  $ nmcli connection modify &lt;YOUR_SSID&gt; ipv4.dns "1.1.1.1, 1.0.0.1"
  $ nmcli connection modify &lt;YOUR_SSID&gt; ipv6.dns "2606:4700:4700::1111, 2606:4700:4700::1001"

  $ nmcli connection modify "Wired connection 1" ipv4.ignore-auto-dns yes
  $ nmcli connection modify "Wired connection 1" ipv6.ignore-auto-dns yes
  $ nmcli connection modify &lt;YOUR_SSID&gt; ipv4.ignore-auto-dns yes
  $ nmcli connection modify &lt;YOUR_SSID&gt; ipv6.ignore-auto-dns yes

  $ sudo systemctl restart NetworkManager
</pre>

### How to use nmcli:
<pre>
  $ nmcli device wifi list
  $ nmcli device wifi connect &lt;SSID_or_BSSID&gt; password &lt;Network's password&gt;
  $ nmcli connection show
  $ nmcli connection delete &lt;Network's name&gt;
  $ nmcli connection up &lt;Network's name&gt;
  $ nmcli device
</pre>

### How to use bluetoothctl:
<pre>
  $ bluetoothctl
  [bluetooth]# power on
  [bluetooth]# scan on
  [bluetooth]# pair XX:XX:XX:XX:XX:XX
  [bluetooth]# trust XX:XX:XX:XX:XX:XX
  [bluetooth]# connect XX:XX:XX:XX:XX:XX
  [bluetooth]# info XX:XX:XX:XX:XX:XX
  [bluetooth]# disconnect XX:XX:XX:XX:XX:XX
  [bluetooth]# remove XX:XX:XX:XX:XX:XX
  [bluetooth]# devices
</pre>
