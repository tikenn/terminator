# Terminator
Backup and system maintenance for Debian-based systems and MySQL style databases.    It logs everything and only emails when something goes wrong.

## Quicklinks
- [Features](#features)
- [Quick Start](#quick-start)
- [How it works](#how-it-works)
    - [Configuration File](#configuration-file)
    - [Programs Used](#programs-used)
    - [Backup Schedule](#backup-schedule)
    - [Remote Host Setup](#remote-host-setup)
    - [SSH Keys](#ssh-keys)
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

# How it works
## Configuration File
 - The configuration file `terminator.conf` is automatically set up when running `./setup`
 - Answers to `./setup` questions are automatically stored there
 - Other values are set to default in `terminator` script
 - Database and remote backup host parameters need to be set in order to take advantage of them

## Programs Used
- Updates: `aptitude`
- Database backups:  `mysql`
- System backups:  `tar`
- Remote backups:  `rsync`

## Backup Schedule
 - Setup file (`./setup`) requests schedule time for daily run and automatically sets up cron job

### Updates
 - Updates and upgrades are run daily

### Database
 - Automagic daily, weekly, monthly, and yearly database backup scheduling based on install date
 - Daily, weekly, monthly, and yearly rollover settings in `terminator.conf` determine how many database backups per each time frame to keep; older backups are deleted
 - Missed weekly, monthly, and yearly backups will occur at the next scheduled daily backup

### System
 - System backups are incremental
 - Settings in `terminator.conf` allow for a *weekly*, *monthly*, or *yearly* level 0 backup (full system backup)
 - Incremental backups are made on a rotating weekly basis after a level 0 backup until the next level 0 (levels: 1, 2, 3, 4, 5, 6, 1, 2, 3, ...)
 - Missed level 0 backups will be attempted on the next scheduled backup
 - Other levels that are missed will cause data loss and are simply skipped ([restores](#system-restore) still take place in the same order)

### Remote
- Settings in `terminator.conf` allow for a *weekly*, *monthly*, or *yearly* backups to remote
- Missed backups will be attempted the next day at the scheduled backup time for the local machine

## Remote Host Setup
- `terminator.conf` allows for two remote hosts based on the principle of having an on-site and off-site backup
- In order to login to a remote host, `terminator` will require the following information per host in `terminator.conf`
    - IP Address or Domain name (on_site_host_domain= and off_site_host_domain=)
    - Port Number (on_site_host_port= and off_site_host_port=)
    - Host user (on_site_host_user= and off_site_host_user=)
    - SSH Key (on_site_host_ssh_key= and off_site_host_ssh_key=) - automatically fille out if requested in `./setup`
    - Directory for storing backups on remote host (on_site_host_backup_dir= and off_site_host_backup_dir=)
- After filling out the appropriate sections of `terminator.conf`, `terminator` will use ***rsync*** to manage remote backups 

## SSH Keys
- The setup file (`./setup`) will create SSH Keys for remote hosts and set up `terminator.conf` with their location upon request (however, these can be set up manually configured if desired)
- `terminator` requires **unencrypted** SSH Keys to login to remote hosts
- Once created, `./setup` will automatically store the private key(s)' location
- Place the public key `*.pub` on the remote host in $HOME/.ssh/ and rename it to `authorized_hosts` ($HOME refers to the home directory of the backup user on the remote host; see [Remote Host Setup](#remote-host-setup))

## System restore
1. Reinstall the same operating system on the same or new machine
2. Copy ALL files in $BACKUP_DIR/systemdump to the new machine ($BACKUP_DIR: backup directory set in `terminator.conf`)
3. For each of the backup levels, run the following command, starting with the lowest level and ending with the highest:
    
    `sudo tar xzpvf backup.tar.gz -C / --numberic-owner`

4. Run the following command if the directories in the command don't exist:

    `mkdir /proc /sys /mnt /media`

5. If database restore is necessary, use the most recent file in $BACKUP_DIR/mysqldump with the following command:

    `mysql -u root -p --all-databases < backup.sql`

## Dependencies
Terminator primarily uses standard Linux tools; however, email notifications requires installation of email software.  These are installed with `./setup`.  Also, remote backups use *rsync*, but this is assumed to be pre-installed on the system.
 - [mailutils](http://mailutils.org/)
 - [mutt](http://www.mutt.org/)
 - [rsync](https://rsync.samba.org/)
