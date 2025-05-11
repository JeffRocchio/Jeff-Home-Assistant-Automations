# Action

## Nighttime Shutdown

An event has occurred that is interpreted to mean that EITHER the last awake person is now heading off to bed OR  that the away-from-home random lights on/off simulation is running and has triggered that it's the configured bedtime.

## Intention

Simulate a human making their way through the house, turning all the lights off.

### Rules, Conditions and Special Cases

#### Alexandra or Jordan is Home

   1. No change. Whoever occupies the house, the last person to head off to bed for the night is to trigger this event by pushing a button.

#### The Upstairs Guest Room is Occupied

   1. If this is true, do not shutdown the Upstairs area on the presumption that the occupant will do so when they are ready.

#### Jeff is Away from Home

   1. If nobody else is home when I am away, then the random lights simulation will be running, and the 'normal bedtime' time will be the trigger.
	   1. In this case also shutdown the Master Bedroom.
   2. Even if somebody else is home when I am away, IF I am away turn off the bedroom lights.

# Approach

Use two loops to walk through each Area, area-by-area, turning off any entity whose domain is either a '*light*' or a '*switch*.'

# Sequence

## Current Pre-ISY Migration:

   1. Event occurs:
	   1. Kitchen or Master-Bedroom keypad button '*G'* is pressed.
	   2. OR 'Normal Bedtime' has arrived while the Away from Home lights simulation is running.
   2. Shutoff HA Native Lights *except*:
	   1. Foyer LED vase
	   2. Mstr Bedroom Lights, not yet
   3. IF Upstairs is NOT Occupied: Shut off using insteon scene group-21.
   4. Call ISY program '*oHouse_Act_HeadToBed*'
	   1. Delay for 14 minutes to allow time for the ISY program to run and complete.
   5. WAIT for ISY program to complete.
	   1. Using the action: 'Wait for a template to evaluate to true':
	      `wait_template: "{{ is_state('binary_sensor.isy_head_to_bed_running', 'off') }}'"
	      `continue_on_timeout: true
	      `timeout: "00:14:00"
   6. THEN IF Jeff is NOT HOME, shut Master-Bedroom off <- Using 'Mstr Bedroom Shutdown' automation (as not all lights are Insteon).
   7. Turn Keypad buttons LED off. 
	   1. NOTE: The kitchen keypad button LED is turned off by the ISY program.
	   2. The HA automation thus ends by turning the Mstr Bedroom keypad button LED off.

## Future Approach:

   1. Event occurs:
	   1. Kitchen or Master-Bedroom keypad button '*G'* is pressed.
	   2. OR 'Normal Bedtime' has arrived while the Away from Home lights simulation is running.
   2. Shutoff all lights, going room-by-room, *except*:
	   1. Any whose label is 'Night Light'
	   2. IF Upstairs is Occupied, do not shut that area off
	   3. IF Jeff is home, do not shut Master-Bedroom off


# YAML Code

NOTE: This code may not be kept up to date with actual automation in HA.

As Of: 1-20-2025:
``` yaml
alias: Nighttime Shutdown
description: ""
triggers: []
conditions: []
actions:
  - action: light.turn_on
    metadata: {}
    data: {}
    target:
      entity_id:
        - light.kp_in_kitchen_g_bedtime
  - action: switch.turn_on
    metadata: {}
    data: {}
    target:
      entity_id: switch.mstrbed_keypad_button_g
  - delay:
      hours: 0
      minutes: 0
      seconds: 3
  - action: light.turn_off
    metadata: {}
    data: {}
    target:
      entity_id: light.sunrm_ceiling
  - delay:
      hours: 0
      minutes: 0
      seconds: 3
  - action: switch.turn_off
    metadata: {}
    data: {}
    target:
      entity_id: switch.living_room_couch
  - delay:
      hours: 0
      minutes: 0
      seconds: 3
  - if:
      - condition: state
        entity_id: input_boolean.bonus_occupied
        state: "off"
    then:
      - action: insteon.scene_off
        metadata: {}
        data:
          group: 21
  - delay:
      hours: 0
      minutes: 0
      seconds: 3
  - action: isy994.send_program_command
    metadata: {}
    data:
      name: oHouse_Act_HeadToBed
      command: run_then
  - wait_template: "{{ is_state('binary_sensor.isy_head_to_bed_running', 'off') }}'"
    continue_on_timeout: true
    timeout: "00:14:00"
  - if:
      - condition: state
        entity_id: input_boolean.jeff_away
        state: "on"
    then:
      - action: automation.trigger
        metadata: {}
        data:
          skip_condition: true
        target:
          entity_id: automation.mstr_bedroom_shutdown
  - action: switch.turn_off
    metadata: {}
    data: {}
    target:
      entity_id: switch.mstrbed_keypad_button_g
mode: single
```
