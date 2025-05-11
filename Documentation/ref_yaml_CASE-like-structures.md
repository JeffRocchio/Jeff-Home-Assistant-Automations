# Purpose

Provide a template for setting up CASE or c-switch like structures in Home Assistant yaml.

# See Also

ChatGPT Session *YAML CASE-Like Structure* @ [https://chatgpt.com/c/67fac93b-fdf0-8013-bcf6-4ff487356b2b](https://chatgpt.com/c/67fac93b-fdf0-8013-bcf6-4ff487356b2b)

# Template

``` yaml
- alias: CASE-Like Block with default section
  choose:

	- alias: CASE-1 
	  conditions:
        - condition: template
          value_template: >
            {{ ...some_condition... }}
      sequence:
        - service: ...
          data: ...

	- alias: CASE-2
      conditions:
        - condition: template
          value_template: >
            {{ ...another_condition... }}
      sequence:
        - service: ...
          data: ...

  default:
    - service: ...
      data: ...
```

# Example

One thing I have been struggling with is when quotes (single, double, or none) are needed and when they are not. So in inspecting the below pay careful attention to that.

``` yaml
# ELEMENT OF: LIGHT-SET VIRTUAL OBJECT
# ==== Change Log ==============================================================
# 2025-04-20: Completed all cases and performed simple validation tests
# 2025-04-16: Started to refactor per ChatGPT best practice for CASE-like structure
# 2025-04-12: Started on Development

lightset_assess_conditions:
  alias: Light Set | Assess Conditions
  description: >
    **Member Of**: *Light-Set / Day-Dim App.* <br>
    **Purpose**: For a given light-set, assess current conditions and take
    appropriate actions to transition it into an appropriate new state.
    **Requirements**: The calling automation must pass in the variables: <br>
    - 'lightset_label.' <br>
    **Documentation**: See sequence diagram *lightset Assess Conditions *

  # ==== Params in from calling automation or script ===========================
  fields:
    lightset_label:
      description: Label passed in by caller
      example: Light Set - Day Dim - Kitchen
      required: true

  # ==== Internal variables used in this script ================================
  variables:
    label_to_act_on: "{{ lightset_label }}"
    lightset_map:
      "Light Set - Lset App Test Light Set":
        state_helper: "input_select.lightset_lset_app_test_set_state"
        operational_state_helper: "input_select.light_set_lset_app_test_light_set_operational_state"
        delay_timer: "timer.lightset_lset_app_test_set_timer"
        ambient_sensor: "input_text.ambient_s9_test"
      "Light Set - Day Dim":
        state_helper: "input_select.light_set_amb_s2_state"
        operational_state_helper: "input_select.light_set_living_area_operational_state"
        delay_timer: "timer.amblight_s2_state_trans_timer"
        ambient_sensor: "input_text.ambient_s2_quantized"
    lset_state_helper: "{{ lightset_map[lightset_label].state_helper | default('NoStateHelper') }}"
    lset_delay_timer: "{{ lightset_map[lightset_label].delay_timer | default('NoTimer') }}"
    lset_ambient_sensor: "{{ lightset_map[lightset_label].ambient_sensor | default('NoSensor') }}"
    lset_current_state: "{{ states(lset_state_helper) | default('NoCurrState') }}"
    lset_operational_state_helper: "{{ lightset_map[lightset_label].operational_state_helper | default('NoOpsStateHelper') }}"
    ambient: "{{ states(lset_ambient_sensor) }}"
    daypart: "{{ states('sensor.daypart') }}" #<-- Note use of the single-quotes here, Won't work without them.
    timer_state: "{{ states(lset_delay_timer) }}" # <-- Yet here they are not needed because I am using a varable which holds the entity ID.

  sequence:
    # Write log entrys for debugging
    - alias: Debug - Show Variable Values in Trace
      service: logbook.log
      data:
        name: Lightset Debug
        entity_id: script.lightset_assess_conditions
        domain: script
        message: >
          FILED RECEIVED: [{{ lightset_label }}] ||
          VARIABLE [label_to_act_on: {{ label_to_act_on }}] ||
          VARIABLE [lset_state_helper: {{ lset_state_helper }}] ||
          VARIABLE [lset_operational_state_helper: {{ lset_operational_state_helper }}] ||
          VARIABLE [lset_delay_timer: {{ lset_delay_timer }}] ||
          VARIABLE [lset_ambient_sensor: {{ lset_ambient_sensor }}] ||
          VARIABLE [lset_current_state: {{ lset_current_state }}] ||
          VARIABLE [ambient: {{ ambient }}] ||
          VARIABLE [daypart: {{ daypart }}] ||
          VARIABLE [timer_state: {{ timer_state }}] ||
          VALUE [states(lset_delay_timer): {{ states(lset_delay_timer) }}] ||

    - alias: Debug - Debug CASE-2 Logic Test
      service: logbook.log
      data:
        name: Debug Conditional Test
        message: >
          [timer_state == 'idle': {{ timer_state == 'idle' }}] ||
          [states(sensor.daypart) == 'Day': {{ states('sensor.daypart') == 'Day' }}] ||
          [lset_current_state == 'Pending - OFF': {{ lset_current_state == 'Pending - OFF' }}] ||
          FULL TEST of value_template: >
                {{ timer_state == 'idle'
                and ambient == 'Bright'
                and lset_current_state == 'Pending - OFF' }}

      # Condition Tests/Filter
    - alias: Test conditions and take appropriate action
      choose:
        # ----CASE-1 ------
        - alias: 1. Time to reactivate the light-set for the next day
          conditions:
            - condition: template
              value_template: >
                {{ lset_current_state == 'Deactivated'
                and daypart == 'Night' }}
          sequence:
            # Set the ops state helper and another automation will take it from there
            - action: input_select.select_option
              metadata: {}
              data:
                option: "Activate"
              target:
                entity_id: "{{ lset_operational_state_helper }}"

        # ----CASE-2 ------
        - alias: 2. Light went 'not Bright' during the daytime and Disabled is False
          conditions:
            - condition: template
              value_template: >
                {{ ambient != 'Bright'
                and daypart == 'Day'
                and lset_current_state == 'Inactive' }}
          sequence:
            - action: input_select.select_option
              metadata: {}
              data:
                option: "In Process - ON"
              target:
                entity_id: "{{ lset_state_helper }}"

        # ----CASE-3 ------
        - alias: 3. Light went 'Bright' during the daytime while in 'Idle' state
          conditions:
            - condition: template
              value_template: >
                {{ ambient == 'Bright'
                and daypart == 'Day'
                and lset_current_state == 'Idle' }}
          sequence:
            - action: input_select.select_option
              metadata: {}
              data:
                option: "Pending - OFF"
              target:
                entity_id: "{{ lset_state_helper }}"

        # ----CASE-4 ------
        - alias: 4. 'Not Bright' happened while Transition Timer running
          conditions:
            - condition: template
              value_template: >
                {{ ambient != 'Bright'
                and lset_current_state == 'Pending - OFF' }}
          sequence:
            - action: script.lightset_timer_off
              metadata: {}
              data:
                lightset_label: "{{ label_to_act_on }}"
            - action: input_select.select_option
              metadata: {}
              data:
                option: "Idle"
              target:
                entity_id: "{{ lset_state_helper }}"

        # ----CASE-5 ------
        - alias: 5. Transition timer from 'went Bright' event has now expired, and it is still 'Bright'
          conditions:
            - condition: template
              value_template: >
                {{ timer_state == 'idle'
                and ambient == 'Bright'
                and lset_current_state == 'Pending - OFF' }}
          sequence:
            - action: input_select.select_option
              metadata: {}
              data:
                option: "In Process - OFF"
              target:
                entity_id: "{{ lset_state_helper }}"

mode: parallel
```
