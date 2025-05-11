Where the Helper, `input_text.ambient_s1_off_areas`, contains the text: 'Kitchen,Dining'

NOTE the use of the rejectattr() and regex expression to exclude the non-load Insteon keypad buttons from this. (Keypad buttons meant to be in-sync with other lights need to have a seperate automation triggered to toggle the relevant keypad button.)

``` yaml
{% set selected_areas = states('input_text.ambient_s1_off_areas').split(',') %}
{{ selected_areas }}

{{ 
  expand(
    selected_areas
    | map('area_entities')
    | list
  )
  | selectattr('domain', 'in', ['switch', 'light']) 
  | selectattr('state', 'eq', 'off') 
  | rejectattr('entity_id', 'search', 'keypad_button_[b-h]')
  | map(attribute='entity_id')
  | list
}}
```

Result is:
```
['Kitchen', 'Dining']

[
  "light.diningrm_light_over_table_sw",
  "light.kitchen_keypad_button_a",
  "switch.kitchen_all_on_off_sw",
  "light.kitchen_lower_leds_sw"
]
```
