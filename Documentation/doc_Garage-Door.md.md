# Purpose

Keep notes on installing the insteon I/O Link device to sense and operate the garage door.

# Device ID

4D.88.B2

# Physical Install

See photos at /home/jroc/Dropbox/FileCabinet/HouseholdAssets/HomeAutomation/Insteon/GarageDoor

I had to bodge up a Chamberlain $10 wired wall button to be able to use the Insteon device's open/close relay feature to operate the door as the new garage doors use some propritary encryption technology such that the wires to the opener carry some sort of encrypted signal, it's not a simple Open/Close circuit. See the photos.

# Home Assistant Config

I added the device to HA, and that went just fine, no issues.

I don't recall if I needed to 'reverse' the sense signal so that HA understand the reed switch state of 'closed' as garage door closed. I did  mount the reed switch at the garage opening, with the magnet on the door such that the magnet activates the switch when the door is closed. I also think that I did change the delay and relay mode properties. In any case the property settings in HA that work correctly for me are as follows:

| Property        | Description                                                             | Modified | Value       |
| --------------- | ----------------------------------------------------------------------- | -------- | ----------- |
| sense_sends_off | Sense Sends Off                                                         | No       | false       |
| momentary_delay | Delay in seconds for the relay to turn on/off (depends on relay mode)   | No       | 2           |
| relay_mode      | Set the relay mode (see owners manual for a description of the options) | No       | Momentary B |

## Insteon Native Links

### In I/O Link Device

"Garage Door" friendly name in HA:

I have linked the sense switch to a button on the Kitchen Keypad as an indicator of the door being open. I am not using the keypad to control the door (given that the real switch in the garage is right outside the kitchen to garage door).

I did have to set the All-Link Database record in the keypad's ALDB 'respond' to the I/O link on Group-1, and with brightness level '0' (off for door closed). So the ALDB Records Are:

| Record ID | In Use | Modified | Target Address | Target Device Name              | Group | Mode       |
| --------- | ------ | -------- | -------------- | ------------------------------- | ----- | ---------- |
| 4095      | Yes    | No       | 43.70.06       | PowerLinc Serial Modem 43.70.06 | 0     | Responder  |
| 4087      | Yes    | No       | 43.70.06       | PowerLinc Serial Modem 43.70.06 | 1     | Controller |
| 4079      | Yes    | No       | 4E.B1.DF       | Kitchen \| Keypad \| 4E.B1.DF   | 0     | Controller |
### In Kitchen Keypad Device

"Kitchen | Keypad | 4E.B1.DF" friendly name in HA:

| Record ID | In Use | Modified | Target Address | Target Device Name | Group | Mode      | Data-1 | Data-2 | Data-3 |
| --------- | ------ | -------- | -------------- | ------------------ | ----- | --------- | ------ | ------ | ------ |
| 3975      | Yes    | No       | 4D.88.B2       | Garage Door        | 1     | Responder | 0      | 0      | 5      |
Data-1 = '0' for Turn Button Off when I/O Link sensor goes ON (mag switch closed)
Data-2 = On Ramp Rate. 0 = Instant
Data-3 = Button on Kitchen Keypad to Turn On/Off = Button #5

### In PLM Device

"PowerLinc Serial Modem 43.70.06" friendly name in HA:

(This link is for HA and is not related to the above link configuration for native Insteon link between the I/O link and Keypad devices.)

| Record ID | In Use | Modified | Target Address | Target Device Name | Group | Mode      | Data-1 | Data-2 | Data-3 |
| --------- | ------ | -------- | -------------- | ------------------ | ----- | --------- | ------ | ------ | ------ |
| 7479      | Yes    | No       | 4D.88.B2       | Garage Door        | 1     | Responder | 7      | 0      | 65     |
Data-1, 2, 3: I cannot explain these values. I have noticed in general that the data fields that HA sets for the PLM ALDB records feels spurious. I assume each value has some rationale, but I don't understand it and haven't come across any documentation about it.
