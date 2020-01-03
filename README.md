### ArchLinux|Hardened Encrypted Luks Lvm Ext4 Systemd-Boot AppArmor All The Traffic Trhough TOR. :) WikiLeaks Like Linux. ###

Needs / Deps 4 This Task .

https://sourceforge.net/projects/revenge-installer/

pacman -S apparmor haveged lvm2 cryptsetup e2fsprogs parted ; systemctl enable --now apparmor; systemctl enable --now haveged 
shred -v -n1 /dev/sdX Or Optional: Overwrite LUKS Partition with Random Data

gdisk /dev/sda

+---------------------------+------------------------+-----------------------+
| Partition name            | Partition purpose      | Filesystem type       |
+---------------------------+------------------------+-----------------------+
| /dev/sda1                 | EFI system partition   | fat32  code: ef00     |             
|                           | LUKS container         | LUKS   code  8e00     |
| |-> /dev/mapper/lvmcrypt  | LVM container          | LVM                   |
|  |-> /dev/vg01/root       | Root partition         | ext4                  |
|  |-> /dev/vg01/swap       | Swap partition         | swap                  |
+---------------------------+------------------------+-----------------------+

parted >>>

name 1 EFI
set 1 esp on
name 2 lvm
set 2 lvm on

mkfs.fat -F32 /dev/sda1
modprobe dm-crypt

Encrypting the LVM Physical Volume Partition
Optimized for security:

cryptsetup -v -c serpent-xts-plain64 -s 512 --hash whirlpool --iter-time 5000 --use-random luksFormat /dev/sda2
Set PASS
cryptsetup open --type luks /dev/sda2 lvm
Open With PASS


Create LVM Container

ls /dev/mapper/lvm
pvcreate /dev/mapper/lvm
vgcreate volume /dev/mapper/lvm
lvcreate -L2G volume -n swap
lvcreate -l 100%FREE volume -n root
pvsacn ; vgscan ; lvscan 
pvchange -ay volume


FORMAT
mkfs.ext4 /dev/mapper/volume-root
mkswap /dev/mapper/volume-swap
mount /dev/mapper/volume-root /mnt
swapon /dev/mapper/volume-swap

NOW Install ARCH In The Installer.


Before REBOOT CHROOT.

blkid -s UUID -o value /dev/sda2 > ~/uuid
blkid

edit /etc/fstab
/dev/mapper/volume-swap    swap    swap    defaults    0 0
/dev/mapper/volume-root    /       ext4    rw,default  0 1

edit /etc/crypttab
volume    UUID=<UUID>    none    luks



Edit the /etc/mkinitcpio.conf. Look for the HOOKS variable and move keyboard to before the filesystems and add encrypt and lvm2 after keyboard. Like: ADD drm md_mod sd_mod dm-crypt to modules 

HOOKS="base udev autodetect modconf block keyboard encrypt lvm2 filesystems fsck"
Regenerate the initramfs:

mkinitcpio -p linux
Install a bootloader:
mount /dev/sda1 /boot/efi

bootctl --path=/boot/ install

/boot/loader/entries/arch.conf

title Arch Linux
linux /vmlinuz-linux-hardened
initrd /initramfs-linux-hardened.img
options cryptdevice=UUID={UUID}:volume root=/dev/mapper/volume-root quiet rw
cryptroot=UUID=<UUID> cryptdm=volume

SECURITY OPTIONS TO PASS TO THE KERNEL CMD = mitigations=auto mds=full,nosmt page_poison=1 vsyscall=none mce=0 slab_nomerge block.events_dfl_poll_msecs=1000 ipv6.disable=0 audit=1 apparmor=1 security=apparmor
 l1tf=full,force vbox
 mds=full,nosmt mitigations=auto,nosmt nosmt=force

 
 
 Other For ME ......
 options root=ZFS=zroot/ROOT/Arch rw  spl.spl_hostid=0x5e2a4d6f rootfstype=zfs rootflags=rw,noatime,xattr,posixacl resume=UUID=77a30146-9b98-4ca4-8920-427b84696fe0 nowatchdog nmi_watchdog=0 psmouse.synaptics_intertouch=0 scsi_mod.use_blk_mq=0 libahci.ignore_sss=1 loglevel=2 i915.fastboot=1 mitigations=auto mds=full,nosmt page_poison=1 vsyscall=none mce=0 slab_nomerge block.events_dfl_poll_msecs=1000 ipv6.disable=0 audit=1 net.ifnames=0 zswap.enabled=1 zswap.compressor=lz4 zswap.max_pool_percent=20 zswap.zpool=z3fold zram.num_devices=4 zram_num.devices=4 apparmor=1 security=apparmor

Setup the LUKS partition and activate the LVs:
cryptsetup luksOpen /dev/sda2
vgchange -ay

arch-chroot 
or

#########




ADD BLACKARCH
Run https://blackarch.org/strap.sh as root and follow the instructions.

curl -O https://blackarch.org/strap.sh
The SHA1 sum should match: 9f770789df3b7803105e5fbc19212889674cd503 strap.sh

sha1sum strap.sh

chmod +x strap.sh

sudo ./strap.sh

Install torctl

sudo pacman -S torctl
To find out your current IP, do:

torctl ip
To start Tor as a transparent proxy:

sudo torctl start
To check the status of services:

torctl status
If you want to change the IP on the Tor network:

sudo torctl chngid
To work with the Internet directly, without Tor, run:

sudo torctl stop
To change the MAC address on all network interfaces, run the command:

sudo torctl chngmac
To recover the original MAC addresses:

sudo torctl rvmac
The following command will add the torctl service to startup, that is, immediately after turning on the computer, all traffic will be sent through Tor:

sudo systemctl enable torctl-autostart.service
To remove a service from startup, do:

sudo systemctl disable torctl-autostart.service
You can also enable automatic memory cleaning every time you turn off the computer:

sudo systemctl enable torctl-autowipe.service
To disable this function:

sudo systemctl disable torctl-autowipe.service
This script knows about the existence of IPv6 traffic and successfully blocks it. DNS queries are redirected through Tor.


 
