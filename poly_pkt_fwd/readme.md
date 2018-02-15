	 / _____)             _              | |    
	( (____  _____ ____ _| |_ _____  ____| |__  
	 \____ \| ___ |    (_   _) ___ |/ ___)  _ \ 
	 _____) ) ____| | | || |_| ____( (___| | | |
	(______/|_____)_|_|_| \__)_____)\____)_| |_|
	  (C)2013 Semtech-Cycleo
	  (C)2015 Modfications by Ruud Vlaming (The Things Network)

Lora Gateway packet forwarder with multiple extensions
======================================================

WARNING: 
  Bleeding edge work. Code not yet properly tested, will most likely 
  fail directly afer startup.  

1. Introduction
----------------

The poly packet forwarder is a program running on the host of a Lora 
Gateway that forward RF packets receive by the concentrator to a server 
through a IP/UDP link, and emits RF packets that are sent by the server. 
It is an 

It allows for the injection of virtual radio packets for testing and
is capable of communication with multiple servers. It is based on the
beacon_packet_forwarder with that distinction the operator has
detailed control over all capabilities. So beacon, gps, injection,
up/down stream, all can be (de)activated by modifying the json.

To learn more about the network protocol between the gateway and the server, 
please read the PROTOCOL.TXT document.

Specific Raspberry PI boards version features
=============================================

To be able to drive Raspbery PI concentrator on boards LED, this program
use ~~bcm2835 library, you need to install them before anything.
see http://www.airspayce.com/mikem/bcm2835/~~ gpiolib which control GPIO using 
linux filesystem, making this packet forwarder more compatible with different
target.

This version has been written to works with Linklabs board and also with
ic880a concentrator + Raspberry Pi Plate.
https://github.com/ch2i/iC880A-Raspberry-PI

Option for LED are in the configuration file so it should works with 
any board just adjusting GPIO settings. Settings are the following, 
a name and the GPIO pin number. Names are :

- `led_heartbeat` LED used for hearbeat, will always blink when running
- `led_down` LED used for downstream, will blink on each downstream from server 
- `led_error` LED used for hearbeat, will blink when a error occured
- `led_packet` LED will blink on each packet received
- `led_pps` LED used for GPS PPS indicator (mainly linklabs board)
- `pin_pps` Pin where PPS signal is connected to (if any, mainly linklabs board)
- `pin_reset` GPIO pin connected to SX1301 concentrator reset

GPS PPS pin on linklabs boards are not connected to LED but on a GPIO, so `led_pps` 
and `pin_pps` are used for a "software link", incoming PPS signal going to GPIO input
is redirected to GPIO led output, thus for example, with linklabs, configuration can be 


```json
  "pin_pps": 4,     /* GPIO4 PPS from GPS */
  "led_pps": 25,    /* GPIO25 PPS RED */
  "led_packet": 27, /* GPIO27 RED   */
```

for ic880a RPI plate (4 leds), configuration can be 
```json
  "led_heartbeat": 4, /* GPIO4 Blue   */
  "led_down": 18,     /* GPIO18 White */
  "led_error": 23,    /* GPIO23 Red   */
  "led_packet": 24,   /* GPIO24 Green */
  "pin_reset": 17,    /* GPIO17 concentrator reset */

```

And for RAK831 RPI Zero plate configuration can be (no led pins)
```json
  "pin_reset": 25,    /* GPIO25 concentrator reset */
```

  
An extented Log ouput on each packet received has also been added to this version, this
can be usefull for monitoring packet recevided by concentrator, regardless if they are or
not send the gateway. It's logged as info with the following informations

