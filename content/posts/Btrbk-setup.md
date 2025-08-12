+++
date = '2025-08-11T13:48:43+02:00'
draft = false
title = 'Btrbk Setup'
+++

# Setting up BTRBK
After installing Arch Linux, I wanted to set up a tool to automate the creation of Btrfs snapshots on an hourly basis.

I had used Timeshift previously and knew it had support for Btrfs, so I initially tried to configure it. However, because my subvolume layout doesn't match what Timeshift expects, this approach was fruitless. I decided to use `btrbk` instead. While `btrbk` doesn't have a graphical UI like Timeshift, I determined that a graphical UI is not necessary since I don't normally need to browse backups graphically, and the snapshots are easily accessible in the `/.snapshots` directory whenever I need them.

Setting up `btrbk` is straightforward, just follow these steps.
1. Install btrbk from the official repository
   `sudo pacman -S btrbk`
2. Configure btrbk, by editing the primary configuration file:
   `sudo vi /etc/btrbk/btrbk.conf`
   ```yaml
   # /etc/btrbk/btrbk.conf
   timestamp_format          long
   snapshot_dir              /.snapshots
   
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
3. Create a script to run during each backup
   `sudo vi /usr/local/bin/btrbk-run.sh`
   ```bash
   #!/bin/bash
   
   /usr/bin/btrbk run # Runs a complete, new backup
   /usr/bin/btrbk clean # Cleans old backups locally and remotelly
   /usr/bin/btrbk resume # Resumes transfer if, at any point a backup has been taken without being transfered to a target.
   ```
4. Create a systemd service file
   `sudo vi /etc/systemd/system/btrbk-hourly.service`
   ```ini
   [Unit]
   Description=btrbk hourly snapshot and backup service
   
   [Service]
   Type=oneshot
   ExecStart=/usr/local/bin/btrbk-run.sh
   ```
5. Create a systemd timer file
   `sudo vi /etc/systemd/system/btrbk-hourly.timer`
   ```ini
   [Unit]
   Description=Run btrbk hourly snapshot and backup script
   
   [Timer]
   OnCalendar=hourly
   Persistent=true
   
   [Install]
   WantedBy=timers.target
   ```
6. Enable and start the systemd timer
   ```bash
   sudo systemctl enable --now btrbk-hourly.timer
   ```

After these steps, a snapshot will be taken every hour, on the hour.
