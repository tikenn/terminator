# Terminator
Backup and system maintenance for Linux systems and MySQL style databases.    It logs everything and only emails when something goes wrong.

## Quicklinks
- [Quick Start](#quick-start)
- [Getting Started](#getting-started)
- [How it works](#how-it-works)

# Quick Start
1. Put *terminator* on server (/opt/)
2. Run ```setup```
3. Fill out ```terminator.conf```
4. Add public keys external backup hosts
5. Sit back and relax

# Getting Started
1. Clone or download a release of this repository onto the server into a suitable location (/opt/ is recommended)
2. Run ```setup``` (```./setup```)
    1. Creates ```terminator.conf``` for system-specific configurations
    2. Creates keypairs for external backup hosts automagically upon request (fills out appropriate section of ```terminator.conf``` upon key creation)
    3. Requests time to run daily maintenance and sets up cron job in ```/etc/cron.d/```
3. Add system-specific configurations to ```terminator.conf``` file created during setup
4. If key(s) were requested, copy the public key (```*.pub```) to the user on the external host indicated in ```terminator.conf```

# How it works