INFO: [#DeviceAddr] containing the device Addr (as seen on TTN dashboard) so you can 
filter with a grep for example
`tail -f /var/log/lora_pkt_fwd.log | grep "\[\#"`

then it's followed by :

- `jRQ` for join request 
- `jAC` for join accept 
- `uUP` for unconfirmed up
- `uDN` for unconfirmed down
- `cUP` for confirmed up 
- `cDN` for confirmed down 
- `RFU` for RFU

Other following data are classic information of frame received. Here below an example of log

```
root@pi01(ro):~# tail -f /var/log/lora_pkt_fwd.log
INFO: [up] PUSH_ACK for server log.gatewaystats.org received in 26 ms
INFO: [down] for server log.gatewaystats.org PULL_ACK received in 25 ms
INFO: [#25FAAE33] RFU CRC:Bad Freq:867.30MHz ch:4 RFch:0 LORA[SF7 125Khz 2/3] RSSI:-107dB SNR:-11.5dB Size:232b
INFO: [down] for server router.eu.thethings.network PULL_ACK received in 40 ms
INFO: [down] for server log.gatewaystats.org PULL_ACK received in 26 ms
INFO: [#1DCDB85F] cUP CRC:OK Freq:867.10MHz ch:3 RFch:0 LORA[SF12 125Khz 4/5] RSSI:-65dB SNR:+9.0dB Size:16b Data:'gF+4zR0ArAEBqkBtIvDn+A=='
INFO: [up] PUSH_ACK for server router.eu.thethings.network received in 40 ms
INFO: [up] PUSH_ACK for server log.gatewaystats.org received in 26 ms
INFO: [down] for server router.eu.thethings.network serv_addr[ic]PULL_RESP received :)
INFO: [down] a packet will be sent on timestamp value 2481851356
INFO: [#87802833] jAC CRC:Bad Freq:867.50MHz ch:5 RFch:0 LORA[SF7 125Khz 4/7] RSSI:-107dB SNR:-11.0dB Size:20b
INFO: [down] for server router.eu.thethings.network PULL_ACK received in 43 ms
INFO: [#DD5343A9] jAC CRC:Bad Freq:868.10MHz ch:0 RFch:1 LORA[SF7 125Khz 4/5] RSSI:-105dB SNR:-7.0dB Size:23b
INFO: [#37F76D0D] jRQ CRC:Bad Freq:867.10MHz ch:3 RFch:0 LORA[SF7 125Khz 4/5] RSSI:-102dB SNR:-7.0dB Size:23b
INFO: [#1D57298D] uUP CRC:OK Freq:867.30MHz ch:4 RFch:0 LORA[SF7 125Khz 4/5] RSSI:-95dB SNR:-3.0dB Size:23b Data:'QI0pVx2ADgABMTFQD+MBsz7x6VL877c='
INFO: [#1D57298D] uUP CRC:OK Freq:867.50MHz ch:5 RFch:0 LORA[SF7 125Khz 4/5] RSSI:-49dB SNR:+7.5dB Size:23b Data:'QI0pVx2ADgABMTFQD+MBsz7x6VL877c='
INFO: [up] PUSH_ACK for server router.eu.thethings.network received in 42 ms
INFO: [up] PUSH_ACK for server log.gatewaystats.org received in 27 ms

```


2. System schematic and definitions
------------------------------------
                                      
         ((( Y ))) 
             |                                   
             |     
         +- -|- - - - - - - - - - - - -+          xxxxxxxx             +---------+
         |+--+-----------+     +------+|       xxx        xxx          |         |-+
         ||              |     |      ||      xx  Internet  xx         |         | |-+
         || Concentrator |<----+ Host |<---- xx      or      xx ======>| Network | | |-+
         ||              | SPI |      ||      xx  Intranet  xx         | Servers | | | |
         |+--------------+     +------+|       xxx        xxx          |         | | | |
         |   ^                    ^    |        | xxxxxxxx |           |         | | | |
         |   | PPS  +-----+  NMEA |    |        |          |           |         | | | |
         |   +------| GPS |-------+    |  +--------+  +---------+      +---------+ | | |
         |          +-----+            |  | ghost  |  | monitor |        +---------+ | |
         |                             |  | server |  | server  |          +---------+ |
         |            Gateway          |  +--------+  +---------+            +---------+
         +- - - - - - - - - - - - - - -+

Concentrator: radio RX/TX board, based on Semtech multichannel modems (SX130x), 
transceivers (SX135x) and/or low-power stand-alone modems (SX127x).

Host: embedded computer on which the packet forwarder is run. Drives the 
concentrator through a SPI link.

Gateway: a device composed of at least one radio concentrator, a host, some 
network connection to the internet or a private network (Ethernet, 3G, Wifi, 
microwave link), and optionally a GPS receiver for synchronization. 

Network Servers: abstract computers that will process the RF packets received and 
forwarded by the gateway, and issue RF packets in response that the gateway 
will have to emit.

Ghost Server: Program on in/external computer the serves simulated radio
packets for testing purposes. Packets are send/received via UDP protocol.

Monitor Server: Program on in/external computer from which it is possible
to maintain the gateway by requesting system status updates or logging in
by an ssh tunnel through a firewall (on the gateways side)


3. Dependencies
----------------

This program uses the Parson library (http://kgabis.github.com/parson/) by
Krzysztof Gabis for JSON parsing.
Many thanks to him for that very practical and well written library.

This program is statically linked with the libloragw Lora concentrator library.
It was tested with v1.3.0 of the library but should work with any later 
version provided the API is v1 or a later backward-compatible API.
Data structures of the received packets are accessed by name (ie. not at a
binary level) so new functionalities can be added to the API without affecting
that program at all.

This program follows the v1.1 version of the gateway-to-server protocol.

The last dependency is the hardware concentrator (based on FPGA or SX130x 
chips) that must be matched with the proper version of the HAL.

4. Usage
---------

To stop the application, press Ctrl+C.
Unless it is manually stopped or encounter a critical error, the program will 
run forever.

There are no command line launch options.

The way the program takes configuration files into account is the following:
 * if there is a debug_conf.json parse it, others are ignored
 * if there is a global_conf.json parse it, look for the next file
 * if there is a local_conf.json parse it
If some parameters are defined in both global and local configuration files, 
the local definition overwrites the global definition. 

The global configuration file should be exactly the same throughout your 
network, contain all global parameters (parameters for "sensor" radio 
channels) and preferably default "safe" values for parameters that are 
specific for each gateway (eg. specify a default MAC address).

The local configuration file should contain parameters that are specific to 
each gateway (eg. MAC address, frequency for backhaul radio channels).

In each configuration file, the program looks for a JSON object named 
"SX1301_conf" that should contain the parameters for the Lora concentrator 
board (RF channels definition, modem parameters, etc) and another JSON object 
called "gateway_conf" that should contain the gateway parameters (gateway MAC 
address, IP address of the server, keep-alive time, etc).

To learn more about the JSON configuration format, read the provided JSON 
files and the libloragw API documentation.

Every X seconds (parameter settable in the configuration files) the program 
display statistics on the RF packets received and sent, and the network 
datagrams received and sent.
The program also send some statistics to the server in JSON format.

5. License
-----------

Copyright (C) 2013, SEMTECH S.A.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright
  notice, this list of conditions and the following disclaimer.
* Redistributions in binary form must reproduce the above copyright
  notice, this list of conditions and the following disclaimer in the
  documentation and/or other materials provided with the distribution.
* Neither the name of the Semtech corporation nor the
  names of its contributors may be used to endorse or promote products
  derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL SEMTECH S.A. BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

6. License for Parson library
------------------------------

Parson ( http://kgabis.github.com/parson/ )
Copyright (C) 2012 Krzysztof Gabis

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

*EOF*
