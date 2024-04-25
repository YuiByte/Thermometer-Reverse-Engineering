# REVERSE ENGINEERING TEMPERATURE & HUMIDITY SENSOR 🌡️
###### by YuiByte, Roméo, Mathieu, Florent

#### Here is some Pictures of the Package of the [Aliexpress Thermometer](https://fr.aliexpress.com/item/1005006534648116.html), PCB, App..
img/bottomInfo.png
![[img/bottomInfo.png||350]]
![[BottomVertical.png||350]]

![[top.png||200]]
It's the top of the thermometer, it's a little bit cheap but however it's a 3$ thermometer.
![[bottom.png||200]]
Here is the bottom of the thermometer, you need to slide to open it.
![[topIN.png||350]]
And the top-in, the thermometer have two AAA 1,5v Batteries but not included in the packaging.
# [Smart Life](https://play.google.com/store/apps/details?id=com.tuya.smartlife&hl=fr&gl=US) APP screen :
![[Screenshot2.png||250]]
## Details of the app :
Here is the home screen of the application [Smart Life](https://play.google.com/store/apps/details?id=com.tuya.smartlife&hl=fr&gl=US) from our Thermometer. The app give Temperature informations and also Humidity. It's possible in the smart menu to do some automations with [Google Home](https://play.google.com/store/apps/details?id=com.google.android.apps.chromecast.app&hl=fr&gl=US), [Tuya Smart](https://play.google.com/store/apps/details?id=com.tuya.smart&hl=fr&gl=US) or other Home apps controls.   
We did not go any further on the automations with this app, they’re all have the same functionalities specified just before. With automations it's possible to activate connected radiator, air conditioner or air purifier to a certain temperature or Humidity.

![[Screenshot.png||250]]
Here is the same things that i said before but here is a built in automation to have notifications on smartphone and some other simple features.
# Detailed Table / Schematic : 
![[Capture d’écran 2024-04-12 à 11.07.59.png||340]]                    ![[card.png||260]]

# More details of Modules / Controllers :

#### *CB3S module :
- Built in with the low-power 32-bit CPU, which can also function as an application processor
- The clock rate: 120 MHz
- Working voltage: 3.0 to 3.6V
- Peripherals: 5 PWMs and 1 UART
- Wi-Fi connectivity
    - 802.11 b/g/n
    - Channels 1 to 14 @ 2.4 GHz
    - Support WEP, WPA/WPA2, WPA/WPA2 PSK (AES), WPA3 security modes
    - Support SmartConfig and AP network configuration manners for Android and iOS devices
    - Onboard PCB antenna with a gain of 1.3 dBi
    - Working temperature: -40℃ to 85℃
- Bluetooth connectivity
    - Support the Bluetooth LE V5.2
    - Support the transmit power of 6 dBm in the Bluetooth mode
    - Onboard PCB antenna with a gain of 1.3 dBi
#### AHT10 Temperature & Humidity sensor :
- Température de fonctionnement: -40 ° C à -85 ° C
- I2C Interface
- Working voltage : 1,8 - 6,0 V
##### Complementary informations :

| 11  | TXD2 | I/O | UART2_TXD (used to display the module internal information), which corresponds to P0 of the IC |
| --- | ---- | --- | ---------------------------------------------------------------------------------------------- |

The 11 pin of the CB3S module can be use to go further, for the moment we didn't search more for that but it will be very interesting to try something.

# Simple Schematics :
![[Capture d’écran 2024-04-11 à 19.16.50.png||400]]
# Oscilloscope Analysis:
![[oscilloscope.webp||700]]
To Analyse the TX port we use [Analog Discovery 3](https://digilent.com/shop/analog-discovery-3/) oscilloscope with [Wave Form](https://digilent.com/reference/)App .The numerical oscilloscope allowed us to find the baud rate witch is approximately 9600Hz witch can help us for future research.

![[image.png||800]]
We also use the [Wave Form](https://digilent.com/reference/) app to analyze the signal of the TX port on reboot. We find a thing that correspond to a json file, we also see an ID (we think that it correspond to the Thermometer ID on the App), and a version that correspond to the version of the [Smart Life](https://play.google.com/store/apps/details?id=com.tuya.smartlife&hl=fr&gl=US).
# Firmware Extraction :
After searching on the web we saw a forum with a guy who want to reverse and change the firmware too. He has the same product as us (location of components and date fabrications but it's the same). So this is the same PCB card with the same WIFI card. You can look at the forum [here](https://www.elektroda.com/rtvforum/topic3968377.html).

We know that the micro controller of the [CB3S wifi module](https://developer.tuya.com/en/docs/iot/cb3s?id=Kai94mec0s076) is the BK7231N, so we can copy the internal firmware connected by the RX / TX, 3,3v & GND using this [BK7231GUIFlashTool](https://github.com/openshwprojects/BK7231GUIFlashTool?tab=readme-ov-file).

[OpenBK](https://github.com/openshwprojects/BK7231GUIFlashTool?tab=readme-ov-file) is a open source flash tool and GPIO config extractor for beginners easy to use. (for GUI, BK7231T/BK7231N). Dedicated for Windows platform, but works on Linux with [Mono](https://www.mono-project.com/).
![[ReverseCardConnections.webp||500]]
## [UART Flasher App ](https://github.com/openshwprojects/BK7231GUIFlashTool?tab=readme-ov-file) :
![[reverseapp.png||550]]
The UART port is automatically detected by the app, you have to select the chip type (BK7231N or BK7231K) after you have to select the good baud rate (98200 / 115200 / 150000). Finally you can select "Do firmware backup (read) only", if you want to copy the internal firmware. Or you can select "Do Backup and flash new", to copy and flash a new firmware compatible with [Home assistant](https://www.home-assistant.io/).

After many tries we use the firmware of the forum guy to analyse because we had some connections problems with the card but what we saids before is working.
## Command to see strings on bin firmware :
After after copying the internal Thermometer firmware and decompress it, you can type this commands on linux. It allow you to see all the strings of your ```.bin``` file.

```bash
# -n allow to print strings that have [10] characters long
strings -n 10 readResult_BK7231N_QIO_2023-30-3--10-35-01.bin
```
### Searching something on the firmware :
After searching the interesting strings on the file, we find the wifi who are connected the Thermometer. Such as Name, hash of the password, IP of the host, subnet mask, ect..
#### Wifi informations Terminal Screen :
![[wifi.png||500]]
```bash
# Wifi host name
SKY185F8

# Hash of the MDP Host
5aa4a2fe7df883956d09c61ff3fbaa0ab602d69e5d846ef7e5724aa471bdc5bfPBCRPXQTVQ

# Host IP's
192.168.0.162
192.168.0.1
255.255.255.0

```
#### Language data Terminal Screen :
We find things that could be language UK / US but we don't really know what it correspond, maybe it defined the Language that chose the user when it configure the Thermometer on the [Smart Life](https://play.google.com/store/apps/details?id=com.tuya.smartlife&hl=fr&gl=US) APP or more simply it's just the internal language of the thermometer. Or it just could be wrong..

```bash
us) |u[)|uc) |uk) |us)|rK7&=Gr*ra9!
```
#### Firmware versions Terminal Screen :
And at least we also find logs that tell the version of the internal firmware witch is the 2.3.1.
```bash
mqtt | 2.3.1 | 01-01 00:00:22 | erc:0 | [9:-60 | 20:-2310]/mqtt | 2.3.1 | 01-01 00:00:60 | 10：-2307］/mqtt | 2.3.1 | 01-01 00:00:41 | erc:10 |[9：-60 | 10：-2307］/mqtt | 2.3.1 | 01-01 00:00:51 | erc:20 | [9:-61 |10:-2307]/
```
# End of this reverse engineering :
In this reverse project we discovered that is possible to reverse all of this 3$ Thermometer product. We find all the different component’s of this device. The thermometer use all the functionality’s that the seller tell witch is Temperature, Humidity and also automations. We didn't change the firmware to use it with Home assistant but if you want to do it please check this [forum](https://www.elektroda.com/rtvforum/topic3968377.html) and the [BK7231GUIFlashTool](https://github.com/openshwprojects/BK7231GUIFlashTool?tab=readme-ov-file). You can also check the 11 port of the [CB3S wifi module](https://developer.tuya.com/en/docs/iot/cb3s?id=Kai94mec0s076)that have a UART2_TXD on it, you can maybe find some interesting informations on it.
