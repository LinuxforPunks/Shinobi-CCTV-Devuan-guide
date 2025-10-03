# Shinobi-CCTV-Devuan-guide
walkthrough and bits of code for getting Shinobi to run on Devuan and possibly other non-systemd distros, not to mention Debian

**Introduction**

Shinobi works very nicely on Devuan Daedalus. For my own benefit and perhaps yours I will put the things I wished I knew here.

**1. Shinobi is not a program.**
   
It is a pair of scripts: **camera.js** and **cron.js**
These two scripts are managed by **node.js** (the pm2 command)
It uses a mysql server such as **mariadb** to store the camera settings and the users and passwords
And it has a webgui, which is never the source of its problems.

**2. You need the gitlab version.** (shinobi-master.tar.gz)
   
The github version still barely works but is terrible. And imo it's too easy to mistake them.
iirc it was something along the lines that there used to be a free Community Edition and a paid-for Pro Edition.
And the Community Edition became deprecated... but the Pro Edition became free.
Don't install the Community Edition version thinking it's the free one.


**3. Obtain RECENT versions for these programs** (DON'T rely on apt!)
   
- node.js (unpack the tarball into /usr/local/lib/nodejs )
- npm (probably included in the same tarball)
- use npm to install pm2 (npm install pm2 -g)
- mariadb (unpack the tarball into /usr/local/ )

- go (golang programming language - not needed directly by Shinobi but I'd recommend it)
- docker engine (also not needed directly)
- docker compose (also not needed directly)

remember to update the PATH variables so that the binaries are in directories your user(s) can run programs from!
  
**4. The official installation guide doesn't really need to recommend Ubuntu 22.04**
   
There are not really any dependencies at all of Ubuntu, or systemd. Being Javascript it's pretty much platform-agnostic.
The ninja way installation script isn't needed, you can git clone the whole project from gitlab and put it in /home/Shinobi
The permissions are reasonably forgiving: it wants to be user: Shinobi, but there's no reason it can't be the normal User. And it can potentially be root.
Each option will throw up different permissions issues and it's just a case of working through them.

**5. Ubuntu is (at time of writing) a dreadful distribution for this project to be based in**

because it is so insistent on taking over control of the mountpoint logic, with udisks2 defaulting to overriding /etc/fstab.
imo this is actively lethal to Shinobi because its own defaults will write video footage to the OS disk
and if the OS disk runs out of space the program will stop working. And it will stop working without useful 
error messages.

**6. Configure the disks first of all so that they are always at a consistent mount point.**

A disk filling up will stop both the recording and the stream of the cameras that use it, so I'd suggest to spread the 
recording between multiple smaller disks. Surveillance disks like WD Purple or Seagate Skyhawk are a must: using normal 
HDDs will make it seem as if the connections are intermittent when it's the disk not keeping up with the cameras.

**7. Install Docker alongside Shinobi** 

Storage would be key to any NVR program but perhaps it affects Shinobi more because of its being in Javascript
and being a little abstracted from the hardware. I recommend to /usr/local/ to provide the following containers
(or alternatives):-

- Scrutiny (for checking disk health)
- Grafana+Loki+Promtail (for syslogging)
- Grafana Alloy (if webhook logging is wanted, but see next point below)
- LibreNMS (for ping/latency monitoring)
- Uptime-Kuma (to show which cameras are up)
- Gotify (for alerting to outages)
- a dashboard

**8. Shinobi's approach to logging is strange.**

The output of the (constant) ffmpeg commands (of camera.js) is directed into http under each camera's configuration page.
And it won't come out of there without patching camera.js! The camera _Events_ (which depend on various outside helper programs for motion detection or face detection etc)
are logged by a webhook that uses HTTP POST but can at least be directed into a Grafana stack.

**9. Quick note about the auto-installer scripts**

The Ninja Way installation script, /INSTALL/start.sh, the UPDATE.sh and the UPDATE-v2-to-v3.sh... are confusing imo because they mostly install the same filenames to the same places.
But if you clone the repo into /home/Shinobi, and run the /INSTALL/start.sh one, that seemed to work and to let you ignore the others.

**10. The MariaDB mysql setup is confusing but does not require many steps to sort out**

When you install mysql you make a "root" user: this is confusing because it's the root user of mysql not necessarily of the PC.
You also make a dedicated user for Shinobi, which by default is MajesticFlame with a blank password. This new user also isn't a user of the PC just of mysql.
This new user of mysql must have the same name and password as in conf.json
There are 3 commands to set this up which I'll attach in the cheat sheet. Although this is automated in the official install scripts I found it kept needing to be
redone manually.

**11. Init.d Scripts**

On Devuan, with SysVinit, you want to autostart mariadb and pm2 (for it to run camera.js and cron.js) with init.d scripts. 
If dockerd is optionally added, it seems to have some problems with being at the same runlevel as those other two, but it works fine to call it from rc.local.
The only actual problem I noticed was that the "pm2 startup" command in the official install script doesn't recognize SysVinit and detects it as another program (iirc Openrun).

**12. You want a graphical desktop.**

It's possible to run the server headless but for getting connections to cameras and checking the vcoding of streams it's best to see what it looks like to the server.
I like LXDE for this.

**13. It's also good to set up an NFS service to share the CCTV disks.**

**I will put my copies of the scripts and a cheatsheet of useful commands here. Obviously these only worked for me on a certain PC at a certain moment in time but the key takeaway is that
you probably only need to make one new init script to launch Shinobi, and configure/enable a couple of others.**

