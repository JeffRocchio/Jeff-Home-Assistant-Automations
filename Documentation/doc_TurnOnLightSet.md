# Purpose

Notes on automation that turns on all the evening 'static' lights.

# Method

My goal is to simply be able to apply the label "Light Set - Evening Dim" to any light that I want to come on when the "Ambient Light - Outside" custom sensor goes to it's 'Evening Dim' state.

So the automation I created simply:
   1. Creates a list of all 'entities' that have the label *Light Set - Evening Dim* on them.
   2. Loops over that list and for each item in the list turns on the light...,
   3. But also inserts a bit of delay between each loop pass so the lights come on kinda of slowly.
1

As of 2025-01-18:
``` yaml
alias: Turn On Light Set - Evening Dim
triggers:
  - trigger: state
    entity_id:
      - input_boolean.static_lights_trigger_on_off
    from: null
    to: "on"
actions:
  - action: light.turn_on
    metadata: {}
    data:
      brightness_pct: 100
    target:
      entity_id: light.living_room_tower_light
  - repeat:
      for_each: "{{ label_entities('Light Set - Evening Dim') }}"
      sequence:
        - data:
            entity_id: "{{ repeat.item }}"
          action: homeassistant.turn_on
        - delay: "00:00:05"
  - action: isy994.send_program_command
    metadata: {}
    data:
      name: oAway_Act_staticON
mode: single
```
