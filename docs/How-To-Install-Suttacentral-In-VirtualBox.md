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

