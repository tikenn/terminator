# Terminator
Backup and system maintenance for Debian-based systems and MySQL style databases.    It logs everything and only emails when something goes wrong.

## Quicklinks
- [Features](#features)
- [Quick Start](#quick-start)
- [Setup](#setup)
    - [Installation](#installation)
    - [Emails](#emails-optional)
    - [Database Setup](#database-setup-optional)
    - [Remote Host Setup](#remote-host-setup)
    - [SSH Keys](#ssh-keys)
- [How it works](#how-it-works)
    - [Configuration File](#configuration-file)
    - [Programs Used](#programs-used)
    - [Backup Schedule](#backup-schedule)
    - [Backup Process](#backup-process)
- [Restore](#restore)
    - [Database Restore](#database-restore)
    - [System Restore](#system-restore)
- [Dependencies](#dependencies)

# Features
- Automatic daily updates
- Daily, weekly, monthly, and yearly MySQL database backups
- Daily incremental backups with tar
- Easy install with `./setup` file
- Automatically sets up cron job for updates and backups

# Quick Start
1. Put *terminator* on server (/opt/)
2. Run `setup`
3. Fill out `terminator.conf`
4. Add public keys to external backup hosts
5. Sit back and relax

# Setup
## Installation
Installation is made super simple through the use of git/GitHub.  Simply follow these steps:

1. Acquire *terminator* from GitHub using one of the following methods:
    - git clone https://github.com/tikenn/terminator.git
    - [Latest Release](https://github.com/tikenn/terminator/releases)
2. Place in a director of your choice on the computer (`/opt` is recommended, but *terminator* will detect any install location through `setup`)
3. Run `./setup`
4. Fill out appropriate components of `terminator.conf`.  Details are below and anything not filled out will either not run ([databases](#database-setup) and [remote hosts](#remote-host-setup)) or simply revert to default

## Emails (optional)
The system will email regarding fatal errors and attach the log file for review if desired.

- In `terminator.conf`, add a comma-separated list (no spaces) of emails as the value of `mailto=`
- Run `./send_test_email` (most email servers will put this in spam, so simply mark the email as 'not spam')

## Database Setup (optional)
If a MySQL-compatible database is present on the system, this will allow it to be backed up appropriately without system backups missing database files that are open for writing during the procedure

- In `terminator.conf`, fill out the following variables
    - `local_host_backup_dir`: directory where database backups will be stored.  Defaults to $INSTALL_LOCATION/terminator/backups
    - `local_host_db_user`:  root user of MySQL-compatible database
    - `local_host_db_pass`:  root password of MySQL-compatible database

## Remote Host Setup
- `terminator.conf` allows for two remote hosts based on the principle of having an on-site and off-site backup
- In order to login to a remote host, `terminator` will require the following information per host in `terminator.conf`
    - IP Address or Domain name (`on_site_host_domain=` and `off_site_host_domain=`)
    - Port Number (`on_site_host_port=` and `off_site_host_port=`)
    - Host user (`on_site_host_user=` and `off_site_host_user=`)
    - SSH Key (`on_site_host_ssh_key=` and `off_site_host_ssh_key=`) - automatically fille out if requested in `./setup`
    - Directory for storing backups on remote host (`on_site_host_backup_dir=` and `off_site_host_backup_dir=`)
        - ***Important: backup directories must already be on the remote system with the proper permissions!***
- After filling out the appropriate sections of `terminator.conf`, `terminator` will use ***rsync*** to manage remote backups
- Finally, place public SSH keys on remote servers (see [next section](#ssh-keys))

## SSH Keys
- The setup file (`./setup`) will create SSH Keys for remote hosts and set up `terminator.conf` with their location upon request (however, these can be set up manually configured if desired)
- `terminator` requires **unencrypted** SSH Keys to login to remote hosts
- Once created, `./setup` will automatically store the private key(s)' location
- Place the public key `*.pub` on the remote host in $HOME/.ssh/ and rename it to `authorized_hosts` ($HOME refers to the home directory of the backup user on the remote host; see [Remote Host Setup](#remote-host-setup))

# How it works
## Configuration File
 - The configuration file `terminator.conf` is automatically set up when running `./setup`
 - Answers to `./setup` questions are automatically stored there
 - Other values are set to default in `terminator` script
 - Database and remote backup host parameters need to be set in order to take advantage of them

## Programs Used
- Updates: `aptitude`
- Database backups:  `mysqldump`
- System backups:  `tar` and `split`
- Remote backups:  `rsync`

## Backup Schedule
 - Setup file (`./setup`) requests schedule time for daily run and automatically sets up cron job

### Update Schedule
 - Updates and upgrades are run daily

### Database Backup Schedule
 - Automagic daily, weekly, monthly, and yearly database backup scheduling based on install date
 - Daily, weekly, monthly, and yearly rollover settings in `terminator.conf` determine how many database backups per each time frame to keep; older backups are deleted
 - Missed weekly, monthly, and yearly backups will occur at the next scheduled daily backup

### System Backup Schedule
 - System backups are incremental
 - Settings in `terminator.conf` allow for a *weekly*, *monthly*, or *yearly* level 0 backup (full system backup)
 - Incremental backups are made on a rotating weekly basis after a level 0 backup until the next level 0 (levels: 1, 2, 3, 4, 5, 6, 1, 2, 3, ...)
 - Missed level 0 backups will be attempted on the next scheduled backup
 - Other levels that are missed will not cause data loss and are simply skipped ([restores](#system-restore) still take place in the same order)

### Remote
- Settings in `terminator.conf` allow for a *weekly*, *monthly*, or *yearly* backups to remote
- Missed backups will be attempted the next day at the scheduled backup time for the local machine

## Backup Process
### Database Backup
- `mysqldump` backups up all databases and database settings (including users and passwords)
- `mysqldump` is run daily
- Snapshots are made at weekly, montly, and yearly time intervals (see [Backup Schedule: Database](#database-backup-schedule)) by copying the daily backup

### System Backup
- `tar` is used to backup the entire system
- `split` is used to break up the backup file into approximately 1 GB files for easier transfer to remote backups
- Finally, backups are compressed with `gzip`

## Remote Backup
- `rsync` transfers backups to remote servers if setup
- Files are compressed during transfer to decrease network strain

# Restore
## Database Restore
Please note that this is a MySQL-specific protocol

1. Please choose the last snapshot level to restore (previous daily, weekly, monthly, or yearly snapshot)
2. Go to the chosen directory using the following command:

    `cd $INSTALL_LOCATION/backups/<<snapshot_directory>>`

3. Type in the following command:
    
    `mysql -u <<root_user>> -p < database_file_to_restore.sql`

## System Restore
1. Reinstall the same operating system on the same or new machine
2. Copy ALL files in $BACKUP_DIR/systemdump to the new machine ($BACKUP_DIR: backup directory set in `terminator.conf`)
3. For each of the backup levels, run the following command, starting with the lowest level and ending with the highest:
    
    `sudo cat backup.tar.gz* | tar xzpvf - -C / --numeric-owner`

4. Run the following command if the directories `/proc`, `/sys`, `/mnt`, or `/media` don't exist after running the command in *3*:

    `mkdir /proc /sys /mnt /media`

5. If database restore is necessary, use the most recent file in $BACKUP_DIR/mysqldump with the following command:

    `mysql -u root -p --all-databases < backup.sql`

# Dependencies
Terminator primarily uses standard Linux tools; however, email notifications requires installation of email software.  These are installed with `./setup`.  Also, remote backups use *rsync*, but this is assumed to be pre-installed on the system.
 - [mailutils](http://mailutils.org/)
 - [mutt](http://www.mutt.org/)
 - [rsync](https://rsync.samba.org/)
