<h1 align="center">🐧Arch Linux + Hyprland Installation Guide🐧</h1>
<h2 align="center">Learn how to install Arch Linux with Hyprland. Minimal installation.</h2>

<ol>
  <li>Download Arch Linux ISO from <a href=https://archlinux.org/download/>official Arch Linux download page</a>.</li>
  <li>Burn it to a USB flash drive. You can use Rufus, Ventoy or dd command.</li>
  <li>Restart your computer and configure your BIOS/UEFI to boot from the USB drive. Make sure <b>Secure Boot</b> is disabled.</li>
</ol>

### [Optional] Connect to Wi-Fi (on Ethernet, you don't have to do this):
<pre>
  # iwctl
  [iwd]# device list
  [iwd]# station <i>DEVICE_NAME</i> get-networks
  [iwd]# station <i>DEVICE_NAME</i> connect <i>YOUR_NETWORK_NAME</i>
  [iwd]# exit
  # ping archlinux.org
</pre>

### Synchronize the system clock:
<pre>
  # timedatectl set-ntp true
</pre>

### Disk partitioning (using fdisk):
<pre>
  # lsblk
  
  # fdisk /dev/<i>YOUR_DISK</i>
  
  <i>[Repeat this command until all existing partitions are deleted]</i>
  Command (m for help): <b>d</b>
  Command (m for help): <b>d</b>
  Command (m for help): <b>d</b>

  <i>[Create partition 1: <b>efi</b>]</i>
  Command (m for help): <b>n</b>
  Partition number (1-128, default 1): <b>Enter</b>
  First sector (..., default 2048): <b>Enter</b>
  Last sector ...: <b>+1G</b>

  <i>[Create partition 2: <b>swap</b>]</i>
  Command (m for help): <b>n</b>
  Partition number (2-128, default 2): <b>Enter</b>
  First sector (..., default ...): <b>Enter</b>
  Last sector ...: <b>+8G</b>

  <i>[Create partition 3: <b>/</b>]</i>
  Command (m for help): <b>n</b>
  Partition number (3-128, default 3): <b>Enter</b>
  First sector (..., default ...): <b>Enter</b>
  Last sector ...: <b>Enter</b>

  <i>[Write all changes]</i>
  Command (m for help): <b>w</b>
</pre>

### Create filesystems on created disk partitions:
<pre>
  # lsblk
  
  # mkfs.fat -F 32 /dev/<i>YOUR_DISK_PARTITION_1</i>
  # mkswap /dev/<i>YOUR_DISK_PARTITION_2</i>
  # mkfs -t ext4 /dev/<i>YOUR_DISK_PARTITION_3</i>
</pre>

### Mount all filesystems to /mnt:
<pre>
  # mount /dev/<i>YOUR_DISK_PARTITION_3</i> /mnt
  # mkdir -p /mnt/boot/efi
  # mount /dev/<i>YOUR_DISK_PARTITION_1</i> /mnt/boot/efi
  # swapon /dev/<i>YOUR_DISK_PARTITION_2</i>
</pre>

### Update your mirrors to get faster download speed:
<pre>
  # reflector --country <i>YOUR_COUNTRY</i> --latest 5 --save /etc/pacman.d/mirrorlist --sort rate --verbose
</pre>

### Install essential packages into new filesystem and generate fstab:
<pre>
  <i>[Install <b>amd-ucode</b> for AMD chipset or <b>intel-ucode</b> for INTEL chipset]</i>
  # pacstrap -K /mnt base linux linux-firmware amd-ucode base-devel grub efibootmgr nano networkmanager
  # genfstab -U /mnt >> /mnt/etc/fstab
</pre>

### Set your timezone, language and synchronize clock:
<pre>
  # arch-chroot /mnt
  # ln -sf /usr/share/zoneinfo/<i>YOUR_REGION</i>/<i>YOUR_CITY</i> /etc/localtime
  # hwclock --systohc
  <i>[Uncomment your locales, e.g., 'en_US.UTF-8', 'pt_BR.UTF-8']</i>
  # nano /etc/locale.gen
  # locale-gen
  <i>[Set the default system language and encoding]</i>
  # echo "LANG=en_US.UTF-8" > /etc/locale.conf
</pre>

