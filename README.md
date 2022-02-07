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
    

## You have to set a DOS (Denial Of Service Attack) protection on your open ports
of your VM.
    
For different type of attack has been created different kind of security services
We gonna use
    
[Fail2ban](https://en.wikipedia.org/wiki/Fail2ban), to secure our server
    
[iptables](https://en.wikipedia.org/wiki/Iptables), to configure and filter out dangerous IPs
    
[apache2](https://www.hostinger.com/tutorials/what-is-apache), to deploy our website in the comming task
    
    
```$ sudo apt install iptables fail2ban apache2```
    
After the installation we make a copy from jail.conf file to jail.local

The reason is that, beacuse with upgrades it might be modified so we want to keep the present one to keep our changes.
    
    $ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
    $ sudo vim /etc/fail2ban/fail2ban.local
    
Here we scroll down or write to the search bar (:/sshd) to locate ```# SSH servers```.
    
adding more rules: 
    
    enable = true
    maxentry = 3
    bantime = 600
    
[Check out for more details](https://www.tothenew.com/blog/fail2ban-port-80-to-protect-sites-from-dos-attacks/)
    
Now we write rules to protect port 80 (http) (our future website from dos attacks)
search for ```zoneminder````

add: 
    
    # protect port 80 (HTTP)
    
    [http-get-dos]
    
    enabled = true
    port = http, https
    filter = http-get-dos
    logpath = %(apache_error_log)s
    maxentry = 300
    findtime = 300
    bantime = 600
    action = iptables[name=HTTP, port=http, protocol=tcp]
    
Go to: ```/ect/fail2ban/filter.d/http-get-dos.conf```
    
Add:
    
    [Definition]
    
    failregex = ^ -.*GET
    ignoreregex =
    
 [For more detail check out](https://serverfault.com/questions/725936/fail2ban-regex-filter-doesnt-work-with-nginx-log-files)
    [and also this](https://docs.python.org/2/library/re.html)
    
after you made the changes we need to restart the firewall:
    
    $ sudo ufw reload
    $ sudo service fail2ban restart
    
now you make sure everything is setted up so:
    
    $ sudo fail2ban-client status
    
## You have to set a protection against scans on your VM’s open ports.
    
    $ sudo apt install portsentry
    
After the installation, edit the file ```/etc/default/portsentry```.
So add +a (advance):
    
    TCP_MODE="atcp"
    UDP_MODE="audp"
    
We also wish that portsentry is a blockage. We therefore need to activate it by passing BLOCK_UDP and BLOCK_TCP to 1 as below :
```/etc/portsentry/portsentry.conf```
    
    ##################
    # Ignore Options #
    ##################
    # 0 = Do not block UDP/TCP scans.
    # 1 = Block UDP/TCP scans.
    # 2 = Run external command only (KILL_RUN_CMD)

    BLOCK_UDP="1"
    BLOCK_TCP="1"
    
Now comment out every comment which starting with "KILL_ROUTE" except this one:
    
    KILL_ROUTE="/sbin/iptables -I INPUT -s $TARGET$ -j DROP"
    
for veryfication:
    
    cat portsentry.conf | grep KILL_ROUTE | grep -v "#"
    
restart the portsentry service so it will start to blocking port scans:
    
    $ sudo /etc/init.d/portsentry start
    
now you can check to make sure it is running and active:
    
    $ sudo service portsentry status
    
[How to protect against the scan of ports with portsentry](https://en-wiki.ikoula.com/en/To_protect_against_the_scan_of_ports_with_portsentry)
    
## Stop the services you don’t need for this project.
    
all the services [you will find](https://linuxhint.com/disable_unnecessary_services_debian_linux/) in ```/etc/init.d```
    
    $ ls /etc/init.d
    
example I don't need console setup, because that's just responsible for fonts size etc.. so with disable i m gonna delete it:
    
    $ sudo systemctl disable console-setup.service
    
## Create a script that updates all the sources of package, then your packages and
which logs the whole in a file named /var/log/update_script.log. Create a scheduled
task for this script once a week at 4AM and every time the machine reboots.
    
    $ sudo touch update.sh
    $ sudo chmod a+x update.sh

in the update.sh to keep updating everything:
    
    sudo apt update -y >> /var/log/update_script.log
    sudo apt upgrade -y >> /var/log/update_script.log
    
so in the shell file we telling where to keep the logs from the updates
And now we tell for cron ```when``` to do these tasks.
So open crontab in edit mode:
    
    $ sudo crontab -e
    
write:
    
    @reboot /home/magic/update.sh &
    0 4 * * MON /home/magic/update.sh &
    
In the cron service you will find also a small guide how to use it as well.
    
    
