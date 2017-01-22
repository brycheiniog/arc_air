# arc_air
Reverse engineering the [Scalextric ARC Air](http://www.scalextric.com/uk-en/introducing-arc-air) wireless slot car controller

#Useful Documents

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


[controller_pcb]: https://github.com/brycheiniog/arc_air/blob/master/controller_pcb.jpg

