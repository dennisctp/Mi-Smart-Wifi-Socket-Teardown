# Mi-Smart-Wifi-Socket-Teardown
Ripping apart ZNCZ02CM to explore its 88MW300 Wifi Board

I have a few of these lying around collecting dust mainly due to security concerns.
Then I wonder whether I could scrap it's Wifi Board and repurpose it.

After ripping it apart and desoldering the board from the main plug, I've identified the chip being use is Marvel 88MW300.
To my suprise, this is quite a powerful chip. Following are some of it's spec from it's datasheet:

--------------------------------------------------------------------------------------------------------------------------------
https://fccid.io/YCJGTIMW302/User-Manual/User-manual-2777835.html
Processor 32-bit ARM Cortex-M4F running at 200 MHz
Memory    128KB ROM, 512KB RAM

PRODUCT OVERVIEW
The Marvell® HomeKit SDK is built on top of the industry-leading Marvell EZ-Connect™ Software SDK and greatly simpliﬁ es the development of HomeKit accessories. Marvell’s latest HAP SDK 2.0.r2+ is certiﬁ ed for Apple’s HomeKit Speciﬁ cation R9 and iCloud implementation. OEM customers using Marvell’s SDK save the cost and months of development and testing effort. 

Along with enabling several new behaviors such as humidity sensors, the Marvell HAP-SDK has ensured that security is at the core of its hardware, software and cloud connectivity. It starts with OTP, secure boot, encrypted key store onboard and extends to TLS through the use of mbedTLS and secure cloud communications.   

These features are enabled by the 88MW300 SoC, which provides a full array of peripheral inter-faces including SSP/SPI/I2S (3x), UART (3x), I2C (2x), General Purpose Timers and PWM, ADC, DAC, Analog Comparator, and GPIOs. It also includes a hardware cryptographic engine, RTC and Watchdog Timer. The 88MW302 includes a high speed USB On-The-Go (OTG) interface to enable USB audio, video and other applications. 

A complete set of digital and analog interfaces enable direct interfacing for I/O avoiding the need for external chips. The application CPU can be used to support custom application development avoiding the need for an external microcontroller or application processor.  (http://www.marvell.com/microcontrollers/88MW300/302/)


--------------------------------------------------------------------------------------------------------------------------------

After extracting the board, there are some extra pinout from the board with the following labels:
(From top to bottom)
1. Boot0
2. TestMode
3. Gate? (Not clear, might be wrong)
4. CS? (Not clear, could be SSN for QSPI Interface)
5. D0 (Judging from Datasheet, looks like QSPI Interface)
6. D1 (Judging from Datasheet, looks like QSPI Interface)
7. D2 (Judging from Datasheet, looks like QSPI Interface)
8. D3 (Judging from Datasheet, looks like QSPI Interface)
9. RX
10. TX 
11. TMS (JTAG Interface?)
12. RST (JTAG Interface?)
13. DO (JTAG Interface?)
14. CK (JTAG Interface?)
15. DI (JTAG Interface?)
16. 3V3 - can be use to power the board with GND
17. GND
18. BOOT1
19. Clk  (Judging from Datasheet, looks like QSPI Interface)

There's also 7 pin at the bottom.
From left to right
1. 3v +
2. 3v - (GND ?)
3. button press
4.
5.
6. 3v +
7. 3v - (GND ?)

Notes
1. Measuring between 1 & 2, 1 & 7, 6 & 7, 6 & 2 will have a 3.3v reading.
2. Jumping any of the 3 & 7 or 3 & 2 or 3 & GND will trigger the button pressed event.
3. Hooking 4 & 5 to RX TX doesn't give any reading.
4. Jumping boot0, boot1, testmode with any of the 3v / GND doesn't seems to trigger anything.
5. There's 2 interface with RX TX from datasheet, which is UART and SSP. not sure which one the RX TX belongs to.

--------------------------------------------------------------------------------------------------------------------------------

out of the obvious, I used a USB UART board and connect to the 3v3 pin, GND, RX and TX
I'm able to read dump of output, sample below is when we pressed the button long enough to reset it

_|      _|  _|_|_|  _|_|_|    _|_|  
_|_|  _|_|    _|      _|    _|    _|
_|  _|  _|    _|      _|    _|    _|
_|      _|    _|      _|    _|    _|
_|      _|  _|_|_|  _|_|_|    _|_|  
JENKINS BUILD NUMBER: 87
BUILD TIME: Oct 14 2015 14:45:35
BUILT BY: jenkins
firmware: 1.2.4
MIIO APP VER: 16
MIIO WIFI VER: SD878x-193.104.9.p243-702.1.0-WM
MIIO MCU VER: N/A
08:00:00.233 [D] xobj sys init complete
08:00:00.239 [D] first boot No did value found.
08:00:00.240 [D] enauth in psm is 1 
08:00:00.249 [D] rst cause is 8
[sdio] Card reset successful
[sdio] Card Version - (0x32)
[sdio] Card initialization successful
08:00:01.653 [D] factory mode magic is 0
08:00:01.653 [I] App thread started.
08:00:02.467 [I] ****psm_relay_status: 1
08:00:02.467 [I] ****psm_recover_disable: 0
08:00:02.470 [I] ****psm_led_off: 0
08:00:02.475 [I] worker_thread started.
08:00:02.477 [I] miio_app_thread while loop start.
[af] app_ctrl: prev_fw_version=0
[net] Initializing TCP/IP stack
digital_did is 87985400 
mac address is 7c49eb21666b
last four byte of key is 6e7a5259
[MIIO-NORMAL-BOOT]
08:00:02.507 [D] dump token value
16 bytes from 0x20001b00
        +0          +4          +8          +c            0   4   8   c   
+0000   5B 8C B8 A1 BD BD 9F 43 79 0C 5A 47 E8 BE 32 A5    [......Cy.ZG..2.
08:00:02.535 [I] otn:net bind(54321)
08:00:02.535 [I] [otn] registed.
08:00:02.538 [D] Starting mdns
08:00:02.541 [D] WAIT_SMT_TRIGGER
08:00:02.543 [D] smart cfg ssid is chuangmi-plug-m1_miap666b
08:00:02.549 [D] start smt config 
08:00:02.552 [D] private multicast addr is 182 
[af] app_ctrl [sta]: State Change: INIT_SYSTEM => UNCONFIGURED
[af] app_ctrl [prov]: State Change: PROV_INIT => PROV_STARTED

--------------------------------------------------------------------------------------------------------------------------------

As we can see, the MAC address, udid, and last 4 byte of the token are shown here.
After the long-pressed reset, the board will enter AP mode and broadcast it's ssid ending with last 2 byte of the MAC address.
It will keep changing the WiFi channel until there's a device connect to it.
