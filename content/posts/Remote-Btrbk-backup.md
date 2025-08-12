+++
date = '2025-08-12T09:12:38+02:00'
draft = true
title = 'Remote Btrbk Backup'
+++
After setting up `btrbk` to take snapshots every hour, I decided it's time to set up a new backup host, and configure btrbk to transfer the backups.
The backup server is set up with [encrypted os disk](posts/installingarch/), and an additional 4TB disk that will be used to back up each of my laptops to.
Later I will add a script to archive the backups to USB disk that I will rotate, but I need to set up the backup server first.

In addition, I have set up a computer with a fairly large btrfs volume that I call work-computer in my .ssh config, this machine can be made reachable e.g. through a tailnet or ssh jump host, If you want to transfer backups to this computer simultaneously with backups, you should set up a separate user instead of running the service as root, e.g. backupuser
1. Create the backup user `sudo useradd -m -U backupuser`
2. Give the backupuser access to run btrbk commands as sudo
   `sudo visudo -f /etc/sudoers.d/btrbk_permissions`
   ```
   # Allows backupuser access to specific btrbk commands
   backupuser ALL=(ALL) NOPASSWD: /usr/bin/btrbk *
   ```
3. Create a ssh key for the backup user 
   ```bash
   sudo -su backupuser
   ssh-keygen
   ```
4. Create a ssh host entry for the backupserver in the backupuser's .ssh config
   `sudo -u backupuser vi /home/backupuser/.ssh/config`
   ```yaml
   Host backup-server
     Hostname 192.168.x.x
     User backupuser
     Port 22
   ```
5. Create the backup user on the backup server like in step [2. Give the backupuser access to run btrbk commands as sudo]
6. Give the backupuser owner permissions to the backup target path on the backupserver
   ```bash
   sudo chown -R backupuser:backupuser /mnt/backups
   ```
7. Add ssh key from the source computer's `/home/backupuser/.ssh/id_ed25519.pub` to `home/backupuser/.ssh/authorized_keys` on the backup server
8. Update the scripts and configurations to run as backup user:
   `# /usr/local/bin/btrbk-run.sh`
   ```bash
   #!/bin/bash
   
   /usr/bin/sudo /usr/bin/btrbk run
   /usr/bin/sudo /usr/bin/btrbk clean
   /usr/bin/sudo /usr/bin/btrbk resume
   ```

   `# /etc/systemd/system/btrbk-hourly.service`
   ```ini
   [Unit]
   Description=btrbk hourly snapshot and backup service
   
   [Service]
   Type=oneshot
   User=backupuser
   Group=backupuser
   ExecStart=/usr/local/bin/btrbk-run.sh
   ```

   `# /etc/btrbk/btrbk.conf`:
   ```yaml
   # /etc/btrbk/btrbk.conf
   timestamp_format          long
   snapshot_dir              /.snapshots
   
   target                    ssh://backup-server/mnt/backups/$(hostnamectl hostname | tr -d '\n')/
   target_preserve_min       2d
   target_preserve           *y 24m 6w 14d
   
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

