# Purpose

Document my key insights, observations, learnings as I go.

# Labels and Automation State

First, note that automations are also 'Entities.' (I think everything in HA is an 'Entity' - even if it is also something else).

The state of an automation is if that automation is 'Enabled' (ON) or 'Disabled' (OFF); it does NOT reflect if the automation is running or not. So be aware of this when assigning and using Areas and Labels with automations. For example, the following template will return a set of automations that have that label assigned as well as any lights and switches. And who knows what else:

``` yaml
	{{ label_entities('HA Native')  | select('is_state', 'on') | list }}

 RESULT
 [
  "light.wiz_dimmable_white_a56b6b",
  "automation.alex_night_lights_button_pressed",
  "automation.new_automation_2",
  "automation.bedtime",
  "automation.nighttime_shutdown",
  "automation.light_set_night_on",
  "automation.light_set_evening_dim_on",
  "switch.mstrbed_keypad_button_b",
  "automation.sync_phillips_bulb",
  "automation.sync_mstrbed_keypad_b",
  "automation.outside_front_off",
  "automation.outdoor_front_lights_off_time"
]
```

Since I am using labels to designate things like which lights to turn on the evening, which lights are Christmas lights, etc; it is important to NOT apply those labels to automations; or really to any other types of entities that I don't want to be controlling to On/Off states in my automations.
# Entities vs Devices

I *think* the object model is 'Device' is-composed-of 1:many 'Entities.' However, an Entity can also stand on it's own. I.e., an 'Entity' is contained within 0:1 'Devices.'

# Referencing Stuff in HA

## Entity ID

I've been told: "Unique ID is an internal structure (that is, don't use it within the UI or in YAML). *Entity_id is a first class citizen - consider it a primary key. It should be your first choice for addressing a ‘thing’."* [Entity-ID as Primary Key](https://community.home-assistant.io/t/truncation-of-v1-at-end-of-unique-id/827285/11?u=jeffrocchio)

## Domains - Light vs Switch vs Sensor vs ...

Very, very confusing - to find things, and by this I mean essentially 'entities' per the above note on Entity ID, you first have to know what '**domain**' that thing resides in. On a day to day basis the most confusion is generally around '*light*' vs '*switch*.' 

### Action First, then Device (entity)

It's kind of doubly confusing because when constructing automations you have to first choose what Action you want your intended device to take, and only after that are you offered choices on what device or 'entity' you wish to perform that action. So first you have to say 'I want to turn a light on,' then after you do that you are offered the opportunity to select a 'light.' But if your device was actually assigned into the 'switch' domain then you won't be able to find it.

Triply worse - if you then go look it up in the long list of Helpers, and copy it's '*entity_id*' and paste that into the automation step you are working on - well, it will accept it. All will look good. But the automation won't actually turn that light on because it's calling the *light.turn_on* service, which won't work for an entity that is really in the 'switch' domain. 

### Dimmable = Light, On/Off = Switch

Even worse! - When Insteon devices get added into HA some get added as 'lights' and some get added as 'switches.' I think what I've discerned is that devices that are dimmable are added to the 'light' domain and things that are not dimmable, are on/off only, get put into the 'switch' domain. As an example: the two 8-button Insteon keypads I have installed. For those, Button-A is in the 'light' domain. And indeed it will dim the load it is connected to. All the remaining buttons, tho, get put into the 'switch' domain. And indeed those just toggle on/off.

### Domains Don't Mix

In an automation (or script or template sensor) yaml block, once you have declared an action block for a domain, you can't then mix entities from different domains together under that action block.

So, e.g., the following will fail (yet, not necessarily with any visible error message to let you know this). The entity_id's that begin with 'switch' cannot be in that 'light' action block.
``` yaml
actions:
  - action: light.turn_on
    metadata: {}
    data:
      brightness_pct: 100
    target:
      entity_id: light.living_room_tower_light
      entity_id: switch.foyer_ledvase
      entity_id: switch.mstrbed_sw_outlet
      entity_id: light.mstrbed_dresser_spot_plug     
```

There is a cheat for this tho. As the 'service' use homeassistant.turn_on. This is some sort of higher level function that will turn the thing on whichever domain it belongs to. So the below will work:
``` yaml
actions:
  - action: homeassistant.turn_on
    metadata: {}
    data:
      brightness_pct: 100
    target:
      entity_id: light.living_room_tower_light
      entity_id: switch.foyer_ledvase
      entity_id: switch.mstrbed_sw_outlet
      entity_id: light.mstrbed_dresser_spot_plug     
```

Where this is essential is in cases like the below - where you can predict the domain:
``` yaml
actions:
  - action: input_boolean.turn_on
    metadata: {}
    data: {}
    target:
      entity_id: input_boolean.evening_lights_on_in_progress
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
```

This is all craziness from a usability and user experience perspective. But you just have to get use to being very, very, mindful attending to the 'domains.'

# Insteon Scenes

See the document: *Ref_Insteon-Scenes.md*.

# Automations

## mode: single

What is the line `mode: single` for? || ChatGPT - In Home Assistant automations, the mode defines how the automation should behave if a new trigger occurs while it’s already running. Here’s a quick overview of the modes:

**single** (default): If the automation is triggered again while it’s still running, the new trigger is ignored. It will only run one instance of itself at a time.

**restart**: If a new trigger arrives while the automation is running, the currently running sequence is stopped, and the automation restarts from the beginning with the new trigger.

**queued**: If the automation is triggered again while running, the new trigger is queued and executed after the current run finishes—one after another.

**parallel**: Multiple runs of the automation can happen in parallel. Each trigger starts its own sequence of actions immediately.

So, the line: `mode: single` simply tells Home Assistant, “If another trigger fires while this automation is already running, ignore that new trigger and let the current run finish without interruption.” For a lighting sequence that takes some time (like turning lights on one by one with a delay), single can prevent overlapping sequences.

You said: interesting. Does the 'ignore new trigger' apply only to triggers in the currently running automation, or does it apply across all automations; even for automations that wouldn't affect the running one?  ||  ChatGPT: The mode setting is local to each automation and only impacts how that specific automation handles multiple triggers at once. In other words, mode: single means if the same automation is triggered again while it’s already running, it will ignore the new trigger and continue its current run. However, it does not affect other automations or triggers unrelated to this particular automation.

