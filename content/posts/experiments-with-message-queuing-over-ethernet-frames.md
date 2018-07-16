+++
title = "Experiments with Messaging over Ethernet Frames"
description = "An alternative messaging protocol which could be ideal for your hardware projects"
date = "2017-07-10T17:27:33+01:00"

tags = [
    "iot",
    "mqtt",
	"efmq",
    "golang",
	"raspberrypi",
	"lan",
	"ethernet"
]

categories = [
    "iot",
	"programming"
]
+++
## In his article, [Network Protocol Breakdown: Ethernet and Go](https://medium.com/@mdlayher/network-protocol-breakdown-ethernet-and-go-de985d726cc1), Matt Layher describes the Ethernet protocol and introduces a couple of libraries written in Go.

I read the article with interest. Application communication, at the Ethernet frame level, a lower level than TCP sockets, was something I'd never considered before. 

Of particular interest was the "broadcast" nature of the communication. In contrast to sockets - though frames can also be addressed to specific devices - frames can be broadcast network-wide, enabling more than one device to listen and use the frame payload.

I could see some parallels between Ethernet frames and MQTT, a protocol I use frequently in my hardware projects. 

I could also see a few distinct advantages over MQTT. I don't want to go into loads of detail here, but, suffice to say, a cheap, convenient but very insecure approach to using MQTT in what are often just "hobby" hardware projects, is to bounce traffic off of one of the free remote MQTT brokers. And I have been known to do this ;)

##  So why Ethernet Frames?

Potentially, by using Ethernet frames there would be no requirement for the MQTT broker so that's one less piece of hardware to configure and manage (if you host your own); messages would stay within the local area network, behind the router firewall, so we're secure by default and; we might achieve higher-speed transmissions, since we would not be sending data all the way off to a remote MQTT broker, only to receive it back from that broker on another machine sitting within the same local area network.

So I set about writing a package on top of Matt's [Ethernet](https://github.com/mdlayher/ethernet) and [Raw](https://github.com/mdlayher/raw) packages with two objectives in mind:-

- Use Ethernet frame messaging to emulate MQTT specifically by providing pub/sub functionality, and; 
- Provide an API which would feel familiar to MQTT users and therefore be a relatively simple drop in replacement for a typical MQTT client library.

## Methodology
The package itself is fairly straightforward - most of the sophisticated stuff is performed by the two packages of Matt's which it imports, but it satisfies both of the objectives mentioned above. 

I took the liberty of naming the package EFMQ (for Ethernet Frames Message Queue), and you can find it [here](http://github.com/olliephillips/efmq).

### Objective 1 - Provide Pub/Sub functionality

Pub/Sub is emulated by devices maintaining a list of their own subscriptions and comparing each received message's topic to this list. If topic matches a subscription, the message is put on an unbuffered channel for subsequent processing. Any messages that do not match a subscription are simply discarded.

### Objective 2 - Provide an API similar to MQTT

This is best illustrated by a couple of examples.

#### Publisher

In this contrived example, we're publishing temperature data on the `temp` topic every second. `wlan0` represents the network interface used by the device, in this case a Raspberry Pi Zero W.

```
mq, err := efmq.NewEFMQ("wlan0") 
if err != nil {
  log.Fatal(err)
}
t := time.NewTicker(1 * time.Second)
for range t.C {
  if err := mq.Publish("temp", "20.5"); err != nil {
 	log.Fatalln(err)
  }
}
```
<br/>
#### Subscriber

In this example another device in the same network subscribes to the `temp` topic and then listens indefinitely. Any messages which match the subscription are made available on the `mq.Message` channel.

```
mq, err := efmq.NewEFMQ("wlan0")
if err != nil {
  log.Fatal(err)
}
mq.Subscribe("fermenter")
mq.Listen()
for msg := range mq.Message {
  fmt.Println("topic:", msg.Topic)
  fmt.Println("message:", msg.Payload)
}
```
<br/>
### Performance

We're off to a good start by removing the latency transmitting messages via a remote server, and the messaging is direct between two devices, no third device (a local MQTT broker) is needed to relay messages, but as yet I don't have any benchmarks.

In testing, I've been running a dummy setup using two Raspberry Pi Zero W devices which have been communicating using EFMQ at an interval of 50ms for over a week with no issues. 50ms means 20 messages per second - more than sufficient for most monitoring and control applications that I'm likely to want to build!

If you give EFMQ a try please let me know how it works for you!