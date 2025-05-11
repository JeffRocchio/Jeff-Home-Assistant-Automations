
# Show All Attributes for Given Entity

In Developer Tools, Template:
``` yaml
{{ states.light.lvgrm_ceiling_sw }}

Result--
<template TemplateState(<state light.lvgrm_ceiling_sw=off; supported_color_modes=[<ColorMode.BRIGHTNESS: 'brightness'>], color_mode=None, brightness=None, insteon_address=56.83.DE, insteon_group=1, friendly_name=Living Room | Ceiling Switch, supported_features=0 @ 2025-01-28T20:37:03.135297-05:00>)>
```

To simply get the current state of an object:
``` yaml
{{ states('input_text.ambient_light_esp_lux_sensor_1') }}
# Result Is, e.g., : Dim
{% if states('input_text.ambient_light_esp_lux_sensor_1') == 'Dim' %}
          Morning Dim
{% endif %}
```

# Selecting Entities

## Entities in Given Area

Get list of all Areas that exist:
`{{ areas() }}`

Get list of all entities in a given area:
`{{ area_entities('Living Room') }}`

Get a list of all entities in a give area that are currently in an 'on' state:
`{{ area_entities('Living Room')  | select('is_state', 'on') | list }}`

Get a list of all entities in a give area that are currently in an 'on' state, and at a certain level of brightness:
`{{ area_entities('Living Room')  | select('is_state', 'on') | select('is_state_attr', 'brightness', 255) | list }}`

So paste the folloiwng into the Developer Tools | Template tool:
```
{{ areas() }}

{{ area_entities('Living Room') }}

{{ area_entities('Living Room')  | select('is_state', 'on') | list }}

# This does not work, don't know why:
{{ area_entities('Living Room') | selectattr("insteon_address", 'defined') }}
```

### Entities for a Set of Areas
``` yaml
 	{{ ['Sun Room', 'Living Room', 'Kitchen']
		| map('area_entities')
		| list }}
```


## Entities for a Given Label

### All Entities for a Label
``` yaml
	{{ label_entities('Light Set - Evening Dim') }}	
```

### Entities for Label That Are Currently ON

This returns a list of all entities that have had the Light Set - Evening Dim label applied to them and which are currently On:
``` yaml
	{{ label_entities('Light Set - Evening Dim')  | select('is_state', 'on') | list }}
```

### All Lights or Switches in a Set of Areas That Are On

``` yaml
{% set my_areas = [ 'Upstairs', 'Living Room' ] %} 
{{ expand( my_areas | map('area_entities') 
	| list )
	| selectattr('domain', 'in', ['switch', 'light']) 
	| selectattr('state', 'eq', 'on') 
	| map(attribute='entity_id') 
	| list }}

```

### Lights OR Switches That are Currently Off or ON

``` yaml
# Option One:
{{ expand(states.light, states.switch)
          | selectattr('state', 'eq', 'off')
          | rejectattr('entity_id', 'in', ['light.lvgrm_ceiling_sw', 'switch.outdoor_driveway_sw'])
		  | rejectattr('entity_id', 'search', '_\\d+$') # <-- Apparently you can also use regex tests
          | map(attribute='entity_id')
          | list }}

# Option Two - All lights/switches in a given area that are ON
{{ expand(area_entities('Upstairs'))
  | selectattr('domain', 'in', ['switch', 'light']) 
  | selectattr('state', 'eq', 'on') 
  | map(attribute='entity_id')
  | list }}

# Another approach --
# To Return Friendly Names:
{% set on_devices = states
| selectattr('domain', 'in', ['switch', 'light'])
| selectattr('state', '==', 'off')
| selectattr('last_changed', '<', now()) # <- Can use to filter on, say, 'turned off 5 or more minutes ago.'
| join('\n', attribute='name') %}
{{ on_devices }}

# To Return Entity IDs:
{% set on_devices = states
| selectattr('domain', 'in', ['switch', 'light'])
| selectattr('state', '==', 'off')
| selectattr('last_changed', '<', now())
| map(attribute='entity_id') 
| list %}
{{ on_devices }}

# All Insteon lights or switches:
{{ expand([states.light, states.switch])
  | selectattr('attributes.insteon_address', 'defined')
  | map(attribute='entity_id')
  | list }}
```

