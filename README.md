# arc_air
Reverse engineering the [Scalextric ARC Air](http://www.scalextric.com/uk-en/introducing-arc-air) wireless slot car controller

It is a bit of a brain dump at the moment. The goal is to be able to replace the Scalextric controllers with my own and to be able to record and playback laps..

# Useful Documents

[FCC Documents for ARC Air controller](https://fccid.io/2ACUF-SSA00189)

[FCC Documents for ARC Air base](https://fccid.io/2ACUF-SSA00185)

[ARC Air users manual](http://www.pendleslotracing.co.uk/downloads/scalextric-arc-air-guide.pdf)

[Nordic nRF51822 Reference Manual](https://lancaster-university.github.io/microbit-docs/resources/datasheets/nRF51822.pdf)

# Base unit notes

* Base unit has two nrf51822 devices
  * One acts as the BLE interface to the tablet
  * The other acts as the propriatary wireless interface to the controllers
  * Not sure which drives the track power supply yet
  * JTAG interface to both devices appear accessible
* Base units broadcasts continiously on multiple bands.

# Controller notes

* Controller has single Nordic nRF51822
* Controller is not using BLE
* JTAG interface is present on both 10 way and 4 way pinouts. See photos below.
* Easy to add 4 pin header for jtag probe
![alt text][controller_pcb]


| Pin        | Function           |
| ------------- |:-------------:|
| 1  (Square pad)    | Ground |
| 2 | SWCLK      |
| 3 | SWDIO |
| 4 | VTref |

* It is possible to connect via JTAG and access code, registers and so on.
* Controller does not appear to transmit anything until it recieves something from the base unit.
   * Makes sense from a power saving POV.

# Investigations

* Connected to controller board via J-Link probe using Segger Embedded Studio
* Controller paired with track one on the base seems to use channel 5 (2405MHz) and occasionaly swaps to channel 41 (2441MHz)
* The radio registers from both my controllers were dumped and are essentially identical from a configuration perspective:

| Register | Value |
| ------------- |:-------------:|
| EVENTS_READY  |  	0x1 |
| EVENTS_ADDRESS  |  	0x1 |
| EVENTS_PAYLOAD  |  	0x1 |
| EVENTS_END  |  	0x1 |
| EVENTS_DISABLED  |  	0x1 |
| EVENTS_DEVMATCH  |  	0x0 |
| EVENTS_DEVMISS  |  	0x0 |
| EVENTS_RSSIEND  |  	0x1 |
| EVENTS_BCMATCH  |  	0x0 |
| SHORTS  |  	0x117 |
| INTENSET  |  	0x10 |
| INTENCLR  |  	0x10 |
| CRCSTATUS  |  	0x1 |
| RXMATCH  |  	0x0 |
| RXCRC  |  	0xb9 |
| DAI  |  	0x0 |
| PACKETPTR  |  	0x200028fe |
| FREQUENCY  |  	0x5 |
| TXPOWER  |  	0x4 |
| MODE  |  	0x2 |
| PCNF0  |  	0x00030006 |
| PCNF1  |  	0x01040020 |
| BASE0  |  	0x864e9676 |
| BASE1  |  	0x43434343 |
| PREFIX0  |  	0x23c343d2 |
| PREFIX1  |  	0x13e363a3 |
| TXADDRESS  |  	0x0 |
| RXADDRESSES  |  	0x3 |
| CRCCNF  |  	0x1 |
| CRCPOLY  |  	0x107 |
| CRCINIT  |  	0xff |
| TEST  |  	0x0 |
| TIFS  |  	0x0 |
| RSSISAMPLE  |  	0x38 |
| STATE  |  	0x9 |
| DATAWHITEIV  |  	0x40 |
| BCC  |  	0x0 |
| DAB[8]  |  	0x0 |
| DAP[8]  |  	0x0 |
| DACNF  |  	0x0 |
| OVERRIDE0  |  	0x0 |
| OVERRIDE1  |  	0x0 |
| OVERRIDE2  |  	0x0 |
| OVERRIDE3  |  	0x0 |
| OVERRIDE4  |  	0x0 |
| POWER  |  	0x1 |

* If you put the controller into pairing mode, it sits waiting for a message on channel 51 (2451MHz).
* I modified the Radio reciever example (nRF5_SDK_12.2.0_f012efa\examples\peripheral\radio\receiver) from the Nordic nrf58122 SDK with the above parameters and was able to recieve packets.
* I modified the reciever to scan through all 100 channels looking for packets. I got hits as per this table:

| Channel | Frequency | Payload | Notes |
| ------- | --------- | ------- | ------ |
| 5 | 2405 | 0x4 0x7 0x10 0x81 0x0 0x81 | Primary channel for Lane 1 |
| 11 | 2411 | 0x4 0x5 0x10 0x82 0x0 0x81 | Primary channel for Lane 2 |
| 17 | 2417 | 0x4 0x3 0x10 0x83 0x0 0x81 ||
| 23 | 2423 | 0x4 0x1 0x10 0x84 0x0 0x81 ||
| 29 | 2429 | 0x4 0x7 0x10 0x85 0x0 0x81 ||
| 35 | 2435 | 0x4 0x5 0x10 0x86 0x0 0x81 ||
| 41 | 2441 | 0x4 0x3 0x10 0x01 0x0 0x81 | Secondary channel for Lane 1 |
| 47 | 2447 | 0x4 0x1 0x10 0x02 0x0 0x81 | Secondary channel for Lane 2 |
| 53 | 2453 | 0x4 0x7 0x10 0x03 0x0 0x81 ||
| 59 | 2459 | 0x4 0x5 0x10 0x04 0x0 0x81 ||
| 65 | 2465 | 0x4 0x3 0x10 0x05 0x0 0x81 ||
| 71 | 2471 | 0x4 0x1 0x10 0x06 0x0 0x81 ||
| 79 | 2479 | 0x4 0x2 0x10 0x00 0x0 0x00 | This one doesn't seem to fit in with the pattern from the others. |
| 81 | 2481 | 0x4 0x1 0x10 0xc7 0x1 0x1 | Pairing packet on Lane 1 |
| 81 | 2481 | 0x4 0x1 0x10 0xc7 0x2 0x1 | Pairing packet on Lane 2 |

* Presumably the remaining slots are for the 4 additional controllers for the digital ARC Pro system.

* Packets transmitted by the base when pairing the lane 1 controller

| Pkt | Byte 0 |  Byte 1 | Byte 2 | Byte 3 | Byte 4 |  Byte 5 |
| ---- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| 1 | 0x4 | 0x1 | 0x10 | 0xc7 | 0x1 | 0x1 |
| 2 | 0x4 | 0x3 | 0x10 | 0xc7 | 0x1 | 0x1 |
| 3 | 0x4 | 0x5 | 0x10 | 0xc7 | 0x1 | 0x1 |
| 4 | 0x4 | 0x7 | 0x10 | 0xc7 | 0x1 | 0x1 |
| 5 | 0x4 | 0x1 | 0x10 | 0xc7 | 0x1 | 0x1 |
| 6 | 0x4 | 0x3 | 0x10 | 0xc7 | 0x1 | 0x1 |
| 7 | 0x4 | 0x5 | 0x10 | 0xc7 | 0x1 | 0x1 |
| 8 | 0x4 | 0x7 | 0x10 | 0xc7 | 0x1 | 0x1 |

* Packets transmitted by the base when pairing the lane 2 controller

| Pkt | Byte 0 |  Byte 1 | Byte 2 | Byte 3 | Byte 4 |  Byte 5 |
| ---- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| 1 | 0x4 | 0x1 | 0x10 | 0xc7 | 0x2 | 0x1 |
| 2 | 0x4 | 0x3 | 0x10 | 0xc7 | 0x2 | 0x1 |
| 3 | 0x4 | 0x5 | 0x10 | 0xc7 | 0x2 | 0x1 |
| 4 | 0x4 | 0x7 | 0x10 | 0xc7 | 0x2 | 0x1 |
| 5 | 0x4 | 0x1 | 0x10 | 0xc7 | 0x2 | 0x1 |
| 6 | 0x4 | 0x3 | 0x10 | 0xc7 | 0x2 | 0x1 |
| 7 | 0x4 | 0x5 | 0x10 | 0xc7 | 0x2 | 0x1 |
| 8 | 0x4 | 0x7 | 0x10 | 0xc7 | 0x2 | 0x1 |

* Clearly byte 4 is being used to indicate the Lane the controller is being paired with
* Presumably this goes into a look up table to determine which channels are mapped to which lane.

# Channel 79

* Channel 79 is odd.. It normally transmits in a cycle as follows:

| Pkt | Byte 0 |  Byte 1 | Byte 2 | Byte 3 | Byte 4 |  Byte 5 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | 0x4 | 0x2 | 0x10 | 0x0 | 0x0 | 0x0 |
| 2 | 0x4 | 0x4 | 0x10 | 0x0 | 0x0 | 0x0 |
| 3 | 0x4 | 0x6 | 0x10 | 0x0 | 0x0 | 0x0 |
| 4 | 0x4 | 0x0 | 0x10 | 0x0 | 0x0 | 0x0 |
| 5 | 0x4 | 0x2 | 0x10 | 0x0 | 0x0 | 0x0 |
| 6 | 0x4 | 0x4 | 0x10 | 0x0 | 0x0 | 0x0 |
| 7 | 0x4 | 0x6 | 0x10 | 0x0 | 0x0 | 0x0 |
| 8 | 0x4 | 0x0 | 0x10 | 0x0 | 0x0 | 0x0 |

* But occasionally another payload appears. These were taken from a sample of abot 100 packets on channel 79

| Pkt | Byte 0 |  Byte 1 | Byte 2 | Byte 3 | Byte 4 |  Byte 5 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | 0x4 | 0x5 | 0x10 | 0x06 | 0x00 | 0x81 |
| 2 | 0x4 | 0x7 | 0x37 | 0x7d | 0xff | 0xff |
| 3 | 0x4 | 0x1 | 0x10 | 0x06 | 0x00 | 0x81 |
| 4 | 0x4 | 0x1 | 0x10 | 0x1d | 0xd5 | 0xdf |

* No idea yet what the function of this channel is.
* It is interesting that during its normal cycle it uses even numbers in Byte 1 rather than odd used by all the other channels..







[controller_pcb]: https://github.com/brycheiniog/arc_air/blob/master/controller_pcb.jpg

