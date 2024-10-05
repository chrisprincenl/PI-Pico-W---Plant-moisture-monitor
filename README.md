# PI-Pico-W---Plant-moisture-monitor
PI Pico W using MQTT and a capacitative Soil Moisture sensor to daily check on a plants need for watering 

Introduction

This project monitors the moisture levels of the soil a (house) plant using a PI Pico W, a Capacitive Soil Moisture Sensor and MQTT.

Important - this is not a completed project - all being developed currently

Background

This project exposes the following area's of learnings:

*) Hardware - Raspberry Pi Pico W which has WiFi to enable sending regular measurements. Use its build-in temperature measure sensor. And how to store data (measurements) as persistent data such that it is kept when the power is disconnected for a short while. The way to minimise power usage, an Adafruit TPL5110 Low Power Timer Breakout is used. Note: in case it is not battery operated, the timer breakout will not be required.
*) Capacitive soil Sensor to sense the level of moist in the plant pot
*) Thonny IDE (Interactive Development Environment) on my laptop to deploy the software onto the PI Pico
*) MicroPython language and libraries
*) MQTT messaging (PubSub - Publish the measurements onto a public - free - hub such that I can subscribe to it with my laptop, a phone app or a homekit device using homebridge)
*) JSON(Javascript Object Notation) - instead of defining own format, use existing JSON format for MQTT messaging.
*) Homebridge - Subscribe to MQTT Hub to read sensor data and view status on Apple HOME 

![Pi Pico W - wiring.png]


Functionality

The key functionality is:

*) Calibrate the sensor - a button is added to the sensor and also a multi colour LED. When the button is pressed, the calibration starts and the sensor then will take the maximum and minimum measures for the duration the button is pressed, and will use that as 0% and 100%. During this calibration, the sensor needs to be put into a grass of water and be outside of the soil fully dry. If the delta is large enough, the LED will be green, if the measurement doesn't show enough difference in value's the sensor isn't activated and the led will remain red.
Multiple moisture sensors (max 3) can be connected and once connected, they only get activated after a successful calibration.
*) Manual measure - a button is added to the Pi Pico enclosure to start the Pico and will do an additional ad-hoc measure. A multi color led is added to each sensor which flashes green when it starts and turns green when wifi is connected. If there is a problem with wifi or with the connection to the MQTT broker, it will turn red.
*) Resent measures and their dates&times. So keep measurements for a month. https://electrocredible.com/rpi-pico-save-data-permanently-flash-micropython/
*) Select alert levels per plant (this is the 'knowledge'. formula's and will require pot size, type of earth, type of plant, size of the plant, temperature, light levels etc)
*) Analyse: (1) Given too much water (can be determined from historical usage and type of plant) (2) Plant is in need of watering (3) Date/time water was given (when the moisture level goes up, it means it was watered).
*) versions - each message will include a software version and a hardware version. This is for future use in case the product evolves, and can make the device backwards compatible.
*) Use homebridge - access plat information on homekit
*) plantfood - a button and day counter when plantfood was provided

Messages

The messages that will be sent are:

All messages have (1) date (2) time (3) which source/sensor it comes from
Battery low
Plant moisture level (0 - dry 1 - Low 2 - Medium 3 - High) or percentage?
Message - date/time/temp/percentage moisture/Problem. Example: 20240118/13:20/+06/100/0 In this example, the measurement was taken on 18 Jan 2024 at 20 min past 1 in the afternoon. The temperature was +6 degrees Celsius and moisture level was 100 percent. The sensor concludes a 0, which means all ok. (4 is warning, 5 battery low, 8 is dry, 9 is too wet for too long).
Hardware & Wiring

Raspberry Pi Pico W

Raspberry-Pi-Pico-W-Pinout_1

Capacitive Soil Moisture Sensor

The soil moisture sensor (https://www.dfrobot.com/product-1385.html) measures soil mositure levels by capacitive sensing rather than resistive sensing like other sensors on the market. It is made of corrosion resistant material which gives it an excellent service life. Insert it in to the soil around the plant. This module includes an on-board voltage regulator which gives it an operating voltage range of 3.3 ~ 5.5V. It is perfect for low-voltage MCUs, both 3.3V and 5V. For compatibility with a Raspberry Pi it will need the PI Pico ADC converter.

To be experimented- when you create a sillicone cast, you can fill the top with epoxy to make the sensor waterproof

(https://www.dfrobot.com/product-1385.html) Capacitive Soil Moisture Sensor (https://www.dfrobot.com/product-1385.html) Specification Operating Voltage: 3.3 ~ 5.5 VDC Output Voltage: 0 ~ 3.0VDC Operating Current: 5mA Interface: PH2.0-3P Dimensions: 3.86 x 0.905 inches (L x W) Weight: 15g

image

Power breaker

Adafruit

Wiring

IMG_1072

Schottky diode battery connection

As mentioned in the Pico datasheet section 4.6 it is possible to connect both usb power and battery power, but you need to add an schottky diode inbetween battery and VSYS.

Tools

Thonny IDE
https://thonny.org
Libraries used:
datetime (micropython)
umqtt.simple
iPhone IOS App
IoT MQTT Panel
Setup and Install

1) Preparations

Install Thonny on your laptop and use a usb cable to connect to the Raspberry Pi Pico W. Make sure the cable is a data cable as there are many cables around that are only used for powering / charging battery packs which do not have the data wires in its usb cable. For this project the Raspberry PI Pico W is programmed with the MicroPython language

2) Wiring

