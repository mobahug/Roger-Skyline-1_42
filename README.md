### V.1 VM Part

First we need to download and install a ***hypervisor*** of our choice.
I choosed [VirtualBox](https://www.virtualbox.org/wiki/Downloads). (newest version)

For that we will need to install a Linux OS of our choice.
I choosed [Debian](https://www.debian.org/distrib/netinst). (newest version)

After the VB installation, create a new OS, called ***Debian*** it will find automaticly
for you the system and add everything to it by default.

During the installation add the downloaded disk to it.

After that we need to setup the Debian, which is itself simple (everything is default), however
pay some attention for the partition part.

From ***partition part*** choose ***manual*** and from there `SCSI3 (0,0,0) (sda) - 8.6 GB ATA VBOX HARDDISK`

Create new partition:

    primal size of `4.2 GB´
    on the root

and

    extend(logical) `<choosable withing what you have left from 8Gib>`
    on /home dir
    
then

`Finish partitioning and choose write changes to disk`

After this left everything as default for minimal black & white environment

### V.2 Network and Security Part

##You must create a non-root user to connect to the machine and work.

  Non-root user login was created when we seted up the OS. just log in.
  However if need to add another ***superuser***
    Go to root
    adduser <username>
    usermod -aG sudo <username>

##Use sudo, with this user, to be able to perform operation requiring special rights.
  
  Go to root:
  
  ```
  sudo apt update -y
  sudo apt upgrade -y
  apt install sudo vim -y
  
  
    

