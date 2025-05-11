
# Purpose

Notes on getting the Adafruit ESP32-S2 board, with light sensor add-on, working in Home Assistant / ESPHome.

# Key References

[Adafruit | ESP32-S2 Specs Web Page](https://learn.adafruit.com/adafruit-esp32-s2-feather)
[Adafruit | BH1750 Ambient Light Sensor Specs Page](https://learn.adafruit.com/adafruit-bh1750-ambient-light-sensor)
[Adafruit | Feather ESP32-S2 with BME280 will not work!](https://www.reddit.com/r/Esphome/comments/18ee880/adafruit_feather_esp32s2_with_bme280_will_not_work/)
[ESPHome | BH1750 Ambient Light Sensor](https://esphome.io/components/sensor/bh1750.html)
[ESPHome | Sensor Component](https://esphome.io/components/sensor/#config-sensor)
[Example | Feather ESP32-S2 with BME280 will not work!](https://www.reddit.com/r/Esphome/comments/18ee880/adafruit_feather_esp32s2_with_bme280_will_not_work/)

# Specs

[Adafruit ESP32-S2 Specs Web Page](https://learn.adafruit.com/adafruit-esp32-s2-feather)
[Adafruit BH1750 Ambient Light Sensor Specs Page](https://learn.adafruit.com/adafruit-bh1750-ambient-light-sensor)
## I2C

BME280 temperature / humidity / barometric pressures sensor connected over I2C on address 0x77 for immediate ambient weather sensing.

# Install and Setup

## First Try

   1. NOTE: I did NOT plug the board into the Raspberry Pi.
   2. In ESPHome Builder I clicked on '+ New Device.' In the pop-up clicked on 'Continue.'
   3. In the Create Configuration pop-up I gave it a name (Adafruit Feather Sensors) and clicked on Next.
   4. Then I selected the ESP32-S2 Board option, and Adafruit Feather option.
   5. Then...not fully sure the exact steps - but I purposefully made selections to 'manually' upload the firmware. I think there was a 'skip' option, which then caused the device to be created in ESPHome without actually plugging the board into the Raspberry Pi.
   6. From the ESPHome Builder's page, I clicked on 'Edit' on the new device card. In the YAML i only added the following, just to keep it very simple until I get it working:
      ```
         sensor:
			- platform: internal_temperature
			  name: "Internal Temperature"
		```

   1. From the device card's 3-dot menu I selected "Install" / "Manual download." It then built and complied the firmware. It then presents me with a pop-up, asking me which version of the compiled file I want to download. There are two choices, neither of which are the .elf file. The two choices are: 
	   1. Factory Format, which gives me the file: *ambient-light-sensor-s2.factory.bin*. 
	   2. OTA Format, which is to get the file: *ambient-light-sensor-s2.ota.bin*.
   2. So I cancel out of the pop-up, and I close the terminal screen, and then close the yaml code window. 
   3. Then, again from the 3-dot menu of the new card created, I selected "Download .elf file" and put the *adafruit-feather-sensors.elf* file onto my desktop linux machine.
   4. I ran elf2uf2 to produce the UF2 file for uploading to the board. `/home/jroc/Dropbox/projects/HomeAssistant/ESPHome/elf2uf2 -f 0xe48bff56 -p 256 -i adafruit-feather-sensors.elf`
   5. FAIL: I plug the board into my desktop PC. But I cannot get the PC to see it as a storage device. In fact, I can't get it to see it at all.
   6. FAIL-1: I plug the board into the Raspberry Pi and try to install on there using ESPHome Builder; but that fails as well: `ERROR Running command failed: Could not open /dev/ttyACM0, the port is busy or doesn't exist. ([Errno 2] could not open port /dev/ttyACM0: [Errno 2] No such file or directory: '/dev/ttyACM0')`
   7. BUT SUCCESS: I then just clicked on 'Retry' at the bottom of the install window and, for whatever reason, it worked. The firmware loaded and started running. The 'Internal Temperature' sensor appeared in HA and started reporting it's values.


# Second Install with 2nd Adafruit ESP32-S2

   1. Plug the board into the Raspberry Pi.
   2. In ESPHome Builder I clicked on '+ New Device.' In the pop-up clicked on 'Continue.'
   3. In the Create Configuration pop-up I gave it a name ('Ambient Light Sensor S2') and clicked on Next.
   4. Then I selected the ESP32-S2 Board option, and Adafruit Feather option.
   5. I select install. It does a lot of stuff; essentially compiling / linking all the modules. It kind of looks like success, but then gives an error about not being able to find an open port.
   6. I 'retry' the install several times - keep getting that message.
   7. So then I go ahead add in the basic internal temp sensor so that I have something I could see if it does work.:
      ```
         sensor:
			- platform: internal_temperature
			  name: "Internal Temperature"
		```
   8. From the device card's 3-dot menu I selected "Install" / "Manual download." It then built and complied the firmware. It then presents me with a pop-up, asking me which version of the compiled file I want to download. There are two choices, neither of which are the .elf file. The two choices are: 
	   1. Factory Format, which gives me the file: *ambient-light-sensor-s2.factory.bin*. 
	   2. OTA Format, which is to get the file: *ambient-light-sensor-s2.ota.bin*.
   9. So I cancel out of the pop-up, and I close the terminal screen, and then close the yaml code window. 
   10. Then, again from the 3-dot menu of the new card created, I selected "Download .elf file" and put the *adafruit-feather-sensors.elf* file onto my desktop linux machine, tho from the prior install of the first board I know I won't be able to use. Still I just want to have it at the ready.
   11. Then I run the 'Install' again. Below is what I get - which is confusing. Did it install the firmware or not? It's reporting out a mix of success and failure. Then I notice that the card for the device in ESPHome Builder is showing it as being 'online.' The I go to the Integrations / ESPHome and it does see the device and offers to add it. Which I do. And it does add it. And it is reporting out the internal temperature. I unplug it from the RPi and plug it into power on the other side of the room and it is still showing up and reporting internal temp. So...seems it **did** install the firmware. Below is what I got when I installed the 'manually' compiled yaml (by selecting 'manual download' option on the first run, then later initiating an 'install' from the device card's 3-dot menu). My conjecture at this point is that it did the install just fine. But then when it rebooted into the new firmware, at that point it doesn't have a USB port. Either the uploaded firmware removed that code from the Adafruit pre-loaded bootloader or ... or what??
```
INFO ESPHome 2024.12.4
INFO Reading configuration /config/esphome/ambient-light-sensor-s2.yaml...
INFO Generating C++ source...
INFO Compiling app...
Processing ambient-light-sensor-s2 (board: esp32-s2-saola-1; framework: arduino; platform: platformio/espressif32@5.4.0)
--------------------------------------------------------------------------------
HARDWARE: ESP32S2 240MHz, 320KB RAM, 4MB Flash
 - toolchain-riscv32-esp @ 8.4.0+2021r2-patch5 
 - toolchain-xtensa-esp32s2 @ 8.4.0+2021r2-patch5
Dependency Graph
|-- AsyncTCP-esphome @ 2.1.4
|-- WiFi @ 2.0.0
|-- FS @ 2.0.0
|-- Update @ 2.0.0
|-- ESPAsyncWebServer-esphome @ 3.2.2
|-- DNSServer @ 2.0.0
|-- ESPmDNS @ 2.0.0
|-- noise-c @ 0.1.6
RAM:   [=         ]  13.5% (used 44304 bytes from 327680 bytes)
Flash: [=====     ]  47.9% (used 878350 bytes from 1835008 bytes)
========================= [SUCCESS] Took 5.63 seconds =========================
INFO Successfully compiled program.
esptool.py v4.7.0
Serial port /dev/ttyACM0
Connecting...
Chip is ESP32-S2FNR2 (revision v0.0)
Features: WiFi, Embedded Flash 4MB, Embedded PSRAM 2MB, ADC and temperature sensor calibration in BLK2 of efuse V1
Crystal is 40MHz
MAC: 84:f7:03:ef:bf:80
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 460800
Changed.
Configuring flash size...
Auto-detected Flash size: 4MB
Flash will be erased from 0x00010000 to 0x000e6fff...
Flash will be erased from 0x00001000 to 0x00004fff...
Flash will be erased from 0x00008000 to 0x00008fff...
Flash will be erased from 0x0000e000 to 0x0000ffff...
Compressed 878736 bytes to 594767...
Wrote 878736 bytes (594767 compressed) at 0x00010000 in 7.4 seconds (effective 944.5 kbit/s)...
Hash of data verified.
Compressed 14624 bytes to 10156...
Wrote 14624 bytes (10156 compressed) at 0x00001000 in 0.2 seconds (effective 609.7 kbit/s)...
Hash of data verified.
Compressed 3072 bytes to 144...
Wrote 3072 bytes (144 compressed) at 0x00008000 in 0.0 seconds (effective 601.7 kbit/s)...
Hash of data verified.
Compressed 8192 bytes to 47...
Wrote 8192 bytes (47 compressed) at 0x0000e000 in 0.1 seconds (effective 755.0 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
INFO Successfully uploaded program.
ERROR The selected serial port does not exist. To resolve this issue, check that the device is connected to this computer with a USB cable and that the USB cable can be used for data and is not a power-only cable.
```

I then edited the yaml config to add in the config for the I2C bus and the BH1750 light sensor. I saved that, clicked 'Install,' and it all compiled and did an OTA upload to the device just fine. And it's all working now.

## Add Built-In BME Sensors

Edited the yaml to add the following:

``` yaml
i2c:
  sda: 3
  scl: 4
  scan: true # Enable I2C scanning
 
sensor:
  - platform: internal_temperature
    name: "Internal Temperature"

  - platform: bme280_i2c
    temperature:
      name: "Temperature"
      id: bme280_temperature
      oversampling: 1x
      accuracy_decimals: 2
    pressure:
      name: "Pressure"
      id: bme280_pressure
      oversampling: 1x
    humidity:
      name: "Humidity"
      id: bme280_humidity
      accuracy_decimals: 2
      oversampling: 16x
    address: 0x77
    update_interval: 120s

  - platform: wifi_signal
    name: "WiFi Signal Sensor"
    update_interval: 60s

  - platform: absolute_humidity
    name: "Absolute Humidity"
    temperature: bme280_temperature
    humidity: bme280_humidity
```
After successfully compiling, I again got the cannot upload error. I did the 'retry' thing; but it still failed, although it seemed to partially start in on the upload, with some success, before failing. Then I tried to update Over The Air (OTA); and that worked. BUT then it kept showing as unavailable in the HA dashboards. But it is showing as being online in the ESPHome Builder card.

In the router network map I can clearly see the Adafruit device, and it is online.

Here is the error I'm getting:
```
INFO Upload took 10.02 seconds, waiting for result...
INFO OTA successful
INFO Successfully uploaded program.
INFO Starting log output from 10.215.100.144 using esphome API
WARNING Can't connect to ESPHome API for adafruit-feather-sensors @ 10.215.100.144: Error connecting to [AddrInfo(family=<AddressFamily.AF_INET: 2>, type=<SocketKind.SOCK_STREAM: 1>, proto=6, sockaddr=IPv4Sockaddr(address='10.215.100.144', port=6053))]: [Errno 113] Connect call failed ('10.215.100.144', 6053) (SocketAPIError)
INFO Trying to connect to adafruit-feather-sensors @ 10.215.100.144 in the background
```

It's something to do with the I2C. If I fall back to creating sensors that don't use the I2C, and fully remove the I2C lines from the YAML then all is well. As in:
``` yaml
esphome:
  name: adafruit-feather-sensors
  friendly_name: Adafruit Feather Sensors

esp32:
  board: esp32-s2-saola-1
  framework:
    type: arduino

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "pqc5/rsMYf94OjjAaYk0zE/Jho6FMAQRiStt8XW4Yoo="

ota:
  - platform: esphome
    password: "5b0e232364824587fb7aa954fd0e84a8"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Adafruit-Feather-Sensors"
    password: "tpeeISn2obg1"

captive_portal:

sensor:
  - platform: internal_temperature
    name: "Internal Temperature"

  - platform: wifi_signal
    name: "WiFi Signal Sensor"
    update_interval: 60s
```
The above works just fine.

OK, so if I use the I2C related yaml bits from a reddit post, [post](https://www.reddit.com/r/Esphome/comments/18ee880/adafruit_feather_esp32s2_with_bme280_will_not_work/) then it works. So these bits:
``` yaml
esphome:
  name: adafruit-feather-sensors
  friendly_name: Adafruit Feather Sensors
  on_boot:
    priority: 801
    then:
      - switch.turn_on: I2C_POWER # Important, not working without

# <snip>

i2c:
  sda: 3
  scl: 4
  frequency: 800kHz
  id: bus_a
  setup_priority: 700

switch:
  - platform: gpio
    id: I2C_POWER
    restore_mode: RESTORE_DEFAULT_ON 
    pin:
      number: 7
  - platform: restart
    name: restart switch

sensor:
  - platform: internal_temperature
    name: "Internal Temperature"

  - platform: wifi_signal
    name: "WiFi Signal Sensor"
    update_interval: 60s
```

The Reddit poster's comment: "*The part that took a while to figure :*
``` yaml
switch: 
 - platform: gpio
    id: I2C_POWER
    restore_mode: RESTORE_DEFAULT_ON 
    pin:
      number: 7
```
*This was inverted in previous versions of the board, but the newest revision has it enabled on HIGH (hard to find, had to go to https://github.com/adafruit/Adafruit-Feather-ESP32-S2-PCB/blob/main/Adafruit%20Feather%20ESP32-S2%20Pinout.pdf to see mention of REV B and REV C…)*"

So then, adding all the on-board sensors:

``` yaml
esphome:
  name: adafruit-feather-sensors
  friendly_name: Adafruit Feather Sensors
  on_boot:
    priority: 801
    then:
      - switch.turn_on: I2C_POWER # Important, not working without

esp32:
  board: esp32-s2-saola-1
  framework:
    type: arduino

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "pqc5/rsMYf94OjjAaYk0zE/Jho6FMAQRiStt8XW4Yoo="

ota:
  - platform: esphome
    password: "5b0e232364824587fb7aa954fd0e84a8"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Adafruit-Feather-Sensors"
    password: "tpeeISn2obg1"

captive_portal:

i2c:
  sda: 3
  scl: 4
  frequency: 800kHz
  id: bus_a
  setup_priority: 700
  scan: true

switch:
  - platform: gpio
    id: I2C_POWER
    restore_mode: RESTORE_DEFAULT_ON 
    pin:
      number: 7
  - platform: restart
    name: restart switch

sensor:
  - platform: internal_temperature
    name: "Internal Temperature"

  - platform: uptime
    update_interval: 60s
    id: up

  - platform: bme280_i2c
    i2c_id: bus_a
    temperature:
      name: "Temperature"
      id: bme280_temperature
      oversampling: 1x
      accuracy_decimals: 2
    pressure:
      name: "Pressure"
      id: bme280_pressure
      oversampling: 1x
    humidity:
      name: "Humidity"
      id: bme280_humidity
      accuracy_decimals: 2
      oversampling: 16x
    address: 0x77
    update_interval: 120s

  - platform: wifi_signal
    name: "WiFi Signal Sensor"
    update_interval: 60s

  - platform: absolute_humidity
    name: "Absolute Humidity"
    temperature: bme280_temperature
    humidity: bme280_humidity
```

And this is working!

# Add the BH1750 Light Sensor

Referencing the web page: [BH1750 Ambient Light Sensor](https://esphome.io/components/sensor/bh1750.html)

First yaml code to try, from the above reference page:
``` yaml
# Example configuration entry
sensor:
  - platform: bh1750
    name: "BH1750 Illuminance"
    address: 0x23
    update_interval: 10s
```


## Median

Now I want to BH1750 to report out a median value, averaged out over several readings. Just to smooth the variations out a bit. Fortunally ESPHome includes a [filter](https://esphome.io/components/sensor/#config-sensor) to do this as part of it's yaml API:

A simple moving median over the last few values. This can be used to filter outliers from the received sensor data. A large window size will make the filter slow to react to input changes.

### Example configuration entry

``` yaml
- platform: wifi_signal
  # ...
  filters:
    - median:
        window_size: 7
        send_every: 4
        send_first_at: 3
```

Configuration variables:

- **window_size** (_Optional_, int): The number of values over which to calculate the median when pushing out a value. This number should be odd if you want an actual received value pushed out. Defaults to `5`.
    
- **send_every** (_Optional_, int): How often a sensor value should be pushed out. For example, in above configuration the median is calculated after every 4th received sensor value, over the last 7 received values. Defaults to `5`.
    
- **send_first_at** (_Optional_, int): By default, the very first raw value on boot is immediately published. With this parameter you can specify when the very first value is to be sent. Must be smaller than or equal to `send_every` Defaults to `1`.


# Deep Sleep

I tried this, and it does work, but while sleeping all the sensor data on the HA dashboard will show as unavailable. And also, when reprogramming you have to 'catch' it while it is awake.

``` yaml
deep_sleep:
  id: deep_sleep_control
  sleep_duration: 300s # = 5 minutes
  run_duration: 120s # = 2 minutes; seems to need this time to be able to come back up, connect, and start reporting out the sensor readings.
```
*The big caveat of the board: BME280 gets hot because of the ESP32 next to it. This makes Humidity reading be off as well…*

*If you don’t use deep_sleep of at least 5 min (per Adafruit’s recommendation) you would have reading as bad as 10°c higher than ambient temperatures, I tried playing with offset, and it was just not reliable enough (I have another sensor on an external board to calibrate). *