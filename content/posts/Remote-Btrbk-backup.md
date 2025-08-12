+++
date = '2025-08-12T09:12:38+02:00'
draft = false
title = 'Remote Btrbk Backup'
+++

After setting up `btrbk` to take snapshots every hour, I decided it was time to build a dedicated backup host for off-device storage.  
The setup will use:

- **Encrypted OS disk** for the server (see [Encrypted Arch Install](/posts/installingarch/)).
- **4 TB encrypted Btrfs volume** for storing backups from multiple machines.
- **Non-root `backupuser`** account for all automated transfers (avoiding root logins entirely).
- **Future USB archive rotation** for off-site redundancy.

---

# 1. Prepare the Backup Host

This guide assumes **Arch Linux is already installed** and **SSH enabled with password authentication**.  
All commands below run as `root` unless otherwise specified.

## 1.1 Encrypt and format the backup disk
```bash
cryptsetup luksFormat /dev/sda1

# Create a keyfile for automatic unlock
dd if=/dev/random bs=4k count=1 iflags=fullblock \
  | install -m 0660 /dev/stdin /etc/cryptsetup-keys.d/backups.key

cryptsetup luksAddKey /dev/sda1 /etc/cryptsetup-keys.d/backups.key
```
## 1.2 Add it to `/etc/crypttab`:
```bash
# UUID can be found via `blkid`
backupdisk UUID=<UUID-of-/dev/sda1> /etc/cryptsetup-keys.d/backups.key
```
## 1.3 Unlock and format as Btrfs:
```bash
cryptsetup open /dev/sda1 backupdisk
mkfs.btrfs /dev/mapper/backupdisk
```
## 1.4 Add to `/etc/fstab`:
```bash
UUID=<UUID-of-backupdisk> /mnt/backups btrfs rw,norelatime,compress=zstd:3,space_cache=v2 0 0
```
# 2. Create Backup Subvolumes
Create a separate subvolume for each client:
```bash
mount /mnt/backups
btrfs su cr /mnt/backups/backuphost
btrfs su cr /mnt/backups/laptop
btrfs su cr /mnt/backups/desktop
...
```

# 3. Create a dedicated Backup user
To avoid root needing to log in over SSH
## 3.1 Create backup user to receive backups on the backup host
   ```bash
   useradd -m -U backupuser
   chown backupuser:backupuser /mnt/backups/*
   su backupuser
   # as backupuser run the following commands on the backup host
   passwd # Create a secure password, we can use to copy ssh identities in later steps
   mkdir -p ~/.ssh
   chmod 0700 ~/.ssh
   touch ~/.ssh/authorized_keys
   chmod 0644 ~/.ssh/authorized_keys
   logout
   # Now we are back at the root shell.
   ```
## 3.2 Give backupuser access to execute required commands as root on the backup host
   ```bash
   visudo -f /etc/sudoers.d/btrbk_permissions
   ```
   ```ini
   backupuser ALL=(root) NOPASSWD: /usr/bin/btrbk *
   backupuser ALL=(root) NOPASSWD: /usr/sbin/btrfs *
   backupuser ALL=(root) NOPASSWD: /usr/bin/readlink *
   ```
## 3.3 Create backup user to send backups on each source machine
   ```bash
   useradd -m -U backupuser
   ```
## 3.4 Give backupuser access to execute btrbk commands as root on each source machine
   ```bash
   visudo -f /etc/sudoers.d/btrbk_permissions
   ```
   ```ini
   backupuser ALL=(ALL) NOPASSWD: /usr/bin/btrbk *
   ```
# 4. Set up SSH Key-based login
On each client: Create a config entry for the backup host for the `root` user
   ```bash
   ssh-keygen -t ed25519
   vi ~/.ssh/config
   ```
   ```yaml,config
   Host backupserver
     Hostname <backup-server-ip>
     User backupuser
     Port 22
   ```
   ```bash
   ssh-copy-id backupserver
   ```
# 5. Configure `btrbk` on each client machine
Example `/etc/btrbk/btrbk.conf`
```conf
# /etc/btrbk/btrbk.conf
timestamp_format          long
snapshot_dir              /.snapshots

target                    ssh://backupserver/mnt/backups/work-laptop/
target_preserve_min       2d
target_preserve           *y 24m 6w 14d
backend_remote            btrfs-progs-sudo

group                     hourly
  subvolume               /
    snapshot_preserve_min 1d
    snapshot_preserve     *y 18m 6w 14d
    snapshot_create       ondemand
  subvolume               /home
    snapshot_preserve_min 1d
    snapshot_preserve     *y 18m 6w 14d
    snapshot_create       onchange
  subvolume               /root
    snapshot_preserve_min 1d
    snapshot_preserve     *y 18m 6w 14d
    snapshot_create       onchange
  subvolume               /boot
    snapshot_preserve_min 1d
    snapshot_preserve     *y 18m 6w 14d
    snapshot_create       ondemand
```
# 6. Modify hourly backups service
## 6.1 Execute btrbk commands as sudo
   `vi /usr/local/bin/btrbk-run.sh`
   ```bash
   #!/bin/bash
   sudo /usr/bin/btrbk run
   sudo /usr/bin/btrbk clean
   sudo /usr/bin/btrbk resume
   ```
## 6.2 Update `btrbk-hourly.service` to run as `backupuser`
   `vi /etc/systemd/system/btrbk-hourly.service`
   ```ini
   [Unit]
   Description=Btrbk hourly snapshot and backup service
   
   [Service]
   Type=oneshot
   User=backupuser
   Group=backupuser
   ExecStart=/usr/local/bin/btrbk-run.sh
   ```
## 6.3 Update systemd
   `systemctl daemon-reload`
