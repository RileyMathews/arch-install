# Arch Install
My Arch install steps

# Minimal Install
## Verify boot mode
```
cat /sys/firmware/efi/fw_platform_size
```
should return `64`

## Verify internet connectivity

`ping archlinux.org`

## Disk Partitions
```
lsblk
```
To determine disk to use
```
gdisk /dev/*disk*
```

For partition instructions '"' means accept the default
### efi partition
```
n -> " -> " -> +1G -> ef00
```

### swap partition
sub `+4g` for how many gigabytes of space you want to commit to swap.
```
n -> " -> " -> +4G -> 8200
```

### root partition
```
n -> " -> " -> " -> 8304
```

```
w -> y
```
To commit the partitions

## Format partitions
```
mkfs.ext4 /dev/root_partition
mkfs.fat -F 32 /dev/efi_partition
mkswap /dev/swap_partition
```

## Mount partitions
```
mount /dev/root_partition /mnt
mount --mkdir /dev/efi_partition /mnt/boot
swapon /dev/swap_partition
```

## Install Arch
```
pacstrap -K /mnt base linux linux-firmware neovim networkmanager grub efibootmgr sudo
```
## Preserve Mounts
```
genfstab -U /mnt >> /mnt/etc/fstab
```

## Chroot
```
arch-chroot /mnt
```

### general configs
#### time
```
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```

#### locale
First edit `/etc/locale.gen` and uncomment the line `en_US.UTF-8 UTF-8`
```
locale-gen
```
```
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

#### network
```
echo *hostname* > /etc/hostname
systemctl enable NetworkManager
```

#### user setup
set root password
```
passwd
```

```
pacman -S zsh
useradd -m -G wheel -s /bin/zsh rileymathews
passwd rileymathews
```

```
EDITOR=nvim visudo
```
Then uncomment line
```
wheel ALL=(ALL:ALL) ALL
```

## Microcode
```
pacman -S amd-ucode/intel-ucode
```

## Grub
```
grub-install --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg
```

# Graphical Setup
These steps will give you a minimal i3 environment that you can start after logging into a tty
## Packages
```
sudo pacman -S xorg xorg-xinit i3 dmenu
```
Select all packages for xorg. Only i3-wm is required for i3 but accepting all will make the default status bar work out of the box

```
cp /etc/X11/xinit/xinitrc ~/.xinitrc
```

Then modify the `~/.xinitrc` file by replacing everything after `twm` at the end of the file with
```
startx i3
```
