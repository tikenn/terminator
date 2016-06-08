# Terminator
Backup and system maintenance for Debian-based systems and MySQL style databases.    It logs everything and only emails when something goes wrong.

## Quicklinks
- [Features](#features)
- [Quick Start](#quick-start)
- [How it works](#how-it-works)
    - [Configuration File](#configuration-file)
    - [Backup Schedule](#backup-schedule)
    - [SSH Keys](#ssh-keys)
    - [Remote Host Setup](#remote-host-setup)
    - [Dependencies](#dependencies)

# Features
- Automatic daily updates
- Daily, weekly, monthly, and yearly MySQL database backups
- Easy install with ```./setup``` file
- Automatically setup cron job for updates and backups

# Quick Start
1. Put *terminator* on server (/opt/)
2. Run ```setup```
3. Fill out ```terminator.conf```
4. Add public keys to external backup hosts
5. Sit back and relax

# How it works
## Configuration File
 - The configuration file ```terminator.conf``` is automatically set up when running ```./setup```
 - Answers to ```./setup``` questions are automatically stored there
 - Other values are set to default in ```terminator``` script
 - Database and remote backup host parameters need to be set in order to take advantage of them

## Backup Schedule
 - Setup file (```./setup```) requests schedule time for daily run and automatically sets up cron job
 - Updates and upgrades are run daily
 - Automagic daily, weekly, monthly, and yearly database backup scheduling based on install date
 - Daily, weekly, monthly, and yearly rollover settings in ```terminator.conf``` determine how many backups per each time frame to keep; older backups are deleteed

## SSH Keys
 - The setup file (```./setup```) will create SSH Keys for remote hosts and set up ```terminator.conf``` with their location upon request (however, these can be set up manually configured if desired)
 - ```terminator``` requires **unencrypted** SSH Keys to login to remote hosts
 - Once created, ```./setup``` will automatically store the private key(s)' location
 - Place the public key (```*.pub```) on the remote host in ```$HOME/.ssh/``` and do one of following:
    - ```authorized_keys``` does not exist:  rename public key to ```authorized_keys``` (```mv *.pub authorized_keys```)
    - ```authorized_keys``` exists:  append public key to ```authorized_keys``` (```cat *.pub >> authorized_keys```)

## Remote Host Setup
 - ```terminator.conf``` allows for two remote hosts based on the principle of having an on-site and off-site backup
 - In order to login to a remote host, ```terminator``` will require the following information per host in ```terminator.conf```
     - IP Address or Domain name (```on_site_host_domain=``` and ```off_site_host_domain=```)
     - Port Number (```on_site_host_port=``` and ```off_site_host_port=```)
     - Host user (```on_site_host_user=``` and ```off_site_host_user=```)
     - SSH Key (```on_site_host_ssh_key=``` and ```off_site_host_ssh_key=```) - automatically filled out if requested in ```./setup```
     - Directory for storing backups on remote host (```on_site_host_backup_dir=``` and ```off_site_host_backup_dir=```)
 - After filling out the appropriate sections of ```terminator.conf```, ```terminator``` will use ***rsync*** to manage remote backups 

## Dependencies
Terminator primarily uses standard Linux tools; however, email notifications requires installation of email software.  These are installed with ```./setup```.  Also, remote backups use *rsync*, but this is assumed to be pre-installed on the system.
 - [mailutils](http://mailutils.org/)
 - [mutt](http://www.mutt.org/)
 - [rsync](https://rsync.samba.org/)
