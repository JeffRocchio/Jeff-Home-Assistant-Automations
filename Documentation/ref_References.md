# Purpose

A variety of potentially useful references for Home Assistant.

# Raspberry Pico with Arduino

See [Arduino-Pico Documentation](https://arduino-pico.readthedocs.io/_/downloads/en/latest/pdf/)

Note that when I installed the Pico boards from 'Boards Manager' and then tried to use the Pico W which that one installed, I couldn't get it to work. Using the board 'core OS' and set of boards that came in with the package: https://github.com/earlephilhower/arduino-pico/releases/download/global/package_rp2040_index.json I was able to then select the "Raspberry Pi Pico W" board; and then the ports option become available. I uploaded the basic blink sketch example and it worked.

``` c++
// the setup function runs once when you press reset or power the board
void setup() {
// initialize digital pin LED_BUILTIN as an output.
pinMode(LED_BUILTIN, OUTPUT);
Serial.begin(115200);
}
// the loop function runs over and over again forever
void loop() {
digitalWrite(LED_BUILTIN, HIGH); // turn the LED on (HIGH is the voltage level)
delay(1000); // wait for a second
Serial.print("Hello world.");
digitalWrite(LED_BUILTIN, LOW); // turn the LED off by making the voltage LOW
delay(1000); // wait for a second
}
```

# Let's Encrypt

See instructions @ [Enable HTTPS using Let’s Encrypt in Home Assistant](https://theprivatesmarthome.com/how-to/enable-https-using-lets-encrypt-in-home-assistant/)

## Spook

Hi! I'm Spook 👻 and I'm a custom integration for use with Home Assistant. I will extend your Home Assistant instance with a huge set of scary powerful tools. 🛠️

Learn all about me in the extensive documentation

⚠️ Just to be very clear... Spook is not affiliated with, endorsed, recommended, or supported by the Home Assistant project. This custom integration is provided as-is, without any warranty.

[Spook on GitHub](https://github.com/frenck/spook)
## pyinsteon

pyinsteon - Python Insteon Package

This is a Python package to interface with an Insteon Modem. It has been tested to work with most USB or RS-232 serial based devices such as the 2413U, 2412S, 2448A7 and Hub models 2242 and 2245. Other models have not been tested but the underlying protocol has not changed much over time so it would not be surprising if it worked with a number of other models. If you find success with something, please let us know.

This pyinsteon package was created primarily to support an INSTEON platform for the Home Assistant automation platform but it is structured to be general-purpose and should be usable for other applications as well.

@ https://github.com/pyinsteon/pyinsteon

## HA Scenes vs Insteon Scenes

Important information to keep in mind concerning scenes : [Multiple questions about Insteon scenes](https://community.home-assistant.io/t/multiple-questions-about-insteon-scenes/224682)

   * BUT - it appears that HA has since been updated so that Insteon scenes **are now actually written into the insteon devices**.

## Linking Problems within HA

This reference might be of use; but hopefully I'll never need it!

**teharris1 commented on Nov 8, 2023**

@bytenik sorry for the long delay to respond. I finally looked at the log you sent back in August. The issue you are having is that many of the devices are rejecting the remote linking request with a direct NAK with reason code 0xFF which means **it will not accept the request to link with the new modem because the new modem is not in its All-Link Database**. I know this is a circular description but it is probably a security feature of some firmware versions. Here are the log records I am looking at:
 
**teharris1 commented on Jan 22, 2024** 
 
[@Doughboy68](https://github.com/Doughboy68) I have a pull request in that added the feature to remove a device from the Insteon integration in HA.

[@bytenik](https://github.com/bytenik) I assume you have moved back to your modem that was connected to your ISY. One option for you is to add a link in each device that looks like this:

```
Mode: Responder
Group: 0
Target: <new modem address>
Data1: 0
Data2: 0
Data3: 0
```

This will add your new modem as a controller of each of the devices. This way the new modem will already be able to send a command to the device to start remote all-linking.

Way down the page @ [Issue migrating Insteon from ISY to native HA integration #96259](https://github.com/home-assistant/core/issues/96259)

## Terminal on RPi

See section [Terminal & SSH Add-On](https://arachnoid.com/insteon_rescue/index.html)  @ https://arachnoid.com/insteon_rescue/index.html

## Worth a Read: YAML and Using Individual YAML Files

See the Section: Phase 6: Advanced Topics @ https://arachnoid.com/insteon_rescue/index.html


## Custom Dashboard

### Custom Card

To, e.g., create a card on a dashboard that can filter entries using the Label attribute. See "[ovelace-auto-entities](https://github.com/thomasloven/lovelace-auto-entities)

### Lovelace Horizon Card

This looks pretty cool. Would be great for a wall mounted tablet dashboard:

**Introduction**

The Horizon Card tracks the position of the Sun and the Moon over the horizon and shows the times of various Sun and Moon events, as well as their current azimuth and elevation, in a visually appealing and easy-to-read format.

[Lovelace Horizon Card](https://github.com/rejuvenate/lovelace-horizon-card)

## Using Custom Devices I Make

### MySensors 

Could I create a 'sensor' device using my ATtiny's and the nrf24s? Maybe. See:

> The MySensors project combines devices like Arduino, ESP8266, Raspberry Pi, NRF24L01+ and RFM69 to build affordable sensor networks. This integration will automatically add all available devices to Home Assistant, after presentation is done. That is, you do not need to add anything to your configuration for the devices for them to be added. Go to the states section of the developer tools to find the devices that have been identified.

See Following Sites:

   * [MySensors on GitHub](https://github.com/mysensors/MySensors)
   * [MySensors Webpage (seems out of date tho)](https://www.mysensors.org/)
   * [MySensors in HA Community Web](https://www.home-assistant.io/integrations/mysensors/)
   * [Home Assistant Integration Source Code on github](https://github.com/home-assistant/core/tree/dev/homeassistant/components/mysensors)

### ESPHome 

How about using my Raspberry Pi Pico Ws? Again, maybe. See the below, although I don't really understand what is going on there.


> ESPHome is a system to control your microcontrollers by simple yet powerful configuration files and control them remotely through Home Automation systems.

@ [ESPHome](https://esphome.io/)


### LEDs

To make my own LED indicators, or even lights.

I have 5 'neopixel' addressable through-hole LEDs I bought from Adafruit awhile back. These might be useful, and fun, for making some sort of status display that I could put on the credenza under the TV. Documentation for these are @ [NeoPixel Diffused 5mm Through-Hole LED - 5 Pack](https://www.adafruit.com/product/1938)



