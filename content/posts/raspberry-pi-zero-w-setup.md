+++
title = "Raspberry Pi Zero W \"headless\" Setup"
description = "Configuring Raspberry Pi Zero W for \"headless\" access, basic security and replication"
date = "2017-03-17T16:06:50Z"
draft = false

tags = [
    "raspberry",
    "pi-zero-w",
    "golang"
]

categories = [
    "IoT",
    "linux"
]

+++

![alt text](https://thingiverse-production-new.s3.amazonaws.com/renders/ee/4c/d9/f7/3d/326e57726fc5c2070bc5c19417e1814d_preview_featured.jpeg "Pi Zero rack/stand")
Pictured: My Raspberry Pi Zero W stand/rack. [Get it on Thingiverse](https://www.thingiverse.com/thing:2169943)

# An end-to-end log of the set-up process I followed from my Macbook

## What's this all about?

- Setting up a Raspberry Pi Zero W in a "headless" configuration, without a keyboard or monitor; 
- Configuring access to multiple Wifi access points;
- Securing via passwordless SSH public key only login and Firewall rules configuration;
- Simple command line Terminal access by configuring Hostname/Hosts;
- Creating a custom image, which we can reuse next time we want to configure another Pi Zero W.

To use the information here, you will need a Micro SD card, a full size adapter and an SD card USB drive, and of course, a Rasberry Pi Zero W.


## Why Raspberry Pi Zero W?

[Raspberry Pi Zero W](https://www.raspberrypi.org/products/pi-zero/) is a small, powerful and connected (onboard Wifi and Bluetooth) computer.  It's the only Raspberry Pi that's really caught my attention, including the original 'unconnected' Pi Zero. 

It's small; not much bigger than a NodeMCU [ESP8266](https://en.wikipedia.org/wiki/ESP8266) Wifi board, but the Pi Zero W also offers Bluetooth as well as better processor, RAM and storage resources.

It's cheap; just launched but only 9GBP. It will give [ESP32](https://en.wikipedia.org/wiki/ESP32) something to think on, a board which also has Wifi and Bluetooth, but to buy in a developer friendly form, 'currently' costs a bit more.

It's available; in the week it launched I ordered one, and I had it in my hands 2 days later. This makes it a strong contender in the space [C.H.I.P](https://getchip.com/) has positioned for itself, which, though arguably a better board with 4GB of onboard storage, is in very short supply.

It runs Linux; so, much of the stuff I do day-to-day is portable to the Pi Zero W.

It runs Go; Go compiles for lots of platforms, but STM32 architectures are not supported - the binaries are too big. So I can't use Go on ESP8266 boards, and I have no clue about programming in C. Fortunately for me, Go cross compiles to the Pi Zero W ARM architecture, just fine.


## Going "headless" 

I'm shooting for a "headless" setup. No screen and no monitor or other peripherals. I want to access my Pi Zero W like I do servers - over SSH. 

You can connect a monitor and keyboard initially to configure everything, but since I don't have either spare, I'm doing it without.

I'm on a Mac, so this log is Mac focussed. It would be easier on Linux - as you'll see :)


## We shoud start!

What follows is my set-up process, end to end. You should be able to follow it verbatim.

I didn't know some of this information initially, I used a couple of web resources myself in piecing it together. I've provided links to these resources where relevant.


### Step 1. Download Raspbian Linux image and make a bootable Micro SD card

We need a blank Micro SD card and adapter. And 8GB card seems to be the minimum requirement.

For a headless configuration we don't need the Pixel GUI/desktop that bundles with the full image, so we can grab the smaller 'lite' image. At time of writing latest was 'Jessie'. 

[Download it here](https://www.raspberrypi.org/downloads/raspbian/)

Run this command in Terminal, prefix with `sudo` if you need to:

```
diskutil list
```
<br/>
Note the lines in the output like these:

```
/dev/disk0 (internal, physical):
/dev/disk1 (internal, virtual):
```
<br/>
Plug the Micro SD card into the adapter and insert it into your SD card slot. Run the same command again. You should see an additional line in the output:

```
/dev/disk0 (internal, physical):
/dev/disk1 (internal, virtual):
/dev/disk2 (internal, physical):
```
<br/>
The additional line is your Micro SD card. 

Next, we'll unmount it, ready for flashing. 

```
sudo diskutil unmountdisk /dev/disk2
```
<br/>
In Terminal, change directory to where you have the downloaded Raspbian Jessie Lite image, which for me was Downloads:

```
cd ~/Downloads
```
<br/>
Use the ```dd``` command to copy the image to the Micro SD card. Note our use of ```rdisk2``` rather than ```disk2```. This means 'raw disk' and is supposed to be faster.

```
sudo dd bs=1m if=2017-03-02-raspbian-jessie-lite.img of=/dev/rdisk2
```
<br/>
You'll loose your Terminal prompt. Wait for the process to complete.

We now have a bootable Micro SD card. 

We could stick it in the Pi Zero W and boot it right away, but this wouldn't do us much good. With no keyboard or monitor, nor any network connectivity we can't interact with it yet.

**EDIT: 05/12/2017.** There are a couple of useful comments at the bottom of this post which suggest the process I describe next, including the need to create a virtual machine later on, can be simplified in general, and especially for Mac users on later versions of OS X. I haven't had the opportunity to try for myself, but I'd encourage you to check that information out also.


### Step 2. Access the Raspbian Linux image to edit files

The Raspbian image comprises two sub-partitions; a boot partition which is formatted as FAT32 and the file system partition, of type LINUX. 

If we mount and inspect the SD card in our Mac's Finder, we can only access the BOOT sub-partition. The LINUX filesystem partition is EXT4 format and our Mac Finder can't see/use it.

We can't mount the sub-partition in Terminal either on Mac, so we need to use Linux. 

A virtual machine is convenient here.

On Mac, an easy way to create a Linux virtual machine is via [Vagrant](https://www.vagrantup.com/) and [VirtualBox](https://www.virtualbox.org/), and [I mostly used this guide, by Jeff Geerling](https://www.jeffgeerling.com/blogs/jeff-geerling/mounting-raspberry-pis-ext4-sd) for this part of the process. 

[Install Vagrant](https://www.vagrantup.com/downloads.html), which will install the VirtualBox dependency.

Grab a recent Ubuntu Linux image using `vagrant init` and start the virtual machine.

```
vagrant init bento/ubuntu-16.04
vagrant up
```
<br/>
Suspend it.

```
vagrant suspend
```
<br/>
Launch VirtualBox, find the virtual machine and discard the saved state (via the right click option). We need to do this so that we can enable USB. 

Right click again and select 'Settings'. Choose 'Ports' and 'USB' then check 'Enable USB Controller'.

Plug the USB SD flash drive into one of the USB ports. Insert the Micro SD card (with the adapter if required) into the USB SD flash drive.

Check in Mac Finder that the card has not been mounted. If it has, unmount it.

Start the virtual machine from within VirtualBox and login as the `vagrant` user:

```
vagrant login: vagrant
Password: vagrant
``` 
<br/>
Click the USB option in the virtual machine window and select the USB Mass Storage device, which represents the USB SD flash drive/Micro SD card to give the virtual machine access to this drive.

If it is not selectable, double check in Finder that it has not been mounted.

Key the following in the virtual machine's Terminal prompt.

```
sudo fdisk -l
```
<br/>
This should identify both sub-partitions on the Micro SD card.

```
Device          Type
/dev/sdb1       Win95 FAT32 (LBA)
/dev/sdb2       Linux
```
<br/>
Mount the Linux sub-partition and change into this directory.

```
sudo mount /dev/sdb2 /media/usb
cd /media/usb
```
<br/>
We now have access and can edit files.

### Step 3. Edit files to pre-configure Wifi

The next step is to configure Wifi, so that when booted our Pi Zero W will connect to a network.

I typically operate between two Wifi access points, so I have set up both in the configuration below.

Maybe you only need to connect to one access point? I'd still recommend the configuration below, since it's extensible - there if/when you need it.

```
sudo nano etc/wpa_supplicant/wpa_supplicant.conf 
## Note no preceding "/". 
## Want to edit file on image not the virtual machine
```
<br/>
I'm working between two access points so configured `wpa_supplicant.conf` like this.

```
country=GB
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="SSID_OF_NETWORK_1"
    psk="password"
    id_str="home"
}

network={
    ssid="SSID_OF_NETWORK_2"
    psk="password"
    id_str="work"
}
```
<br/>
Key ```Ctrl+x``` to  confirm and save.

We need to make amendments to `etc/network/interfaces` also.

```
sudo nano etc/network/interfaces
```
<br/>
The defaults in here need to be amended for multiple access points and roaming between them.

Again this is my configuration, but a redundant config for a second unused access point should not hurt you - and you may need it later.

```
source-directory /etc/network/interfaces.d

auto lo
iface lo inet loopback

iface eth0 inet manual

allow-hotplug wlan0
iface wlan0 inet manual
    wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf

allow-hotplug wlan1
iface wlan1 inet manual
    wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf

iface home inet dhcp
iface work inet dhcp
```
<br/>
Save the file.

Unmount the Micro SD card in the SD card flash drive.

```
cd ../../
sudo umount /media/usb
```
<br/>
Next, we need to make a single amend to the first boot partition of the Micro SD card, since by default the SSH service is not enabled on Pi Zero. 

So we'll create an empty file named `SSH` with no extension. This is deleted at first boot, but SSH is subsequently enabled.

```
sudo mount /dev/sdb1 /media/usb
cd /media/usb
sudo touch SSH
cd ../../
sudo umount /media/usb
```
<br/>
Insert the Micro SD card into your Pi Zero W and power it up. If all went well you'll connect to an access point. If you configured a second access point, when you go to your second location you'll automatically connect to that access point too.

Hopefully, it should be evident how to add a third access point (or more) if needed.

### Step 4. Configure passwordless login over SSH

Now we have networking we can connect to the Pi Zero W itself over SSH. 

You can find the IP address of the Pi Zero W by logging into your router and looking at the list of attached devices. 

Alternatively, many routers with DHCP allocate an IP and expose the Pi Zero on the hostname `raspberrypi.local`

In Terminal, let's try connect to it. 

The default user is `pi` the default password is `raspberry`

```
ssh pi@raspberrypi.local // or ip address
password: raspberry
```
<br/>
Default passwords are never good. Let's sort, for both the `root` and `pi` users, by changing them.

```
sudo passwd root
Enter new UNIX password: verylongpasswdyoucannotsee
passwd: password updated successfully

passwd
(current) UNIX password: raspberry
Enter new UNIX password: verylongpasswdyoucannotsee
passwd: password updated successfully
```
<br/>
Update and upgrade Raspbian, this takes a few minutes to complete. Hit `y` in response to any prompts.

```
sudo apt-get update
sudo apt-get upgrade
```
<br>
Next we're going to set up SSH passwordless login. This is done by transferring our public key from our Mac to a special location on the Pi Zero W.

If you're a developer, it's very likely you'll have a private/public key pair already on your Mac, but if not, [you'll need to create one](https://docs.joyent.com/public-cloud/getting-started/ssh-keys/generating-an-ssh-key-manually/manually-generating-your-ssh-key-in-mac-os-x).

On your Pi Zero W create the following folder in your home directory.
<br/>
Then set the permissions.
```
mkdir /home/pi/.ssh
chmod 700 /home/pi/.ssh
```
<br/>
Create the `authorized_keys` file, and copy over your public key. You need to copy the contents of `~/.ssh/id_rsa.pub` into this file.
<br/>
Then set the permissions. 
```
nano /home/pi/.ssh/authorized_keys
chmod 400 /home/pi/.ssh/authorized_keys
chown -R pi:pi /home/pi
```
<br/>
Open a new Terminal window - !! do not close the current window !!

Attempt to connect.
```
ssh pi@raspberrypi.local
```
<br/>
This time we should log straight in to our Pi Zero W without being prompted for a password.

Finally, we will increase security by disabling all SSH password logins, the root user login, and restricting use of the SSH service to only the user `pi`.

```
sudo nano /etc/ssh/sshd_config
```
<br/>
Add, or amend, the following lines in this file.

```
PermitRootLogin no
PasswordAuthentication no
AllowUsers pi
```
<br/>
Restart the SSH service with `sudo service ssh restart`.


### Step 5. Configure a Firewall

Last step was a bit intense. Step 5 is nice and easy. We're going to install and set up a Firewall, through which we can block port access to our Pi Zero W.

We're going to install and use UFW (Uncomplicated Firewall). This application provides a super easy abstraction to iptables, an application which, personally speaking, I've often struggled with.

```
sudo apt-get install ufw
```
<br/>
Now we need to set some rules. 

I'll probably use my Pi Zero W as a webserver, maybe also as a client to consume API services, so I need HTTP and HTTPS, so I need to open ports 80 and 443.

I also use [Sendgrid](https://sendgrid.com/) to relay server email, and that works on port 587. 

Obviously I'll be using SSH to login in, and by default that uses port 22.

```
sudo ufw default deny incoming
sudo ufw allow 80
sudo ufw allow 443 
sudo ufw allow 587
sudo ufw allow 22
```
<br/>
Now, start UFW then check the configuration.

```
sudo ufw enable
sudo ufw status

Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
80                         ALLOW       Anywhere
443                        ALLOW       Anywhere
587                        ALLOW       Anywhere
22                         ALLOW       Anywhere (v6)
80                         ALLOW       Anywhere (v6)
443                        ALLOW       Anywhere (v6)
587                        ALLOW       Anywhere (v6)
```
<br>
We're firewalled. Feels good yes? 
<br/>
At this point our setup is fairly robust. I think so anyway.
<br/>
But we can do more to make using Pi Zero W a lovely experience.

### Step 6. Hostname configuration

I bet you, like me, have more than one Raspberry Pi Zero W - or will have soon!

They're handy things, and cheap. 

But, if we're going to use more than one Pi Zero W on the same Wifi network, we have a small problem.

Unless we want to keep identifying them by IP address, which means checking the router's attached device list, we need a way to tell them apart, as, by default, they all have the hostname of `raspberrypi`, addressed as `raspberrypi.local` over SSH.

This is not useful. So let's change it - it's simple.

```
sudo nano /etc/hostname
```
<br/>

Replace the `raspberrypi` value in this file with the name you want. There are restrictions on what `hostname` can be, but I name/number mine like this.
```
pizero1
```
<br/>
Confirm and save.

Next edit this file. 
```
sudo nano /etc/hosts
```
<br/>
Change the last line to match the entry you placed in the `/etc/hostname` file.
```
127.0.1.1       pizero1
```
<br/>
Repeat for all your pi zero w boards, and each will be separately identifiable and addressable on your Wifi network.

For example..

```
ssh pi@pizero1.local
ssh pi@pizero2.local
ssh pi@pizero3.local
```
<br/>
We're done. 

### Step 6. Creating an image of your work

We can't go through this long process - as much fun as it was :/ - every time we want to configure a new Raspberry Pi Zero W, so in this final step we're going to create an image file of everything we've done so far.

This image will give us a snapshot of the Operating System and File System, exactly as it is now. One that we can use, in place of the default Raspbian image, when we get another Pi Zero W and need an OS for it.

With an image, our next setup will just comprise of two simple steps!.

1) We'll just need to copy the image to a new Micro SD card as we did in Step 1, and then;

2) log in, and change the hostname just like we did in Step 5. 

Everything else including Wifi, SSH passwordless login and Firewall will be preconfigured out-the-box

To create the image we use the `DD` command from the Mac Terminal application. We unmount the Micro SD card first.

```
sudo diskutil unmountdisk /dev/disk2
sudo dd bs=1m if=/dev/rdisk2 of=pi/060317.img
```
<br/>
This creates the image file `060317.img` in the folder `pi` relative to the our current working directory.

To copy this image to a new Micro SD card for use in another Pi Zero W, we use the `DD` command again and simply reverse the parameters such that the source is `pi/060317.img` and the destination `/dev/rdisk2`.

```
sudo diskutil unmountdisk /dev/disk2
sudo dd bs=1m if=pi/060317.img of=/dev/rdisk2 
```
<br/>
That's it! We're all done!

Configuring a Raspberry Pi Zero W from this image is now effortless.

### Potential stumbling blocks

I've worked back through this process a couple of times now. I've had a few issues, which I've listed here as a heads up.

- Unmounting. If you get an unexpected error. Double check the Micro SD card is available, unmounted.

- SD Cards of same size. Turns out they are not always. So a 32GB image written back, to a new 32GB card might not always fit. I've seen this with a cheap 32GB Micro SD card

- SD cards of different sizes. Apparently you [can write an image taken from a 32GB card back to a 16GB card, or a 16GB to an 8GB](http://raspberrypi.stackexchange.com/questions/1058/is-this-an-ok-way-to-transfer-my-16gb-sd-card-to-an-8gb-sd-card-simple-dd?rq=1). The data of the pi image is much less than 8GB. You can also do it the other way round, but by default you lose access to the additional space outside the image. Use `sudo rasp-config` to expand the root partition to reclaim this space.

- Imaging takes a while. Unless you really need lots of space, working with 8GB cards is quick, consistent and cheap!

- SSH known hosts. You might be prompted that an entry exists which doesn't match the new entry if you're SSHing to a new Pi Zero W. Just follow the filepath and delete the line. It will then work fine.