Kitchen | Keypad | 4E.B1.DF

# Purpose
On 3/30/2025 I did a factory reset of this Insteon device as I was feeling that it was kind of messed up. So I am here keeping notes on 'recovery' of this device.

# Delete Device

While the All-Link Database did clear out, the Properties do not appear in Home Assistant to have changed at all. In particular they are still showing that mistaken button group; which physically on the reset device is now gone. When I do a reread from the device command it doesn't change anything. So I'm going to just delete the device and start over with it in HA.

# Naming
## Device Name

Kitchen | Keypad | 4E.B1.DF

## Button Names and Entity IDs

Kitchen | Keypad | A - Ceiling Lights [light.kitchen_keypad_button_a]
Kitchen | Keypad | B - Alex Nightlights [switch.kitchen_keypad_button_b]
Kitchen | Keypad | C - Living Room [switch.kitchen_keypad_button_c]
Kitchen | Keypad | D - Outside [switch.kitchen_keypad_button_d]
Kitchen | Keypad | E - Garage Door [switch.kitchen_keypad_button_e]
Kitchen | Keypad | F - Night Simulation [switch.kitchen_keypad_button_f]
Kitchen | Keypad | G - Bedtime [switch.kitchen_keypad_button_g]
Kitchen | Keypad | H - XMas [switch.kitchen_keypad_button_h]

## Labels

Kitchen | Keypad | A - Ceiling Lights: ==Light Set - Evening Dim==
Kitchen | Keypad | H - XMas: ==XMas Light==