See the picture in the wiring section.

Connect the sensor to the Pico
Connect the Pico to the laptop and start Thonny
Connect a two color led with resistors
Connect a button
(optional) Connect a three 1.5v battery pack with a Schottky diode
3) Coding

1) First test the sensor and calibrate

To read the sensor and to determine the percentage of the soil moisture, this example programme reads it and then calibrates.

See the MoistureSensor example programme

2) WiFi connection

The coding to enable the Pi Pico W Wifi and then sync the Real time clock sync over WiFi. Connect to a wifi router using WPS means that the Raspberry PI can be connected without the need for entering a password or hardcode in the software. The WPS function (WIFI Protected Setup) on a router is only active for around 2 minutes. Hence it will need to be activated simultaneously with booting the raspberry.
Info on the RTC in this link Youtube video on RTC vie WiFi

Not supported - See https://forums.raspberrypi.com/viewtopic.php?t=345301

To be researched is Wifi simple connect (DDP). It es exlained for ESP c3 board and works that a phone connects its wifi to the device and then exchanges credentials that are written to file for a next reboot.

2) MQTT Publishing messages using EMQX

Note that in this test EMQX is used, but there are also other free test brokers like HiveMQ

To send measurements, MQTT is chosen as it allows for PubSub (Publish and Subscribe) and basically makes it easier use for this kind of purpose. Before we can use MQTT, we first need to install the MQTT module for MicroPython. With this module, we can publish and subscribe to messages via a broker. Open Thonny and click on Tools >> Manage Packages. Then search for and install umqtt.simple (see MQTT MicroPython). Any MQTT broker can be used, but for this example HiveMQ is chosen as it has a free test service.

Install and start the IoT MQTT Panel app on the phone (or any other MQTT app on the laptop like MQTT Explorer) and configure to receive the JSON message. In the instructions of that app (link: instructions ) it explains the parsing of the JSON Data using simple JsonPath (more info in JSON Path).

See the MQTT_example programme, which publishes values to an MQTT broker.

3) Store data permanently on Pi Pico

After calibration the max and min values need to be stored so that they can be used again when the Pi Pico is powered of and on again. Also the Pico will keep measurements so that analysis can be done on the day (ie how much water does the plant use and how much did it get over a period. Also the temperatures can be kept and determine whether the plant is always in a warm or cold area. In this code example (called StoreData) the calibration is taken and then stored on the Pico. The code will first look whether there was already a calibration done, and then use that, and if there wasn't a calibration done, it will calibrate and store the min and max values.

See the StoreData_example programme, which writes two values (JSON format) to a file on the PI Pico.

4) Battery level measurement

In case the Pi Pico is powered from battery, the device needs to figure out whether the battery level is low such that it can still send out a message to warn that it needs battery replacement. In this code example it will first figure out whether the Pi Pico is powered by battery (VSYS or VBUS). Then if it is powered by battery (=VSYS), then determine whether the battery is low or not. Typically a low battery threshold is 2.8V). In the references section you'll find websites used as references to build this code example. In the PI Pico datasheet (see reference section) it mentions battery operation and recommended operating conditions (VSYS Min 1.8V, VSYS Max 5.5V and VBUS 5V ± 10%). As the VSYS can not be easily read in a PI Pico W (see explanation in trouble shooting and example code), the current code example only detects whether the device is powered by battery or not. Battery level is for a future improvement.

See the BatteryCheck_example programme, which will flash the internal Pico led and detect whether it is powered by battery.

5) Programming a button and external multi colour LED

To calibrate the moisture sensor it requires a button press and an LED to show the device is ready to start calibrating and when it has finished calibrating. In this program a button is connected to the GPIO port and a Green/Red LED is connected to two other GPIO ports. See the wiring section showing how the wiring is done.

Button press is done using an interrupt handler (also called an ISR - Interrupt Service Request). See further explanation in reference section. As interrupts can interrupt themselves and other problems can occur, it is important to follow strict rules for ISR as explained ISR rules

Note that:

LED connection - the LED will require for each LED a 220 Ohm resistor. This is to protect the LED as 3V is too high for an LED.

