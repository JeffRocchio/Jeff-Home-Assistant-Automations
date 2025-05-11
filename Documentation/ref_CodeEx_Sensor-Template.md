This is an interesting example of a custom template sensor structure:

``` yaml
	template:
	  - trigger:
	    - trigger: state
	      entity_id:
	        - sensor.example_1
	        - sensor.example_2
	    # debugging functionality
	    action:
	      - if:
	          - condition: template
	            value_template: |
	              {{ 'TEMPHADEBUG' in states('input_text.amn_template_debug_target') }}
	        then:
	          - action: notify.send_message
	            target:
	              entity_id: notify.debug_log
	            data:
	              message: "{{ trigger }}"
	    sensor:
	      - name: "Example Sensor"
	        state: >
	          {% set temp = [states('sensor.example_1'), states('sensor.example_2')]
	          | select('ne', 'unknown') | first | round(0, default=-99) %}
	          {{ temp }}
```