
# Attribute Based
### Entities With a Given Attribute

``` yaml
{{ states.light 
  | selectattr('attributes.insteon_address', 'defined')
  | map(attribute='entity_id')
  | list }}
```

# Area Based

Get list of all Areas that exist:
`{{ areas() }}`

## Entities in Given Area

Get list of all entities in a given area:
`{{ area_entities('Living Room') }}`

Get a list of all entities in a give area that are currently in an 'on' state:
`{{ area_entities('Living Room')  | select('is_state', 'on') | list }}`

Get a list of all entities in a give area that are currently in an 'on' state, and at a certain level of brightness:
`{{ area_entities('Living Room')  | select('is_state', 'on') | select('is_state_attr', 'brightness', 255) | list }}`

### Entities for a Set of Areas
``` yaml
 	{{ ['Sun Room', 'Living Room', 'Kitchen']
		| map('area_entities')
		| list }}
```

### Entities In a Set of Areas That Are Currently Turned OFF
``` yaml
{% set my_areas = [ 'Upstairs', 'Living Room' ] %}
{{ 
  expand(
    my_areas
    | map('area_entities')
    | list
  )
  | selectattr('domain', 'in', ['switch', 'light']) 
  | selectattr('state', 'eq', 'off') 
  | map(attribute='entity_id')
  | list
}}
```

### Entities in Given Areas That Also Have a Given Label
``` yaml
{% set my_areas = [ 'Upstairs', 'Living Room' ] %}
{{ 
  expand(
    my_areas
    | map('area_entities')
    | list
  )
  | selectattr('entity_id', 'in', label_entities('Light Set - Day Dim'))
  | selectattr('domain', 'in', ['switch', 'light']) 
  | selectattr('state', 'eq', 'on') 
  | map(attribute='entity_id')
  | list
}}
```

### Only Insteon Entities In Areas That Are Currently Off
``` yaml
{% set my_areas = [ 'Upstairs', 'Living Room' ] %}
{{ 
  expand(
    my_areas
    | map('area_entities')
    | list
  )
  | selectattr('entity_id', 'in', integration_entities('insteon'))
  | selectattr('domain', 'in', ['switch', 'light']) 
  | selectattr('state', 'eq', 'off') 
  | map(attribute='entity_id')
  | list
}}
```

# Label Based

### All Entities for a Label
``` yaml
	{{ label_entities('Light Set - Evening Dim') }}	
```

### Entities for Label That Are Currently ON

This returns a list of all entities that have had the Light Set - Evening Dim label applied to them and which are currently On:
``` yaml
	{{ label_entities('Light Set - Evening Dim')  | select('is_state', 'on') | list }}
```

### All Lights or Switches In a Set of Labels That Are On
``` yaml
{% set my_labels = [ 'Light Set - Day Dim', 'Light Set - Day Dark' ] %}
{{ 
  expand(
    my_labels
    | map('label_entities')
    | list
  )
  | selectattr('domain', 'in', ['switch', 'light']) 
  | selectattr('state', 'eq', 'on') 
  | map(attribute='entity_id')
  | list
}}
```

# Based on Other Critera
### Lights OR Switches That are Currently Off or On
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