### Setup hostname and usernames:
<pre>
  # echo <i>YOUR_HOSTNAME</i> > /etc/hostname
  # nano /etc/hosts
      <i>127.0.0.1 localhost</i>
      <i>::1 localhost</i>
      <i>127.0.1.1 <i>YOUR_HOSTNAME</i></i>
  # useradd -m -G wheel,storage,power,audio,video,uucp -s /bin/bash <i>YOUR_USERNAME</i>
  # passwd root
  # passwd <i>YOUR_USERNAME</i>
  # EDITOR=nano visudo
      <i>[Find and uncomment the following line]</i>
      <i>%wheel ALL=(ALL) ALL</i>
</pre>

### Enable networking:
<pre>
  # systemctl enable NetworkManager
</pre>

### Install and setup GRUB:
<pre>
  # grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB --removable
  # grub-mkconfig -o /boot/grub/grub.cfg
</pre>

### Exit chroot, unmount all disks and reboot:
<pre>
  # exit
  # umount -R /mnt
  # poweroff
  <i>[After shutting it down, remove the USB flash drive and turn it on again]</i>
</pre>

<h1 align="center">🎉 Congratulations on Installing Arch Linux! 🎉</h1>
<h2 align="center">Setup Userspace and Hyprland</h2>

<ul>
  <li>Login as the user you have created.</li>
</ul>

### Activate time synchronization using NTP:
<pre>
  $ timedatectl set-ntp true
</pre>

### [Optional] Connect to Wi-Fi using nmcli:
<pre>
  $ nmcli device wifi list
  $ nmcli device wifi connect <i>YOUR_NETWORK_NAME</i> --ask
</pre>

### Enable multilib (32-bit packages):
<pre>
  $ sudo nano /etc/pacman.conf
  <i>[Find and uncomment [multilib] section (the lines below)]</i>
  <i>[multilib]</i>
  <i>Include = /etc/pacman.d/mirrorlist</i>
  $ sudo pacman -Syyu
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

<b>INTEL</b>
<pre>
  $ sudo pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel
</pre>

### [Optional] Enable service that will discard unused blocks on mounted filesystems (once per week). This is useful for SSDs:
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

### Install necessary fonts:
<pre>
  $ sudo pacman -S noto-fonts noto-fonts-emoji
  $ sudo pacman -S noto-fonts-cjk ttf-liberation
  $ sudo pacman -S otf-font-awesome
  $ sudo pacman -S ttf-jetbrains-mono ttf-jetbrains-mono-nerd
</pre>

### Install sound drivers and sound support:
<pre>
  $ sudo pacman -S pipewire wireplumber pipewire-audio
  $ sudo pacman -S pipewire-alsa pipewire-pulse
  $ sudo pacman -S pipewire-jack
  <i>[When installing pipewire-jack, if it asks to replace jack-2, replace it]</i>
  $ sudo pacman -S lib32-pipewire pavucontrol
</pre>

### [Optional] Enable Bluetooth support:
<pre>
  $ sudo pacman -S bluez bluez-utils
  $ sudo systemctl enable bluetooth
</pre>

### Install Hyprland and some useful packages:
<pre>
  $ sudo pacman -S hyprland kitty ly
  $ sudo pacman -S xdg-desktop-portal-hyprland
  $ sudo pacman -S dunst hyprpolkitagent
  $ sudo pacman -S qt5-wayland qt6-wayland
  $ sudo pacman -S waybar hyprpaper
  $ sudo pacman -S yazi mpv rofi
  $ sudo pacman -S hyprpicker hyprlock
  $ sudo pacman -S thunar tumbler gvfs ffmpegthumbnailer
  $ yay -S qimgv-git hyprshot
  $ yay -S wl-gammarelay-rs
  $ sudo systemctl enable ly.service
</pre>

### [Optional] Install additional softwares:
<pre>
  $ sudo pacman -S steam discord spotify-launcher
  $ sudo pacman -S qbittorrent firefox obs-studio
  $ sudo pacman -S obsidian neovim zathura
  $ sudo pacman -S zathura-pdf-poppler
  $ sudo pacman -S kdenlive audacity
  $ yay -S visual-studio-code-bin
</pre>

### Reboot:
<pre>
  $ reboot
</pre>

<h1 align="center">🎉 Congratulations on Finishing the Installation! 🎉</h1>
<h2 align="center">Troubleshooting and additional stuff</h2>

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
