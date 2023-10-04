# Solivia G3 inverter Home Assistant integration using ESPHome

My house has a small solar array with a Solivia G3 inverter which I wanted to import the data from into Home Assistant. Luckily for me, there are already some resources available for this, but they need to be tweaked for your specific usecase. I figured I would note down what I know so far along with my ESPHome configuration so that anybody with enough knowledge to setup home assistant should be able to set this up too. 

## Sources
This project is based off [this repo](https://github.com/htvekov/solivia_esphome) by htvekov, along with [this repo](https://github.com/it-koncept/Solvia-Inverter-G3) by it-koncept. I will also make reference to the solivia communication protocol documentation which is available [here](https://forums.ni.com/ni/attachments/ni/170/1007166/1/Public%20RS485%20Protocol%201V2.pdf)

## Basic Setup
My setup consists of a single Solivia 3.3 AP G3 unit (variant 59). 

To see if my configuration would work for you, first go to section 5 (page 11) of the [communication protocol documentation](https://forums.ni.com/ni/attachments/ni/170/1007166/1/Public%20RS485%20Protocol%201V2.pdf) and find your inverter in the list. Then, go to section 8.6 (page 25) and see if your variant is listed in the title.

If your inverter is in that list, then you shouldn't need to modify much for a single inverter setup. If not, then the communication protocol will be different, this documentation may still help but will require more changes. Note that if you have a G4 inverter, htvekov's configuration may work for you.

Also note that it is possible to chain multiple units together so that up to 254 can be communicated with via the same serial interface. If you have multiple units connected like this, refer to it-koncept's config and the Advanced Setup section as my config only allows for a single inverter.

### Hardware Setup

Required Items:
- Sacrificial Ethernet cable (Needs all 8 conductors, so Cat 5e or better), cut to the required length.
- Female jumper wires (need at least 3 female ends so either 3 female-male wires, or 2 female-female wires). Can also get female crimps for solderless or solder directly to the ESP.
- ESP8266 or ESP32 setup with [ESPHome](https://esphome.io/index.html)

In my case, I'll be using an ESP8266 since the additional features of the ESP32 aren't necessary. 

Next, look at page 6 of the communication protocol documentation for the wiring. Depending on the type of ethernet cable you may have the wires in different orders, however you should be able to see the wire colors.

With serial, RX is for recieving data and TX is for transmitting data, thus the wiring should be as follows:

- RX from inverter -> TX on ESP
- TX from inverter -> RX on ESP
- GND from inverter -> GND on ESP

My config uses TXD0/GPIO1 & RXD0/GPIO3 on the ESP8266. There are multiple serial ports, however this is the hardware serial port which htvekov found more reliable than the software serial ports. If you wish to use a different port, or an ESP32, then you should adjust the config to use the appropriate GPIO pin number. A pinout of the ESP8266 is below for your info.

<img src="https://99tech.com.au/wp-content/uploads/esp-nodemcu-v1_pinouts_ll.jpg" width="500">

### Software Setup

Finally, place the solivia-G3.h file in the esphome config folder. On home assistant OS, this is probably located at `/root/config/esphome`. You can do this using SFTP and the "Advanced SSH & Web Terminal" on home assistant OS. Once you have adopted the ESP into home assistant, you can copy-paste the sections from the old config into the new config from this repo.

Make sure you also read through the config, as there are some important notes. Also keep in mind that my setup will send a command to the inverter every 10s, as I do not have the Solivia gateway which usually does this.

### Troubleshooting

If you are not recieving any data from the inverter, check the wiring of the serial cable and that it is connected to the inverter properly. As far as I'm aware, either port on the inverter will work. Also try swapping the TX and RX pins on the esp and if using hardware serial, ensure hardware debugging is disabled

If you are recieving a response from the inverter but the values look odd/wrong, try inverting the RX pin (I had to do this).

## Advanced Setup

### Multiple inverters

Each inverter is assigned it's own slave ID from 1 to 254, in order to add more inverters you must modify the solivia-g3.h file to add an additional set of sensors for each inverter, and to check the slave ID from the inverter response and assign the response to the appropriate values. The config must also be modified in order to add these additional sensor.
