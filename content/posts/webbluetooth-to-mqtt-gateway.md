+++
title = "Web Bluetooth to MQTT Gateway with Puck.js"
description = "Part 2 of my experiments with Puck.js and the Web Bluetooth API"
date = "2017-04-07T12:36:46+01:00"
tags = [
    "javscript",
    "iot",
	"puckjs",
	"webbluetooth",
	"mqtt"
]
categories = [
    "IoT",
]
+++

## If you read [A little bit of Web Bluetooth](https://slippytrumpet.io/posts/a-little-bit-of-web-bluetooth/) you'll know a bit about Puck.js and the new Web Bluetooth API already. But what, aside from building a gimmicky two factor authentication system, can you do with it?

I had some ideas. 

What about an interface through which you can program multiple Pucks at the same time? The Web IDE allows you to connect and program one Puck at a time. What if I want a few doing the same thing, it would be nice to set them up in one go? We'll call this a Console.

What about a Relay - whereby one Puck controls others in a master/slave configuration? Now you can do this directly between Pucks, but we could use Web Bluetooth to make a Relay.

What about individual Pucks, communicating via Web Bluetooth, with each other over the Internet using MQTT, or just controlled from a remote location with MQTT? A Gateway?

Three interesting ideas: a Console, a Relay and a Gateway. 

So here is [WebbleMQ](https://olliephillips.github.io/webbleMQ/), which is all of these things. It's a HTML5 & Javascript application which allows you connect multiple Pucks.

Each Puck is tracked on a connection ID, and depending on the mode that is set, it lets you interact with the connected Pucks in different ways.

<center>
![alt text](/images/webblemq.png "The WebbleMQ Interface")
Pictured. The WebbleMQ interface
</center>

Though I wrote it specifically for Puck.js, the Web Bluetooth API connection filter is quite broad. It looks for devices which, like the Puck, support the NORDIC UART bluetooth service. So it could well work for other devices which support this service. I haven't tested.

The source code for the application, as well as being readable in the web page, [can also be found on Github here](https://github.com/olliephillips/webbleMQ).

### Relay mode

One device, operating as Master, can control several devices which are configured as Slaves. 

For example, it's simple to turn the Pucks' onboard LEDs on and off with a program running on master device such as this:

```
var prog1 = "LED1.set();LED2.set();LED3.set();\n";
var prog2 = "LED1.reset();LED2.reset();LED3.reset();\n";
var on = false;
setInterval(function(){
  var prog = prog2;
  if(!on){prog = prog1;}
  Bluetooth.write(prog);
  on = !on;
}, 5000);
```
<br/>
### Console mode

In Console mode, all devices listen to input from the console. We can programatically control several devices at once, we just key our commands to the console window and the code is passed to, and executed on, all connected Pucks.

### MQTT modes. The fun stuff

We've two modes available here, "Announced" and "Unannounced". I'll explain the difference, but to be clear, neither mode is secure.

The MQTT broker used by default is the public `iot.eclipse.org` broker and though the topic ids used for publishing on are quite obscure in their format, e.g. `/wmq/pub/t7fldnrthv205btnafjzg`, this is not secure. If someone knows or guesses the topic id, they can listen and send on the same topic too. Don't send your bank details :)

That said, if you need security, you could fork the application and change the MQTT broker to one which provides authentication. I didn't need.

### MQTT "unannounced"

In this mode each device has its own publish topic and subscribe topic. If you connect multiple Pucks, then Pucks can subscribe to another Puck's publish topic, so they'd in effect be listening to it. 

### MQTT "announced"

Same as above, but device advertises its presence, both the topic it is publishing on and subscribed to, every 10 seconds on a third topic: `/wmq/playing`. 

This topic is used by all connected devices in MQTT announced mode. So, in announced mode someone could check this topic and find pucks they could control, those users happy to allow their Puck to be controlled remotely over MQTT.

## That's it

The appilication is not without a few glitches, it was a weekends work, and primarilly a proof of concept to help me get familiar with Web Bluetooth.

It could be improved. For example a free text box rather than select for MQTT topic would allow subscriptions to Pucks found publishing in `wmq/playing`. Better yet a check of the `wmq/playing` topic which adds all available devices to the publish and subscribe select box options. Feel free to fork it and send pull requests.

It's not especially practical either. You have to have a computer running a Chrome Browser in order to let your Pucks communicate over MQTT. A proper gateway would be a server application on a Raspeberry Pi for example.

But it is, hopefully, a bit of fun, which other Puck owners might want to experiment with, and which shows off the potential of Web Bluetooth a little more.


Enjoy!