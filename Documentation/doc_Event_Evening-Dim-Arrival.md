
**Feb-20-2025: This is obsolete**. For now I am only using the Daypart custom sensor to initiate the evening lights on. At some point I may use the Ambient Light Sensors for this. But for now it's too complex; thus too unreliable to use those as I am still trying to work out all the ins-and-outs and kinks of the light sensing.

# Note For Reference

In the Evening Lights On routine I need to 'reset' the Day-Dim state in case those routines got activated leading up to the Evening. But I should also test to be sure that neither of the two automations for that are currently running before I begin the Evening Lights On automation. IF the Day-Dim lights are in process of being turned either ON or OFF then should stop their in-flight automations. To accomplish that note the following:
### Automation - Stop In Flight

There is no direct way to tell a running automation to stop. But is you use a two-step process you can do it. Turn it Off (i.e. 'disable' it) then turn it back on (i.e., 'endable' it):

Action `automation.turn_off` then follow with:
Action `automation.turn_on` 

This action disables the automation’s triggers, and optionally stops any currently active actions.

| Data attribute | Optional | Description                                                                        |
| -------------- | -------- | ---------------------------------------------------------------------------------- |
| entity_id      | no       | Entity ID of automation to turn off. Can be a list. none or all are also accepted. |
| stop_actions   | yes      | Stop any currently active actions (defaults to true)                               |

Source: [Automation actions](https://www.home-assistant.io/docs/automation/services/)

# Event

The dim evening has arrived. This is determined by the state of the **sensor.ambient_light_outside** custom template sensor I have scripted.

# Response

   1. Turn on the 'Light Set - Evening Dim.'
   2. Run ISY program 'oAway_Act_staticON.'

# Sequence

   1. The state of the **sensor.ambient_light_outside** flips to "*Evening Dim*."
   2. **automation.evening_dim_arrived** fires and sets i**nput_boolean.static_lights_trigger_on_off** to *ON* state.
   3. **automation.turn_on_light_set_evening_dim** then fires and runs, turning on all lights that have the label '*Light Set - Evening Dim*' applied to them.
   4. **automation.turn_on_light_set_evening_dim** then runs the ISY program  *oAway_Act_staticON* to turn on any legacy ISY 'staticON' lights.
   5. --END--
