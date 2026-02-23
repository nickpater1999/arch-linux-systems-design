# Base Installation

This document describes the base system installation for the architecture defined in this repository.

The installation targets:
- Legacy BIOS systems
- No secure boot available
- GUID Partition Table (GPT)
- Full root encryption using LUKS2
- Btrfs with structured subvolumes
- No LVM
- No keyfile-based unlock
- Passphrase-based disk unlock at boot
- GRUB as bootloader

This section assumes familiarity with the standard Arch installation flow.


## 1. Installation Assumptions

- System booted in legacy BIOS mode
- Disk device identified (e.g., /dev/sdX)
- Network connectivity established
- System clock synchronized
- Arch mirror servers selected (e.g., Reflector used in the live environment)


## 2. Partition Layout

Root is fully encrypted.

`/boot` remains unencrypted to allow GRUB to load the kernel and initramfs.

**Target Layout**

| Device    | Size        | Role                          |
|-----------|-------------|-------------------------------|
| /dev/sdX1 | 512M        | Unencrypted /boot (ext4)      |
| /dev/sdX2 | *Remainder* | LUKS2 container -> Btrfs root |

No LVM layer is used.

Encryption is applied directly to the root partition.


## 3. Create Partitions

The GUID fdisk (gdisk) utility may be used to create a GUID Partition Table (GPT) and its partitions.

To enter the gdisk interactive environment:
```bash
gdisk /dev/sdX
```

Input `?` to list available actions.

Create the two partitions, using type codes for hygiene:
- 1st partition: size 512M, type code `8300`
- 2nd partition: remainder of disk, type code `8304`


## 4. Encrypt the Root Partition

Format and open the encrypted container:
```bash
cryptsetup luksFormat /dev/sdX2
cryptsetup open /dev/sdX2 cryptroot
```

This creates `/dev/mapper/cryptroot`. All subsequent file system operations target this mapped device.


## 5. Btrfs File System and Subvolume Layout

Format the file system in the encrypted container as Btrfs:
```bash
mkfs.btrfs /dev/mapper/cryptroot
```

Mount the file system and create subvolumes in a flat layout:
```bash
mount /dev/mapper/cryptroot /mnt

btrfs su cr /mnt/@
btrfs su cr /mnt/@home
btrfs su cr /mnt/@snapshots
btrfs su cr /mnt/@var_cache
btrfs su cr /mnt/@var_log
btrfs su cr /mnt/@var_spool
btrfs su cr /mnt/@var_tmp
```

Unmount the file system and mount the subvolumes using mount options:
```bash
umount /mnt

mount -o noatime,compress=zstd,subvol=@ /dev/mapper/cryptroot /mnt
mount --mkdir -o noatime,compress=zstd,subvol=@home /dev/mapper/cryptroot /mnt/home
mount --mkdir -o noatime,compress=zstd,subvol=@snapshots /dev/mapper/cryptroot /mnt/.snapshots
mount --mkdir -o noatime,compress=no,subvol=@var_cache /dev/mapper/cryptroot /mnt/var/cache
mount --mkdir -o noatime,compress=no,subvol=@var_log /dev/mapper/cryptroot /mnt/var/log
mount --mkdir -o noatime,compress=no,subvol=@var_spool /dev/mapper/cryptroot /mnt/var/spool
mount --mkdir -o noatime,compress=no,subvol=@var_tmp /dev/mapper/cryptroot /mnt/var/tmp
```

Format and mount `/boot`:
```bash
mkfs.ext4 /dev/sdX1
mount /dev/sdX1 /mnt/boot
```

This establishes a clean separation between:
- Encrypted system state
- Unencrypted bootloader


## 6. Base System Installation

Install essential packages:
```bash
pacstrap -K /mnt base linux linux-firmware grub
```

Consider also installing:
- CPU microcode (`intel-ucode` or `amd-ucode`)
- networking software
- a firewall
- `reflector`, a tool for Arch mirror server selection
- a privilege escalation utility
- `xdg-user-dirs` for user XDG home directory management
- a console text editor
- packages for accessing man and info pages
- `tlp` for laptop power management tools

Generate `fstab`:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Change root:
```bash
arch-chroot /mnt
```


## 7. Initramfs Configuration (Encryption Hook)

Edit `/etc/mkinitcpio.conf` and ensure the `encrypt` hook appears before `filesystems`.

Example:
```
HOOKS=(base udev autodetect keyboard keymap modconf block encrypt filesystems fsck)
```

Regenerate initramfs:
```bash
mkinitcpio -P
```

This enables early userspace unlocking of the LUKS container during boot.


## 8. GRUB Configuration for Encrypted Root

Retreive the UUID of the encrypted partition:
```bash
blkid /dev/sdX2
```

Edit `/etc/default/grub` and add the cryptdevice paramater:
```
GRUB_CMDLINE_LINUX="cryptdevice=UUID=<UUID-of-sdX2>:cryptroot root=/dev/mapper/cryptroot"
```

Install GRUB (BIOS target):
```bash
grub-install --target=i386-pc /dev/sdX
```

Generate the configuration:
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

## 9. Pre-Reboot Configuration

In the live chrooted environment, proceed with the standard base configuration:
- Timezone
- Locale
- Hostname
- Root password

Exit chroot, safely unmount, and reboot into the system.


## 10. Post-Boot Configuration

Proceed with base configuration:
- Networking
- Firewall
- Time and NTP servers
- Arch mirror server selection
- User creation
- User privilege escalation

Enable linger for a user to allow the use of their `systemd user` services, if a display manager will not be used to start user sessions:
```bash
loginctl enable-linger UserName
```

`xdg-user-dirs` may be installed for XDG home directory management.

Log in as the non-root user you wish to configure XDG home directories for.

Create and configure the default selection of XDG home directories:
```bash
xdg-user-dirs-update
```

Edit `~/.config/user-dirs.dirs` to personalize the XDG home directory selections. Run `xdg-user-dirs-update` to flush any changes.

`tlp` may be installed for laptop power management.

Enable its service:
```bash
systemctl enable --now tlp
```

Optionally add laptop-lid functionality by editing values in `/etc/systemd/logind.conf`:
```ini
HandleLidSwitch=suspend
HandleLidSwitchExternalPower=ignore
```


## 11. Boot Flow Summary

On boot:
1. BIOS loads GRUB from disk
2. GRUB loads kernel and initramfs from `/boot`
3. The initramfs `encrypt` hook prompts for the LUKS passphrase
4. The encrypted container is unlocked
5. Btrfs root subvolume mounts
6. System continues normal initialization

This design maintains:
- Full root encryption
- Minimal layering
- Deterministic boot behavior
- Clear separation of trust boundaries


## 12. Security Model Notes

- `/boot` remains unencrypted due to BIOS constraints
- Disk encryption protects against offline disk access and device theft
- Physical bootloader tampering is outside the scope of this architecture

Passphrase-only unlocking is intentionally chosen over embedded keyfiles to preserve physical security.


## 13. Next Steps

At this stage, the system provides:
- Encrypted root file system
- Structured Btrfs layout
- Deterministic boot process

The next layer of the architecture is the **System Recovery Model**, which formalizes snapshot management, rollback strategies, and external backup replication.
