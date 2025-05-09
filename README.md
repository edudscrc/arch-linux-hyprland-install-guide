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

### Update your mirrors to get faster download speed on pacstrap:
<pre>
  $ reflector --country Brazil --protocol https --latest 5 --save /etc/pacman.d/mirrorlist --sort rate --verbose
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
  <i>[uncomment your locale, i.e. 'en_US.UTF-8' or 'pt_BR.UTF-8']</i>
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
  $ useradd -m -G wheel,storage,power,audio,video,uucp -s /bin/bash yourusername
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
  $ sudo pacman -S git btop wget fd unzip
  $ sudo pacman -S bash-completion openssh eza
  $ sudo pacman -S python python-gobject
  $ sudo pacman -S ripgrep fuse2
  $ sudo pacman -S reflector less sassc
</pre>

### Install yay:
<pre>
  $ git clone https://aur.archlinux.org/yay.git
  $ cd yay
  $ makepkg -si
  $ cd ..
  $ rm -rf yay
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
  $ sudo pacman -S hyprpicker hyprlock
  $ sudo pacman -S dysk
  $ yay -S qimgv-git hyprshot
  $ yay -S wl-gammarelay-rs
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
  $ sudo pacman -S pipewire-jack  <i># If it asks to replace jack-2, replace it.</i>
  $ sudo pacman -S lib32-pipewire pavucontrol
</pre>

### Enable bluetooth support:
<pre>
  $ sudo pacman -S bluez bluez-utils
  $ sudo systemctl enable bluetooth
</pre>

### Install additional softwares:
<pre>
  $ sudo pacman -S steam discord spotify-launcher
  $ sudo pacman -S fastfetch qbittorrent firefox
  $ sudo pacman -S obsidian nwg-look
  $ yay -S visual-studio-code-bin
</pre>

### Reboot:
<pre>
  $ reboot
</pre>

<h1 align="center">
    Troubleshooting and additional stuff
</h1>

### Change icon theme to Papirus:
<pre>
  $ sudo pacman -S papirus-icon-theme
  $ yay -S papirus-folders-git
  $ papirus-folders -C violet --theme Papirus-Dark

  <i>Use nwg-look to change icon theme</i>
</pre>

### Compile Stremio from source:
<pre>
  $ sudo pacman -S wget librsvg nodejs mpv openssl make gcc qt5-base qt5-webengine qt5-quickcontrols qt5-quickcontrols2 --needed
  $ git clone --recurse-submodules https://github.com/Stremio/stremio-shell.git
  $ qmake
  $ make -f release.makefile
  $ sudo make -f release.makefile install
  $ sudo ./dist-utils/common/postinstall
</pre>

### Disable the loud beep sound in TTY:
<pre>
  $ sudo rmmod pcspkr
</pre>

### Fix cedilla on us-intl with dead keys:
<pre>
  $ sudo nano /usr/lib/gtk-3.0/3.0.0/immodules.cache
  <i>[Find the lines starting with "cedilla" "Cedilla" and add :en to the line]</i>

  $ sudo sed -i /usr/share/X11/locale/en_US.UTF-8/Compose -e 's/ć/ç/g' -e 's/Ć/Ç/g'

  <i>[Add the following environment variables]</i>
  GTK_IM_MODULE=cedilla
  QT_IM_MODULE=cedilla
</pre>

### How to change DNS with NetworkManager:
<pre>
  <i>[List your connections with the command below]</i>
  $ nmcli connection

  $ nmcli connection edit Wired\ connection\ 1
  nmcli> set ipv4.ignore-auto-dns yes
  nmcli> set ipv4.dns 1.1.1.1
  nmcli> set ipv4.dns 1.0.0.1
  nmcli> set ipv6.ignore-auto-dns yes
  nmcli> set ipv6.dns 2606:4700:4700::1111
  nmcli> set ipv6.dns 2606:4700:4700::1001

  nmcli> save persistent
  nmcli> quit

  $ sudo systemctl restart NetworkManager

  <i>[You can check if the settings were applied]</i>
  $ cat /etc/resolv.conf
</pre>

### Incorrect password (when you are sure you are typing it correctly):
<pre>
  $ su
  # faillock --reset
</pre>
