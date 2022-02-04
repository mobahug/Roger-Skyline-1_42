# V.1 VM Part

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

    primal size of 4.2 GB

on the root

and

    extend(logical) <choosable withing what you have left from 8Gib>

on /home dir
    
then

    Finish partitioning and choose write changes to disk

After this left everything as default for minimal black & white environment

# V.2 Network and Security Part

## You must create a non-root user to connect to the machine and work.

Non-root user login was created when we seted up the OS. just log in.
However if need to add another ***superuser***

Go to root
    adduser <username>
    usermod -aG sudo <username>

## Use sudo, with this user, to be able to perform operation requiring special rights.
  
Go to root:
    $ sudo apt update -y
    $ sudo apt upgrade -y
    $ apt install sudo vim -y

log back to you user and give acces (writible) to ´/etc/sudoers file, because we need to still
initialize the user to be able to make all kind of changes.
    $ /etc
    $ sudo chmod +w sudoers
    $ sudo vim sudoers

add your user to ***# User privilage specification***

so:

```<username>  ALL=(ALL:ALL) ALL```

## We don't want you to use the DHCP service of your machine. You’ve got to configure it to have a static IP and a Netmask in \30.
    
We have to change first the network configuration so go to:
    
    1. Virtual Box settings
    2. Network
    3. Attached to
    4. ChooseBridged Adapter
    
By default we don't have ifconfig so we can [get](https://www.how2shout.com/linux/install-ifconfigon-debian-11-or-10-if-command-not-found/) it.

Check with ***ifconfig*** what is our name of our bridge adapter.

To make static IP simply [go](https://linuxconfig.org/how-to-setup-a-static-ip-address-on-debian-linux) to: 
    ```/etc/network/interfaces```

Also remember to give ***write*** permission to be able to modify the file.

Delete everything from under the ```# The primary network interface```

replace with ```auto enp0s3```
    
After this we have to create a file called ```enp0s3```in ```/etc/network/interfaces.d/```
here we write

        iface enp0s3 inet static
                adress 10.11.1.200 (choosend ip under same subnet, i choosed 200)
                netmask 255.255.255.252 (/30)
                gateway 10.11.254.254

Now we have to restart the service:

        $ sudo service networking restart (stay in the same dir)
    
Command ***ifconfig*** to make sure the changes has been made.

## You have to change the default port of the SSH service by the one of your choice.
SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT
be allowed directly, but with a user who can be root.
    
First we need to [go](https://www.cyberciti.biz/faq/howto-change-ssh-port-on-linux-or-unix-server/) to ```/etc/ssh/sshd_config```
    
    $ sudo vim /etc/ssh/sshd_config
    
Change line ```# Port 22```

[Check out](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) IANA.
   
    Comment it out and choose port between 49152 - 65535 
    
"Dynamic ports—Ports in the range 49152 to 65535 are not assigned, controlled, or registered.
They are used for temporary or private ports. They are also known as private or non-reserved ports.
Clients should choose ephemeral port numbers from this range, but many systems do not."
    
    I choosed Port 55556
    
Now we restarting our sshd service so:
    
    $ sudo service sshd restart
    
login with ssh and it's ask will you create an ECDSA key "yes"
    
    $ sudo ssh magic@10.11.1.200 -p 55556 (on VM terminal)
    
Make sure the connecion is active
    
    $ sudo systemctl status ssh
    
If you work on school computer then you might already have RSA key so don't need to re-create,

However if you do not have then:
    
    # host/computer terminal
    
    $ ssh-keygen -t rsa
    
Now on your own computer log in to the VM:
    
    $ ssh magic@10.11.1.200 -p 55556
    $ exit (logout from connection)
    
The next step is super important to keep secure our system:

Otherwise this could be a hall where hackers could get in easily and modify/delete services, take important datas away
    
[Disable ssh login for root user](https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user)
    
    $ /etc/ssh/sshd_config
    
change:
    
    # PermitRootLogin Yes

to:
    
    PermitRootLogin No
    commented out
    
## You have to set the rules of your firewall on your server only with the services used
outside the VM.
    
For this we use ```ufw``` (Uncomplicated FireWall)

Installing and activating the firewall:
    
    $ sudo apt install ufw
    $ sudo ufw status
    $ sudo ufw enable
    
now that we installed our [Firewall](https://opensource.com/article/18/9/linux-iptables-firewalld) we need to [allow](https://help.ubuntu.com/community/UFW) for our port to be able to go through, also for later the optional task we gonna need http services.
    
    $ sudo ufw allow 55556/tcp
    $ sudo ufw allow 80/tcp
    $ sudo ufw allow 442/tcp
    
now if we check the firewall status we can see these ports are allowed
    
    $ sudo ufw status
    
