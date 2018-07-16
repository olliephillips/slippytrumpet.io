+++
title = "Building a Remote Control Car with Espruino on ESP8266"
description = "'We can make this car work again - but over Wifi', I told the kids. And that was my first mistake."
date = "2016-11-16T09:18:08Z"
tags = [
    "espruino",
    "esp8266",
	"websockets",
    "riotjs",
	"html5",
	"nodeMCU"
]

categories = [
    "IoT"
]
+++

## A broken radio controlled microcar; just lying in a box, useless. A little knowledge of Espruino and winter nights closing in. 

## "We can make this car work again - but over Wifi", I told the kids. And that was my first mistake.

First, one of those, "blimey this is a long post; maybe I won't bother" things.. AKA "TL;DR".

Espruino/JavaScript code which runs on a NodeMCU ESP8266 microcontroller and does this..

1. Provides motor and steering control functions on NodeMCU ESP8266 pins connected to a dual H-bridge module;
2. Includes a HTML page to serve which allows us to control the car from a Smartphone on the same Wifi network (tested on iPhone);
3. Creates a http/websocket server, which serves the HTML page in (2) and creates a websocket connection which listens for message events from (2) and passes them to (1);
<br/><br/>
..can be [found here](https://github.com/olliephillips/espruinoCar). 
<br/><br/>
You're still here? Excellent.

## What is Espruino?

Let's cover this first. [Espruino](http://www.espruino.com) is a JavaScript interpreter on a microcontroller.

The project itself comprises both the official microcontroller hardware - there are three different boards: [Espruino, Espruino Pico, and the latest, Espruino Wifi](http://www.espruino.com/Order) - and the firmware binary which is a C application that gets flashed (written) to the microcontroller.

The microcontroller boards have a number of pins that can be written to and read from - their voltage determining their state. The pins can be connected to "things" so you can literally write JavaScript to control them.

Espruino is open source, the firmware can be freely downloaded and there are builds for other microcontrollers based on a similar chip architecture. There are even builds for Linux, so you could use Espruino with a Raspberry Pi.

Implementations vary though. Some builds may not give you all the features of Espruino, and only the official Espruino boards are fully supported and guaranteed to work "out-the-box". It's also worth noting that some boards won't offer as much memory as the official boards which can limit their usefulness for some applications.

For this project I used an ESP8266 NodeMCU development board. They are cheap, have built in Wifi, and enough memory for my application.

Before I move on, I should emphasise that Espruino is funded through donations and sales of official boards. Though I'm choosing ESP8266 here, I own plenty of the original boards - two Espruino, three Picos and one Espruino Wifi in fact.

When starting out it's a really good idea to get an official board just because they are plug and play. You'll get good support in the [Espruino forums](http://forum.espruino.com) too, plus you'll find them useful when prototyping even if your project will eventually run on unofficial hardware.

## The objective

Essentially to drive the car over Wifi, maybe even remotely over the Internet, using a web page to control it. For communications, MQTT was my first inclination since I could leave the RC car behind a firewall, whereas using Websockets and a HTTP server I'd have to expose the car to the world via port forwarding. But more on this later.

## First attempt

Our donor car was a radio controlled microcar not much bigger than a matchbox car - remember those? I think the scale was something like 1:42.

The steering mechanism of the RC microcar was broken, reason unknown - but it only steered one way. On inspection the steering was operated using two solenoid coils and a magnet on a steering rack. With current applied to each coil the steering rack moved left and right - or should have.

Without using the circuit board of the RC car, I did not understand how I could control the solenoid setup - even if it wasn't broken, and what if it was the circuit board itself that was faulty? 

So my first decision was to remove the solenoids, magnet and the circuit board from the car.

In their place I sourced a tiny micro servo on ebay, glued it in and created a simple linkage to the steering rack with stiff wire.

Steering was sorted, but this was a fiddly job and the addition of the servo made it likely the car's body would not fit back on without modification.

![alt text](/images/micro-servo.png "Micro servo. Fiddly.")
Pictured: Micro servo, attached to steering rack

### Next issue, "drive"
After some help from a couple of people in the Espruino forums I better understood what was involved in controlling the motor - which required switching the supply polarity to provide forward and reverse motion.

My first attempt used multiple ESP8266 pins, in pairs which I switched as groups between high and low state, essentially wiring them in parallel to hopefully provide sufficient current to drive the motor.

Unfortunately this didn't work.  The motor was rated 75mA at 4.5v. The ESP8266 NodeMCU board could supply 12mA at 3.7v per pin. I wasn't going to have enough pins to get that current.

At this point I wondered about using my Espruino Wifi instead, which I believe can provide 20mA per pin, possibly enough when grouped in parallel. And maybe I would have gone this route, had discussion in the forums not focussed my attention on something call a "H-bridge".

A H-bridge is a circuit which operates like a 3 way switch. It allows polarity of voltage to be reversed, which when connected to a motor, rotates it in opposing directions - forward and reverse for our purposes.

After more discussion, and the prospect of making a H-bridge circuit myself from the component parts, I chickened out and went back to ebay where I found a dual H-bridge module for about £1.50 delivered. To control our motor we needed only one of the H-bridge channels. 

But there was no way the car's shell was going to fit back on now, modified or not.

![alt text](/images/h-bridge-with-motor.png "H-bridge attached to motor")
Pictured: L9110S Dual H-bridge module

I'm not going to cover wiring, pin selection nor code yet, since this is not how the car ended up and could confuse. 

I'll cover all this later, but here's the finished "first attempt". Now devoid of shell, obviously I went to a lot of trouble to make it look like a car again. Check that rear spoiler out!

![alt text](/images/microcar.png "RC Microcar")
Pictured: Micro servo, 3.7v li-ion battery, NodeMCU ESP8266, H-bridge

At this point we had a working car and a basic code library, which combined allowed us to control the car over Telnet (wireless console) to send stop/go/direction commands. 

We got about 20 minutes out of it. Then the car ceased to steer. The micro servo was turning but did not make its stops with the control arm. I had a look inside and it appeared one of the tiny gears had stripped its teeth. I could have ordered a replacement micro servo, but for it to potentially happen again. So this didn't seem like a great idea.

Fortunately, my kids were now interested enough in the potential of the project, that they donated another RC car.

Actually they sold it to me :/

## Second attempt

One bit of feedback from my kids. Despite my efforts, it didn't look like a car. The broken RC microcar was previously a Lamborghini but look what I had turned it into - a monster.

Fortunately, the new donor vehicle, a Mini, was much bigger. We would have no issues with space or refitting the shell. 

Two other plus points: first, steering on this car was motor driven. I wouldn't need to add a servo or scratch my head over solenoids, we could use the second channel of the H-bridge to control the steering motor and second; power. The car carried 4 AA batteries, so supplied 6v. I could run that to the 5v power input on the NodeMCU board and we would have enough for everything.

## Building

Building the second car was almost trivial. Most of the learning had been done with the first car, and there was so much room to play with.

Power was from the batteries and the two motors had wires we easilly extended and routed to the two channels of the H-bridge. All the components were mounted with hot glue. 

Wiring had to be modified slightly to accomodated a second motor attached to the H-bridge, but this second build probably only took about 30 minutes.

We added a buzzer to work as a horn which, as well as being controllable over wifi, we decided would serve to notify us once the car had successfully connected to the Wifi network - there'd be no point trying to control the car until it had.

![alt text](/images/mini.png "Mini")
Pictured: RC Mini shell and floorpan, with the NodeMCU and H-bridge module and buzzer glued in.

The code library also needed to be modified to control the steering. You can find the [full library on Github here](https://github.com/olliephillips/espruinoCar), but here's the control piece.

```
// Control methods for espruinoCar 'ec'
var ec = {
  pin: {
    forward: D4,
    reverse: D5,
    left: D14,
    right: D0,
    horn: D12
  },
  left: function() {
    digitalWrite(ec.pin.left, 1);
    digitalWrite(ec.pin.right, 0);
  },
  right: function() {
    digitalWrite(ec.pin.right, 1);
    digitalWrite(ec.pin.left, 0);
  },
  ahead : function() {
    digitalWrite(ec.pin.left, 0);
    digitalWrite(ec.pin.right, 0);
  },
  drive: function(speed, time){
    if(time){
      setTimeout(function(){
        analogWrite(ec.pin.forward, 0);
      }, ec.convert.time(time));
    }
    analogWrite(ec.pin.forward, ec.convert.speed(speed)); 
    analogWrite(ec.pin.reverse, 0);
  },
  reverse: function(speed, time) {
    if(time){
      setTimeout(function(){
        analogWrite(ec.pin.reverse, 0);
      }, ec.convert.time(time));
    }
    analogWrite(ec.pin.reverse, ec.convert.speed(speed)); 
    analogWrite(ec.pin.forward, 0);
  },
  stop: function() {
    analogWrite(ec.pin.forward, 0);
    analogWrite(ec.pin.reverse, 0);
    digitalWrite(ec.pin.left, 0);
    digitalWrite(ec.pin.right, 0); 
  },
  horn: {
    toot: function() {
      setTimeout(function(){
         digitalPulse(ec.pin.horn, 1, 200);
      },300);
      digitalPulse(ec.pin.horn, 1, 200);
    },
    honk: function() {
      digitalPulse(ec.pin.horn, 1, 1000);
    }
  },
  init: function() {
    pinMode(ec.pin.left, "output");
    pinMode(ec.pin.right, "output");
    pinMode(ec.pin.forward, "output");
    pinMode(ec.pin.reverse, "output");
    pinMode(ec.pin.horn, "output");
    analogWrite(ec.pin.forward, 0);
    analogWrite(ec.pin.reverse, 0);
    digitalWrite(ec.pin.left, 0);
    digitalWrite(ec.pin.right, 0); 
  },
  convert: {
    speed : function(speed) {
              var convSpeed = speed/100;
              return convSpeed;
    },
    time : function(time) {
             var convTime = time* 1000;
             return convTime;
    }
  }
};
```
<br/>
One thing to note. The pin numbers on the NodeMCU board do not relate to the GPIO pin numbers you need to access them via Espruino, which makes wiring more complicated than it should be.  So for clarity the pin references in the excerpt above all relate to the GPIO pin numbers, not the numbers printed on the board.

The code is fairly self-explanatory I should think. The drive and reverse methods accept ```speed``` and ```time``` arguments, so the car can be programmed to cover a course with this library alone should we wish.

There are also two utility functions; one which relates speed to a number between 0 and 1, the parameter range for the ```analogWrite()``` function and a second, which converts time to a number in milliseconds for the ```setTimeout()``` function.

## Communicating over MQTT

I mentioned earlier that we wanted to control the RC car over Wifi, possibly remotely over the web too. 

MQTT seemed a good choice for communication. The RC car could subscribe (listen) to a topic on a MQTT broker (of which there are many free/test ones) and our client on iPhone would publish short control messages to the MQTT broker via a websocket connection.

The advantage was that we didn't need a direct HTTP connection to the car - something that could be abused. With MQTT the RC car would not serve anything but instead simply listen for messages. It could do this from behind the Wifi router's firewall too.

![alt text](/images/mqtt.png "MQTT testing")
Pictured: Publishing control messages to our RC car.

However I decided it was unlikely we'd control the car over the Internet in reality, so it would be sufficient to either connect the RC Car to Wifi, or run the ESP8266 NodeMCU as an access point - whereby it creates a Wifi network we can join, and can expose HTTP on the IP address ```192.168.4.1```.  

Using the ESP8266 as an access point would necessitate that it serve all the code and assets we'd need for our web based control panel too. While possible this would be significantly more complex, since there's no filesystem on Espruino as such. We'd need to persist the assets by writing them to flash memory and we'd need to serve them by reading them back from flash memory. 

So I elected to use the ESP8266 in station mode, connecting to an existing Wifi network so that any client accessing the HTML control panel page could download the assets required by the page from the Internet.

We'd still use websockets to send data, but to a websocket server running on the ESP8266 rather than relaying via an MQTT broker.

## The Controller 

First things first. Since there was no display on the car, and no communication about the Wifi connection other than a horn toot when connected, it would be helpful to know the IP address in advance. 

Typically routers will assign IP addresses on a "first come first served" basis, meaning our car's IP address would be random, depending on what else was on the network when we switched it on. With no display we'd either have to guess it or connect to the Router every time to determine it.

There's a better way - so the first thing I did was connect the ESP8266 NodeMCU to my Wifi network. Once identified in the Router control panel, I assigned it a specific 'reserved' IP address. 

Now it would always join the network on the same IP address, so I knew how to reliably connect to the HTTP/Websocket server I was about to write.

### Introducing iotUI.js

[iotUI.js](https://github.com/olliephillips/iotUI.js) is a suite of web component controls that I started writing last year. The controls up until this point were mainly aimed at beer brewing so included things like a thermometer and level guages. The interfaces themselves are SVG graphics, which respond to touch and mouse events. 

iotUI.js is built on [Riot.js](http://riotjs.com), which is like React, but much easier to get along with in my opinion. As a library it's also more lightweight than React. 

Riot.js allows us to create custom HTML components referred to as "tags" and specify all the logic for how the control works in the tag itself.

To cut what could be a long story, short, all the complexity of adding, interfacing and interacting with the component is abstracted away from the end user. Adding a Riot component to your HTML page that may take input, or trigger events is pretty much as simple as adding a single HTML tag.

I decided to build my control interface with Riot.js, but then went a step further and implemented the component - [a steering wheel](https://rawgit.com/olliephillips/iotUI.js/master/examples/f1wheel.html) - in iotUI.js. 

Should you wish, you can easilly implement it in your projects too. 

I must mention that I didn't design this steering wheel from scratch, but modified it for my needs based on this [image here](https://commons.wikimedia.org/wiki/File:Formula_one_steering_wheel_back.svg). As per the Creative Commons share-alike licensing the image is offered under, you can find the [modified SVG here](https://github.com/olliephillips/iotUI.js/blob/master/svg_files/f1-wheel.svg).

The code for the component is fairly simple; it basically creates event emitters on all the buttons and, using HTML device orientation, a 'steer' event is emitted as the smartphone is rotated like a steering wheel.

If you're interested in how a Riot component is written you can inspect [the code for the tag here](https://github.com/olliephillips/iotUI.js/blob/master/tags/iotui-f1wheel.tag).

Adding the component to the page, is a simple as this:

```
<iotui-f1wheel 
  id="wheel1" wheellabel="iotUI.js" 
  primarybuttonlabel="TOOT" 
  startbuttonlabel="START/STOP" 
  secondarybuttonlabel="HONK" 
  width="540">
</iotui-f1wheel>
```
<br/><br/>
You can see that various attributes can be set to denote, wheel and button labels. 

I've embedded the actual steering wheel component below - open your web browser's javascript console if you like and give the buttons a go! You'll see all emitted events logged to the console. 

My MacBook Pro also seems to have a tilt sensor - if you're using a MacBook too, try tipping it forwards and away from you. You'll see some steer event changes, albeit not very useful ones.

<iotui-f1wheel 
  id="wheel1" wheellabel="iotUI.js" 
  primarybuttonlabel="TOOT" 
  startbuttonlabel="START/STOP" 
  secondarybuttonlabel="HONK" 
  width="500">
</iotui-f1wheel>

The entire HTML page, which also includes the websocket connection and event handlers to send control information back to the websocket server running on the NodeMCU connected to our RC car, looks like this:

```
<!DOCTYPE html>
<html lang="en"> 
<head><meta charset="utf-8"><meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0">
</head>
<body>
<iotui-f1wheel id="wheel1" wheellabel="iotUI.js" primarybuttonlabel="TOOT"  startbuttonlabel="START/STOP" secondarybuttonlabel="HONK" width="540"></iotui-f1wheel>	
<script>
var ws;
var evnt = {};
ws = new WebSocket("ws://" + location.host + "/espruinocar", "protocolOne");

function ignitionHandler(){
  evnt.type = "ignition";
  evnt.state = this.state.ignition;
  ws.send(evnt);
}

function driveHandler(){
  evnt.type="drive";
  evnt.state = this.state.drive;
  ws.send(evnt);
}

function primaryHandler(){
  evnt.type="primarybuttonpush";
  ws.send(evnt);
}

function secondaryHandler(){
  evnt.type="secondarybuttonpush";
  ws.send(evnt);
}

function steeringHandler(){
  evnt.type="steer";
  evnt.state=this.state.steer;
  ws.send(evnt);
}

window.onload = function(){	
  iotUI.addListener('wheel1', 'ignition', ignitionHandler);
  iotUI.addListener('wheel1', 'primarybuttonpush', primaryHandler);
  iotUI.addListener('wheel1', 'secondarybuttonpush', secondaryHandler);
  iotUI.addListener('wheel1', 'drive', driveHandler);
  iotUI.addListener('wheel1', 'steer', steeringHandler);
}
</script>
<script src="https://rawgit.com/olliephillips/iotUI.js/master/iotUI.js"></script>
</body>
</html>
```

## Websocket server

We're nearly there. 

To recap, we have an Espruino powered RC car; a control library that will send signals to ESP8266 NodeMCU pins; a web component built with Riot.js which will provide a control interface and; a HTML page in which we'll serve the component.

So to the final piece of the puzzle. We need a HTTP server to serve the HTML page and a websocket server, to listen for websocket messages from our web component, and relay them to our control library.

First the servers - actually it's just one server since the websockets module for Espruino, which upgrades the connection to a websocket, can also function as a HTTP server and serve our HTML:

```
var wifi = require("Wifi");
var ap = {
  "ssid": "wif_ssid",
  "pwd": "wifi_password"
};

// Our servePage handler which renders our controls
function servePage(req, res) {
  var page='<!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><meta name="viewport"';
  page+='content="width=device-width, user-scalable=no, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0"></head>';
  page+='<body><iotui-f1wheel id="wheel1" wheellabel="EspruinoCar" primarybuttonlabel="TOOT" startbuttonlabel="START/STOP"';
  page+='secondarybuttonlabel="HONK" width="540"></iotui-f1wheel>';
  page+='<script>var ws;var send={};ws=new WebSocket("ws://" +';
  page+='location.host + "/espruinocar", "protocolOne");function ignitionHandler()';
  page+='{send.type="ignition";send.state=this.state.ignition;ws.send(JSON.stringify(send));console.log(JSON.stringify(send));}function driveHandler()';
  page+='{send.type="drive";send.state=this.state.drive;ws.send(JSON.stringify(send));console.log(JSON.stringify(send));}function primaryHandler()';
  page+='{send.type="primarybuttonpush";send.state=null;ws.send(JSON.stringify(send));console.log(JSON.stringify(send));}function secondaryHandler()';
  page+='{send.type="secondarybuttonpush";send.state=null;ws.send(JSON.stringify(send));console.log(JSON.stringify(send));}function steeringHandler()';
  page+='{send.type="steer";send.state=this.state.steer;ws.send(JSON.stringify(send));console.log(JSON.stringify(send));}window.onload=function(){';
  page+='iotUI.addListener("wheel1", "ignition", ignitionHandler);iotUI.addListener("wheel1", "primarybuttonpush",';
  page+='primaryHandler);iotUI.addListener("wheel1", "secondarybuttonpush", secondaryHandler);iotUI.addListener("wheel1",';
  page+='"drive", driveHandler);iotUI.addListener("wheel1", "steer", steeringHandler);}</script>';
  page+='<script src="https://rawgit.com/olliephillips/iotUI.js/master/iotUI.js"></script></body></html>';
  res.writeHead(200, {'Content-Type': 'text/html'});
  res.end(page);
}

// Start everything
function initCar() {
  // Initialise pins
  ec.init();

  // Wifi connection
  wifi.connect(ap.ssid, {"password": ap.pwd}, function(err){
    if(!err){
      var server = require('ws').createServer(servePage);
      server.listen(8000);
      server.on("websocket", function(ws) {
        ws.on('message', function(evt){
          var ev = JSON.parse(evt);
          controlMap(ev);
        });
      });

      // Signal connected to wifi
      ec.horn.toot();
    }
  });
}

// Start up from boot
E.on("init", initCar);
```
<br/><br/>
Next, the ```controlMap(evt)``` function into which we pass the websocket event message, an object with two properties 'type' and 'state', and in which there is logic which maps the event to the control library (ec):

```
// Maps events received over websocket to ec object controls
function controlMap(event){
  var forwardSpeed = 70;
  var reverseSpeed = 40;
  console.log("message:" +  event.type +":" + event.state);
  switch (event.type){
    case "ignition":
      if (event.state == "off"){ec.stop();}
    break;
    case "steer":
      switch(event.state){
        case "left":
          ec.left();
        break;
        case "neutral":
          ec.ahead();
        break;
        case "right":
          ec.right();
        break;
      }
    break;
    case "drive":
      switch(event.state){
        case "stop":
          ec.stop();
        break;
        case "forward":
          ec.drive(forwardSpeed);
        break;
        case "reverse":
          ec.reverse(reverseSpeed);
        break;
      }
    break;
    case "primarybuttonpush":
      ec.horn.toot();
    break;
    case "secondarybuttonpush":
      ec.horn.honk();
    break;
  }
}
```
<br/><br/>
And that's it!

All that remains is to upload this code to our ESP8266 on the RC Car, and then save it using ```save()``` in the console on the left of the Espruino Web IDE. Now whenever the car starts the ```E.on("init", initCar)``` piece will run our ```initCar``` function and the car will connect to the Wifi network.

So we disconnect USB, and switch on the car. 

Wait for the horn toot, then connect to ```http://192.168.0.19:8000``` from our smartphone browser and off we go!

Actually there's a small gotcha on iPhone. Browsers on iPhone, whether Safari, Chrome or Firefox are all based on Webkit, which does not support the HTTP/1.0 standard, at least not for the purpose of upgrading the connection to websocket, but HTTP/1.0 is the standard used by Espruino.

You can fix this by taking a HEX editor to the Espruino binary and replacing all text instances of "HTTP/1.0" of which there are two, with "HTTP/1.1". This is enough to bamboozle the browser into thinking it's a HTTP/1.1 standard connection, and it seems to upgrade it to websocket with no apparent detrimental side effects.

Flash the customized binary to your ESP8266 and you are away. Thanks to Gordon Williams, the man behind Espruino itself, for that clever idea!

Don't forget, if some of the above code snippets are not that clear, [all the code is available here](https://github.com/olliephillips/espruinoCar).

Finally, my favorite picture if not the best quality one. My two youngest driving their Espruino powered remote control car. 

![alt text](/images/drive-it.png "First drive")
Pictured: First drive of our Espruino Wifi remote control car.

Have fun!








<script>
var evnt = {}
function output(evnt) {
  console.log("Event: '" + evnt.type + "' fired. State: '" + evnt.state + "'");
}

function ignitionHandler(){
  evnt.type = "ignition";
  evnt.state = this.state.ignition;
  output(evnt);
}

function driveHandler(){
  evnt.type="drive";
  evnt.state = this.state.drive;
  output(evnt);
}

function primaryHandler(){
  evnt.type="primarybuttonpush";
  output(evnt);
}

function secondaryHandler(){
  evnt.type="secondarybuttonpush";
  output(evnt);
}

function steeringHandler(){
  evnt.type="steer";
  evnt.state=this.state.steer;
  output(evnt);
}

window.onload = function(){	
  iotUI.addListener('wheel1', 'ignition', ignitionHandler);
  iotUI.addListener('wheel1', 'primarybuttonpush', primaryHandler);
  iotUI.addListener('wheel1', 'secondarybuttonpush', secondaryHandler);
  iotUI.addListener('wheel1', 'drive', driveHandler);
  iotUI.addListener('wheel1', 'steer', steeringHandler);
}
</script>
<script src="https://rawgit.com/olliephillips/iotUI.js/master/iotUI.js"></script>
