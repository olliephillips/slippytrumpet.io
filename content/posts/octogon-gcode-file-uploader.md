+++
title = "Octogon G-Code File Uploader"
description = "Small utility written in Go which moves G-code files from your local computer to your OctoPrint over SSH"
date = "2017-12-05T23:05:39Z"

tags = [
    "raspberry pi",
    "golang",
	"gcode",
	"3dprinting",
	"octoprint",
]

categories = [
    "3D Printing",
]

+++

## An automatic file upload utility for OctoPrint

Octogon is a simple command line utility which will monitor a single folder on your local file system for the addition/modification of ```.stl``` and ```.gcode``` extension files and automatically copy them to OctoPrint so they appear in your files list.

### Why?

My 3D Printer is right next to me at the moment. But getting files onto it, configuring it, and monitoring it, are jobs that are a bit painful in this wireless age. Everything is done via the LCD control panel and SD card.

So when I found out about [OctoPrint](http://octoprint.org/), a print server for 3D Printers which runs on a Raspberry Pi, I did it. I bought a Raspberry Pi and installed [OctoPi](https://octopi.octoprint.org/), the prebuilt Raspberry Pi image which includes OctoPrint as well as some other utilities.

OctoPrint has been a relevation. It offers remote monitoring and control of the 3D printer over LAN - and WAN so long as you configure your router's firewall correctly  - so you can view, monitor, tweak and pause/stop a print job, over a wifi connection.

[Octogon](https://github.com/olliephillips/octogon) is a simple utility written in Go. It addresses one tiny weakspot in my OctoPrint setup and workflow: the need to physically upload my G-code files via OctoPrint's web interface. Using Octogon, they just appear in the file list and are available for printing.

With Octogon running. A single folder on my computer is monitored. Any new or modified `.gcode`and `.stl` files saved into that folder are copied over to OctoPrint automatically using SSH. Save a file locally, and it's almost immediately available on the Raspberry Pi.

Using it is simple. In most cases the only command line option needed to run the program is the password `-p` flag. 

For example, you can start Octogon monitoring the current folder like this:

```
$ octogon -p sshpasswordtoraspberrypi
```
<br/>

### Download Octogon

You can checkout the documentation and source code, as well as download compiled binaries for Linux and Mac, [here](https://github.com/olliephillips/octogon). 

Unfortunately the Windows build currently doesn't work - the file monitoring is not reliable - and I'll look into this in the near future.