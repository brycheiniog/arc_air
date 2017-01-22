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
* Base units broadcasts continiously on multiple 

# Controller notes

* Controller has single Nordic nRF51822
  * JTAG interface is present on both 10 way and 4 way headers
  