Button connection - In this wiring the button is connected straight to the Pico GPIO port without resistors. On the internet there are articles suggesting additional resistors and so on - and the main reason is purely to protect the Pi - and that really means to protect you from changing the pin from an input to an output, and then writing the output to logic 1. This will effecively short-circuit the output to ground and has the potential to destroy the pin, the SoC, etc. In input mode, the pin has a very high input impedance and is susceptible to stray voltages changing it from 0 to 1 to 0, to random ,etc. so it's normal to bias the pin one way or the other. e.g. connect a resistor from the pin to the 3.3v line and it will read 1 until you push the botton (connected from pin to 0v) when it will read 0. But even then, you do not need any resistors as the SoC has them built in.

See the ButtonPressLEDs_example programme

6) Store measurements

To enable plant/soil moisture levels and determining whether the plant needs watering (or has been given too much water), the measurements need to be stored. As the PI Pico W has persistent storage, but is limited, this example programme reads measurements and will store them in JSON format. When the PI is powered up, it will then first read what it has available. If there isn't any data, it will create the data. Also it will manage the amount of data to ensure that the device doesn't get out of storage.

See the StoreMeasurements_example programme

Plant information

This section explains the watering requirements for each plant and the logic for the sensor to determine whether it needs water or not (yet).

Philodendron monstera

During the summer months the Philodendron monstera requires watering when the soil starts to dry out. During the winter the the soil needs to be a week dry before the plant requires watering. Roughly in summer once in the 3 weeks, and in winter once in the 6 weeks. Info: Logic, determine whether summer or winter, then determine last time it was watered. if the soil is more than a week dry, it needs watering.

Trouble shooting

umqtt.simple install fails - Whilst installing in Thonny IDE the simple MQTT package it had an error. The reason was that the Thonny application on the laptop was an old version. Upgrading Thonny to the latest version fixed the problem.
Measuring battery level VSYS not working when network module is imported - As described in VSYS there is a problem using ADC(3) / GPIO29 to read VSYS on the Pico W and still have WiFi working. The resolution is to check VSYS before importing the network library. The pin definitions must be reset when the library loads. This works ok. ps Also see USBcharging Pico W
Internal LED does not switch on or off - The pins wiring to the Pi Pico W are different than the standard Pi Pico. Hence different code is required. Search for led=machine.Pin("LED", machine.Pin.OUT), led.on() and led.off().
MQTT JSON message could not be parsed on iPhone app IoT MQTT Panel - Two problems, first the message that was sent was a dictionary object and needed to be converted to a JSON string. The function for this is json.dumps(). The second problem was the parsing the JSON message in the app. Finally using in JsonPath for subscribe the following string: $.MoistPerc it worked and the app recognised the value.
Determining Battery level - UNRESOLVED - Currently not worked out how to code the battery level. Coding whether the PI Pico is powered by usb or battery is possible, but unclear how to then code the battery level. Any suggestions?
Pi pico error - a full reset to factory resolved the problem. https://electrocredible.com/how-to-reset-raspberry-pi-pico-w/
References

Sensor - https://how2electronics.com/capacitive-soil-moisture-sensor-with-raspberry-pi-pico/#
Sensor - other example - https://how2electronics.com/iot-soil-moisture-monitor-with-raspberry-pi-pico-w-blynk/
MQTT - https://www.tomshardware.com/how-to/send-and-receive-data-raspberry-pi-pico-w-mqtt
UMQTT simple - https://randomnerdtutorials.com/micropython-mqtt-esp32-esp8266/
Examples MQTT Connect - https://github.com/emqx/MQTT-Client-Examples/tree/master/mqtt-client-Micropython
Homebridge - https://www.instructables.com/Magic-MQTT-Button-for-HomeKit-Homebridge/#step1
Homebridge - https://github.com/arachnetech/homebridge-mqttthing/blob/master/docs/Configuration.md
video homebridge example - https://youtu.be/xlB1Js3Wmus?si=ouxNLGfudgIYg8w-
Save data permanently on PI Pico - https://electrocredible.com/rpi-pico-save-data-permanently-flash-micropython/
Raspberry Pi Pico W Temperature Data Logger - https://electrocredible.com/raspberry-pi-pico-w-data-logger-temperature-micropython/#google_vignette
Plants and watering them - https://www.123planten.nl/verzorging/water-geven
Battery check - https://github.com/TuriSc/RP2040-Battery-Check/tree/main
RaspBerry PI Pico data sheet - https://datasheets.raspberrypi.com/pico/pico-datasheet.pdf
Button and LED connections - https://projects.raspberrypi.org/en/projects/introduction-to-the-pico/8
ISR for button press - https://electrocredible.com/raspberry-pi-pico-external-interrupts-button-micropython/
Pumps - https://thepihut.com/products/peristaltic-liquid-pump-with-silicone-tubing-5v-to-6v-dc-power
RTC - https://github.com/LutzEmbeddedTec/Pico_w_RTC
URL Encode - https://www.w3schools.com/tags/ref_urlencode.asp



References:
1) https://how2electronics.com/capacitive-soil-moisture-sensor-with-raspberry-pi-pico/#
