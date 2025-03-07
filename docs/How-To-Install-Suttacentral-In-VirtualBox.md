# Installing Suttacentral in VirtualBox

Recently I've been setting up a virtual machine on my desktop to run a
full instance of SuttaCentral.net locally. For my own benefit I've 
documented the steps I followed as I had to start from scratch several times.

You, lucky reader, may need to do this fewer times if you follow this procedure.

## What is VirtualBox?

VirtualBox lets you run a guest operating system on the host. For my own purposes
I want to develop on my Linux Mint desktop while having the server running separately
on Debian. You can get VirtualBox at http://www.virtualbox.org

It should be possible to run the server with supported Windows and Mac hosts, but I've
not tried that myself. 

## Configuring the VM

First download the net install image for Debian "Bookworm":

https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.9.0-amd64-netinst.iso

Start up VirtualBox and add a new VM. My settings:

```
Name:           suttacentral-bookworm
Folder:         /home/jr/virtualbox-vm
ISO Image:      The downloaded iso file above
Memory:         16GB (half my total)
CPU Processors: 6 (half my total)
Storage:        50GB (Total used after install is about 14GB)
Network:        NAT (I had trouble in bridge mode, may work for you)
```

## Install Debian

After starting the VM, you can start the OS installation. 

1. Select "Install" for a text mode installation.
2. Set language: English (or your own)
3. Country: Australia (or your own)
4. Keyboard: American English (or your own)
5. Hostname: `suttacentral-bookworm`
6. Domain: `local`
7. Set a root password
8. Add user: `jr` (or your own, you'll need to substitute this in a lot of places)
9. Set user password
10. Configure Clock: Western Australia (or your own)
11. Partitioning: Guided - Use Entire Disk
12. Choose the disk to partition, there should only be one.
13. Select "all files in one partition"
14. Finish partitioning and write to disk.
15. Wait for base system install
16. Configure package manager - "no" to scanning extra installation media
17. Select mirror country - Australia (or whatever is closest)
18. Select archive mirror - deb.debian.org (or as you prefer)
19. No to proxy
20. Wait for apt to scan the mirror
21. Popularity contest: up to you
22. Software selection: deselect desktop environment and gnome, select SSH and standard system utilities.
23. Yes to install grub on primary drive
24. Select `/dev/sda`
25. Installation is complete! Reboot

## Set up sudo

This isn't configured out of the box as on Ubuntu. As root:

```bash
apt install sudo
usermod -aG sudo jr
```

Logout and login again and you can use `sudo`

## Port forwarding

I had problems with using bridge mode, so the following instructions use NAT
and port forwarding. 

1. In the VM run `ip addr show` to see the guest IP address. Mine is `10.0.2.15`.
2. Shudown the VM
3. In the VirtualBox GUI click the settings button and make sure you're in expert mode.
4. Add these rules, substituting the VM IP if it differs from the one above:

### ArangoDB
```
  Protocol    TCP
  Host IP     127.0.0.1
  Host Port   2529
  Guest IP    10.0.2.15
  Guest Port  8529
```

### Flask
```
  Protocol    TCP
  Host IP     127.0.0.1
  Host Port   2501
  Guest IP    10.0.2.15
  Guest Port  5001
```

### HTTP
```
  Protocol    TCP
  Host IP     127.0.0.1
  Host Port   2580
  Guest IP    10.0.2.15
  Guest Port  80
```

### SSH
```
  Protocol    TCP
  Host IP     127.0.0.1
  Host Port   2529
  Guest IP    10.0.2.15
  Guest Port  22
```

Save the changes, start the VM and you should now be able to SSH in from the host machine:

`ssh -p 2522 jr@127.0.0.1`

## Start up headless

From now on we can configure the VM without the VirtualBox GUI.

```commandline
VBoxManage startvm "suttacentral-bookworm" --type headless
```

Wait a bit for it to start up, then login via ssh from the host.

## Install software required for SuttaCentral

To get the source and build it you'll need a couple of packages:

```commandline
sudo apt-get install git make
```

I like to use the Github CLI. Installation instructions here:

https://github.com/cli/cli/blob/trunk/docs/install_linux.md

And authenticate:

https://cli.github.com/manual/gh_auth_login

Now you can get the source:

```commandline
gh repo clone suttacentral/suttacentral
```

### Docker
Configure the apt repository as explained here:

https://docs.docker.com/engine/install/debian/

Then install the recommended packages:

```commandline
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Add your user to the docker group:

```commandline
sudo groupadd docker
sudo usermod -aG docker jr
newgrp docker
```

### Node.js

Start by installing Node Version Manager:

https://github.com/nvm-sh/nvm

Logout/login then run:

```commandline
nvm install 18
node -v
npm -v
```

And you should see node and npm are installed with the appropriate versions.

## Install Suttacentral

The readme covers it well:

https://github.com/suttacentral/suttacentral/

Once you've done that, you should be able to hit suttacentral in your browser at 

http://127.0.0.1:2580/

Note: Swagger is a bit strange. If you go to the `/api/docs` endpoint it will give an error. 

Entering http://localhost:2580/api/spec in the explore field will allow you to interact with the
server API. You can fix it for good by changing the `sc-swagger` section in the docker compose file.