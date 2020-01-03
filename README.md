### ArchLinux|Hardened Encrypted Luks Lvm Ext4 Systemd-Boot AppArmor All The Traffic Trhough TOR. :) WikiLeaks Like Linux. ###

Needs / Deps 4 This Task .

https://sourceforge.net/projects/revenge-installer/



options root=ZFS=zroot/ROOT/Arch rw  spl.spl_hostid=0x5e2a4d6f rootfstype=zfs rootflags=rw,noatime,xattr,posixacl resume=UUID=77a30146-9b98-4ca4-8920-427b84696fe0 nowatchdog nmi_watchdog=0 psmouse.synaptics_intertouch=0 scsi_mod.use_blk_mq=0 libahci.ignore_sss=1 loglevel=2 i915.fastboot=1 mitigations=auto mds=full,nosmt page_poison=1 vsyscall=none mce=0 slab_nomerge block.events_dfl_poll_msecs=1000 ipv6.disable=0 audit=1 net.ifnames=0 zswap.enabled=1 zswap.compressor=lz4 zswap.max_pool_percent=20 zswap.zpool=z3fold zram.num_devices=4 zram_num.devices=4 apparmor=1 security=apparmor

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










 