### Entities With a Given Attribute

``` yaml
{{ states.light 
  | selectattr('attributes.insteon_address', 'defined')
  | map(attribute='entity_id')
  | list }}
```

# Date and Time

## Setting Helpers Date / Time Values

For one source, see [# Input Datetime](https://www.home-assistant.io/integrations/input_datetime/)

```yaml
# Sets time to 05:30:00
- action: input_datetime.set_datetime
  target:
    entity_id: input_datetime.XXX
  data:
    time: "05:30:00"
# Sets time to time from datetime object
- action: input_datetime.set_datetime
  target:
    entity_id: input_datetime.XXX
  data:
    time: "{{ now().strftime('%H:%M:%S') }}"
# Sets date to 2020-08-24
- action: input_datetime.set_datetime
  target:
    entity_id: input_datetime.XXX
  data:
    date: "2020-08-24"
# Sets date to date from datetime object
- action: input_datetime.set_datetime
  target:
    entity_id: input_datetime.XXX
  data:
    date: "{{ now().strftime('%Y-%m-%d') }}"
```

## Set Helper Datetime Using Sun

This example did work for me:

----------
123 TarasRegular Jun '22

The following example sets an Input Datetime to the next sunset minus 1 hour. It assumes the Input Datetime is configured to store time _and_ date.

```yaml
- service: input_datetime.set_datetime
  target:
    entity_id: input_datetime.your_helper
  data:
    timestamp: "{{ (state_attr('sun.sun', 'next_setting')|as_datetime - timedelta(hours=1)).timestamp() }}"
```

If your Input Datetime is configured to store time only then the template will need modification. Let me know.

----------


# Randomness

JDv1 Jd | Apr '24

I hope I can ask you another question. I’m trying to set a number variable input_number.time_delta_offset with a random value at midnight. I tested {{ range(1,10) | random | int }} is generating a random value when I refresh but I can’t get it in the variable.

armedad | Apr '24

i’m not sure what you’re asking. how to set the value of that helper? if so use this format:

service: input_number.set_value
data:
  value: >
    {{ range(1,10) | random | int }}
target:
  entity_id: input_number.test_number

armedad JDv1 | Apr '24

oh, you wanted to do it at midnight… so something like this?

description: ""
mode: single
trigger:
  - platform: time
    at: "00:00:00"
condition: []
action:
  - service: input_number.set_value
    data:
      value: |
        {{ range(1,10) | random | int }}
    target:
      entity_id: input_number.test_number


JDv1 Jd | Apr '24

Wow. It works thanks. The syntax is difficult with no manual available. I had exactly that and it didn’t work. I didn’t use the | in my code at the line value. Is that that the reason why it didn’t work ?

armedad | Apr '24

did you have that verbatim except the | ?

the value: needs something to instruct it whether to take your input as literal input or to try to interpret it.

in this particular case you could do:

      value: "{{ range(1,10) | random | int }}"

that’s how you tell it to treat it as a template but do it all on 1 line.

or you can do what i did above or also:

      value: >
        {{ range(1,10) | random | int }}

this means to treat the multiple subsequent lines as a template to be interpreted. most people seem to do this only when they have a long multi-line template. but i prefer to do it pretty much for all templates. i don’t like the above " " method.

the difference between > and | is only if you have a multi-line output… so more important for text things like notification messages.

   message: >
      this is a random number:
      {{ range(1,10) | random | int }}

may yield something like:

this is a random number:  5

whereas using a | would yield something like:

this is a random number:  
5

**Source**: https://community.home-assistant.io/t/sun-next-rising-issue/712760/8
