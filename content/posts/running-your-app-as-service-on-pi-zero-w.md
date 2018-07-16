+++
title = "Running your Application as a Service on Pi Zero W"
description = "Daemonize your application so that it's easily started, stopped and resilient to crashes"
date = "2017-06-08T10:27:15+01:00"

tags = [
    "raspberry",
    "pi-zero-w",
    "golang",
	"daemon",
    "systemd",
    "linux"
]

categories = [
    "IoT",
	"linux"
]
+++

# Daemonize your application so that it is easily started, stopped and resilient to crashes

So you've got your headless Raspberry Pi Zero W all set up and you've been running applications you've written for yourself on it? I have, I'm using Go with a combination of [Embd](http://embd.kidoman.io/) and [Periph.io](https://periph.io/) libraries to interact with the hardware on the Pi Zero W itself and external to it.

But it feels clunky. 

Running "headless", with no screen or keyboard, everytime I boot the device I need to SSH to it and kick off the application from the command line. 

One of the apps I have monitors temperature using Onewire and two DS18B20 temperature probes. To have to SSH to the Pi Zero W to run the program that does this, is inconvenient: I need access to a computer with a Terminal, and the Pi Zero W must be accessible over the network via SSH. This won't always be possible.

Much better would be for me to simply switch on the Pi Zero W and have the program run automatically. Plug and Play.

We can do this, and more, by running the program as a service, often referred to as "daemonizing".

## How to daemonize?

There are [Go packages that you can include in your codebase to emulate running as a service](https://github.com/sevlyar/go-daemon), and whilst I looked into these, this post does not cover them, for there is a simpler way, one which is useful for a program written in any language, not just Go.

### systemd

systemd is a service manager for Linux which ships with Raspbian, so it's already installed on your Pi Zero W and allows us to easilly daemonize our application.

Running the application as a service has several benefits.

1) We can start, stop and ascertain the status of our application service with simple commands.

2) systemd will monitor and restart the service in the event our program crashes, so our application is more robust to unforseen errors and bugs.

3) systemd will start the application for us once our Raspberry Pi W has booted - subject to a few constraints - so we'll no longer need to SSH to the device to kickstart the program.

Certainly sounds like the way to go!

## Step 1 - Create a systemd configuration file for your program

systemd needs to know a few things about your application, so you need to give it a configuration file which provides the information required.

Configuration files have the `.service` extension and are stored in this location on your Pi Zero W `/etc/systemd/system/`

Change directory to `/etc/systemd/system` and use `sudo nano` to create a new file, with the `.service` extension. For example I have `tempmon.service`.

In this file you need to add a few directives. This is my `tempmon.service` file, simply amend to suit your application.

```
## tempmon.service

[Unit]
Description=Temperature Monitor
After=network.target

[Service]
User=root
WorkingDirectory=/home/slippytrumpet
ExecStart=/home/slippytrumpet/tempmon
Restart=on-failure
StartLimitIntervalSec=1800

[Install]
WantedBy=multi-user.target
```
<br/>
The directives, for the most part, need no explanation, but a couple bear comment.

`After=network.target`, means systemd will run the program once networking is available.

`User=root`, sets the user that the process is to be run as. Since my temperature monitoring application is interfacing with hardware on the Pi Zero W, it needs root privileges.

`StartLimitIntervalSec=1800`, by default the application will be restarted if it crashes, but if it crashes more than the default of 5 times, there will be a pause of 1800 seconds (30 minutes) before systemd attempts to start it again.

The MAN file for systemd contains all the directives which can be used and [you can find an online version of that, here](https://www.freedesktop.org/software/systemd/man/systemd.directives.html).

## Step 2 - Enable the config file

To enable your new config file, so that systemd daemonizes your program, use this command (substituting the name of your file in place of `tempmon`):

```
sudo systemctl enable tempmon
```
<br/>
If you edit the file once enabled you will need to reload it with this command:

```
sudo systemctl daemon-reload
```
<br/>
Now if you reboot your Pi Zero W, your program will be started by systemd. You can check it's status with the command:

```
sudo service tempmon status
```
<br/>
And, you can stop and start the program just like any other Linux service:

```
sudo service tempmon stop
sudo service tempmon start
```



