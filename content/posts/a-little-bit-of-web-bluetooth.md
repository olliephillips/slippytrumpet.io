+++
title = "A little bit of Web Bluetooth"
description = "Experimenting with the new Web Bluetooth API and Puck.js."
date = "2017-02-09T15:08:26+01:00"
tags = [
    "espruino",
    "webbluetooth",
	"mqtt",
	"websockets",
	"puckjs"
]

categories = [
    "IoT",
	"Web Development"
]
+++

##  Communicating using the Bluetooth protocol has traditionally been the sole preserve of native applications. Now you can do it from a web page!

This is the first of two short posts on my experiments with [Puck.js](http://www.espruino.com/Puck.js) and [Web Bluetooth](https://developers.google.com/web/updates/2015/07/interact-with-ble-devices-on-the-web#before_we_start). This one looks at security; the second will look at a [Web Bluetooth to MQTT gateway](https://olliephillips.github.io/webbleMQ/) experiment.

### What's Web Bluetooth?

Some may know, that from Google Chrome v56, the Web Bluetooth API went from being an experimental "opt-in" feature to enabled by default. It supports communication between devices that implement Bluetooth 4.0 (and above) and uses Bluetooth Low Energy (BLE), a more power efficient protocol.

As stated in the introduction earlier, once the preserve of native applications, now Web Developers can build applications which talk to physical hardware over BLE from websites - applications which match the capabilities of native apps.

This is exciting.

### Is it secure?

Since the Chrome 56 release, some [reports have cited security and privacy concerns](https://www.theregister.co.uk/2017/02/05/chrome_56_quietly_added_bluetooth_snitch_api/), and many people are clamouring to turn the feature off.

But should they be concerned? In my view, no.

It's easy to surmise that unscrupulous website owners might try and take advantage of this; that device detection could be used as a mechanism for tracking - just like cookies - but the mechanics of the Web Bluetooth API seem to have security firmly in mind. It looks like it would be hard to abuse from an application, both intentionally and inadvertently.

First, it's HTTPS only. You can't use Web Bluetooth unless your website is served securely.

Second, to initiate a device scan, it has to be in response to a user action such as a mouse click or finger swipe. You can't trigger it directly from code - and you can't emulate these events programmatically to fool the Web Bluetooth API.

However, one might argue that website visitors could be fooled.

For example, what if you attach an `onClick` event listener to the ```<body>``` element of the DOM? Would a mouse click anywhere on the page initiate a device scan? 

It would.

But hold your horses, so you can do this, is there any real risk? I'd say 'no' again. 

To understand why, we need to look at what happens next. We need to look at how a device scan works; how does it notify the user  of devices in range which it can connect to, and; more importantly, at what stage does the browser have access to any information about these devices (specifically, we're concerned about information exposed by the Web Bluetooth API object - about scanned devices - which could be stored, used for connection attempts, reading [service characteristics](https://www.bluetooth.com/specifications/gatt/characteristics), or just just 'plain evil' tracking?

Let's see.

This code initiates a Web Bluetooth device scan. You have to set it up as callback to an event listener like `click`, but once the event triggers, a 'device chooser' pop-up will appear near the address bar of the Chrome Browser listing devices in range.

```
navigator.bluetooth.requestDevice({
	acceptAllDevices: true
}).then(device => { 
	console.log(device)
}).catch(error => { console.log(error);});
```
<br/>

The above will list all BLE devices in range, but in your application, you can define filters so that only the types of device your app can interact with, will appear. For example, use a name prefix if devices have a common name, or you can filter on [GATT service](https://www.bluetooth.com/specifications/gatt/services) (GATT represents "Generic Attribute Profile", and broadly put, GATT services describe what a device is able to do).

Following the scan, the user is able to pick the device they want the web page to connect to, but until they do, our script can't process code to connect, and our script/web page knows NOTHING about the devices in the device chooser, the devices in range.

Also that 'device chooser'.. it's a part of the Chrome browser application, and nothing to do with the webpage and its DOM. So you can't interact with it from code.

From here the user can choose a device, or cancel. I've noticed ignoring the device chooser and doing something else like opening the Developer Console will trigger 'cancel' too - I don't know what other actions will do this.

In the above code 'cancelling', however activated, will cause an error to be logged to console. 

If the user chooses to connect to a device, then a ```BluetoothDevice``` object is passed to the script which, for my Puck device, contains the following information.

```
BluetoothDevice {
	id: "0L3/Rdv+ZzblpiXmJPlO3A==", 
	name: "Puck.js bb18", 
	gatt: BluetoothRemoteGATTServer, 
	ongattserverdisconnected: null
}
```
<br/>
But, remember, I allowed the web application to see this, by allowing my browser to connect to it.

This device scan & connection process must be followed every time the user wants to connect a device - as far as I can tell there is no memorizing, nor any pairing.

So this looks fairly secure to me. Hard for web developers to abuse. 

Can networks of websites leverage Web Bluetooth? Like they might do cookies?

I don't know, but in my testing, the `BluetoothDevice.id` is not a constant. It seems to be random and rotates on connection. So, if trying to track JUST allowed/connected devices, you could not rely on `id` website to website. `BluetoothDevice.name` can be constant though.

Can Google see any data...? What if you compromise Chrome...? Is the API secure...? We'll all know in time.

But, the last point I want to make on Web Bluetooth, is that where Web Bluetooth offers no data on, nor any programmatic access to, devices in range.. a native app, written in Node, Go, C or similar, running on hardware as accessible and cheap as a Raspberry Pi 3, will give you a lot of info - without asking for any permission whatsoever!

So, if you're going to be tracked unknowingly via your Bluetooth devices, it's less likely to be via a website, than from a cheap home built Bluetooth scanner, running out of someone's living room.

### So, that said, what can you do with Web Bluetooth?

Since security is the general theme of this post, I've built a totally impractical 2 factor authentication demo which uses Puck.js as a hardware fob to authenticate the user, a bit like how the banks do with online account access.

It's impractical since it's all client side, so not secure, but some server calls could remedy this. But you'd still need a Puck to use it!

You can see the [interface here](https://olliephillips.github.io/areyoupucker/). The website sends a random four colour sequence to the Puck (which has three onboard LEDs - Red, Green and Blue). 

The sequence, once replicated on the web form interface, unlocks the form so that it can be submitted.

Video below shows how it works.

<blockquote class="instagram-media" data-instgrm-captioned data-instgrm-version="7" style=" background:#FFF; border:0; border-radius:3px; box-shadow:0 0 1px 0 rgba(0,0,0,0.5),0 1px 10px 0 rgba(0,0,0,0.15); margin: 1px; max-width:658px; padding:0; width:99.375%; width:-webkit-calc(100% - 2px); width:calc(100% - 2px);"><div style="padding:8px;"> <div style=" background:#F8F8F8; line-height:0; margin-top:40px; padding:50.0% 0; text-align:center; width:100%;"> <div style=" background:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACwAAAAsCAMAAAApWqozAAAABGdBTUEAALGPC/xhBQAAAAFzUkdCAK7OHOkAAAAMUExURczMzPf399fX1+bm5mzY9AMAAADiSURBVDjLvZXbEsMgCES5/P8/t9FuRVCRmU73JWlzosgSIIZURCjo/ad+EQJJB4Hv8BFt+IDpQoCx1wjOSBFhh2XssxEIYn3ulI/6MNReE07UIWJEv8UEOWDS88LY97kqyTliJKKtuYBbruAyVh5wOHiXmpi5we58Ek028czwyuQdLKPG1Bkb4NnM+VeAnfHqn1k4+GPT6uGQcvu2h2OVuIf/gWUFyy8OWEpdyZSa3aVCqpVoVvzZZ2VTnn2wU8qzVjDDetO90GSy9mVLqtgYSy231MxrY6I2gGqjrTY0L8fxCxfCBbhWrsYYAAAAAElFTkSuQmCC); display:block; height:44px; margin:0 auto -44px; position:relative; top:-22px; width:44px;"></div></div> <p style=" margin:8px 0 0 0; padding:0 4px;"> <a href="https://www.instagram.com/p/BQTLhASA27a/" style=" color:#000; font-family:Arial,sans-serif; font-size:14px; font-style:normal; font-weight:normal; line-height:17px; text-decoration:none; word-wrap:break-word;" target="_blank">Are you pucker?</a></p> <p style=" color:#c9c8cd; font-family:Arial,sans-serif; font-size:14px; line-height:17px; margin-bottom:0; margin-top:8px; overflow:hidden; padding:8px 0 7px; text-align:center; text-overflow:ellipsis; white-space:nowrap;">A video posted by Ollie (@slippytrumpet) on <time style=" font-family:Arial,sans-serif; font-size:14px; line-height:17px;" datetime="2017-02-09T17:44:45+00:00">Feb 9, 2017 at 9:44am PST</time></p></div></blockquote>
<script async defer src="//platform.instagram.com/en_US/embeds.js"></script>


I've also written a [Web Bluetooth MQTT gateway](https://olliephillips.github.io/webbleMQ/) which facilitates master/slave communication between Pucks, console control of multiple Pucks at the same time, as well as communication of data over MQTT.

I will cover this in another post.









 




