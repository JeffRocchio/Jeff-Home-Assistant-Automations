# Purpose

Document how I plan to handle turning lights off for the 'Nighttime Shutdown' automation.

# Requirements

   1. Lights turn off in a manner that roughly simulates how I would do it manually. Which means room-by-room, and at a slow-ish pace as I make my way to my bedroom.
   2. IF I am not at home then the automation needs to also turn off the Master Bedroom. IF I am home then it needs to leave the master bedroom on as I will shutdown the master bedroom when I am ready.
   3. IF the upstairs is occupied with a guest - then the same things as the master bedroom. Assume guest will turn off when they are ready. Otherwise the automation needs to shut it off.
   4. The Foyer 'night light' needs to stay on.
   5. Need to have all the indicator LEDs on all the linked devices stay in sync. So this means I can't simply walk through 'entities,' I have to consider whatever scenes are applicable for that; and/or be sure I turn off the linked devices along with the load devices.
   6. Need to consider the Keypad buttons - another reason for not being able to just get a list of all switch/light entities and loop through them. I can't just simply turn off all the keypad buttons as part of this automation. E.g., the buttons that show the outside lights as being 'ON' needs to say on as I'm not shutting those lights off with this automation.
   7. Leave the Hallway niche spot on for 10 mins or so. This provides me some time to forget things and go back into the kitchen or living room to set the thermostat, or get my phone that I left by the recliner, etc.

# Current Approach

## Brute Strength

I haven't been able to work out in my head a good programmatic way to handle the 'Nighttime Shutdown' automation so that I get the results I want. So I'm doing it like I did it in the ISY. BUT with some differences.

Here is what I am doing:

   1. **Shutdown Automations**. I have created a separate automation that performs a shutdown for each individual area of the house. I have categorized those as 'Action - Room Shutdown.'  
   2. **Nighttime Shutdown Calls the Rooms**. Then in my master 'Action' automation - "`Nighttime Shutdown`" - I call each of those room shutdown routines in turn.
   3. **Pacing Delays**. For now I have inserted a standard delay of 3 seconds between each light turn off action. May want to randomize this in future.
   4. **Paced Shutdown Automation Calls**. To prevent all the rooms from running their shutdown automation routines at once I am using the '`Wait for a template to evaluate to true`' action after calling a given room's shutdown automation. NOTE that apparently I could also use the '`Wait for trigger`' action to accomplish this as well. I don't have any sense of if one or the other is a better choice.
   5. **Upstairs**. At this point I am simply shutting the upstairs off using the Insteon scene I created for the Upstairs All On/Off. That is embedded in a conditional block to test for Upstairs occupancy.
   6. **Jeff Is Away Conditional**. I am using a conditional to handle the 'Jeff is Away' or not situation. NOTE that in there I am also accounting for the time delay of both when I turn off the bedroom if I am away, to make it look like I am home; and how that timing relates to when I turn the Hallway niche spot off. The result is that the timing delay for the niche spot is fully contained within this conditional block. The goal being that the niche spot timing will appear to be about the same whether I am home or not.
	   1. So this looks like:
	  ``` yaml
		if:
		  - condition: state
			entity_id: input_boolean.jeff_away
			state: "on"
		# Jeff is Away - so shutdown the master bedroom.
		# But delay that, like I would if I were home.
		# And this delay also causes a delay before the
		# hallway niche spot gets turned off so don't
		# insert a extra delay action after this
		# conditioinal block.
		then:
		  - delay:
			  hours: 0
			  minutes: 12
			  seconds: 0
		  - action: automation.trigger
			metadata: {}
			data:
			  skip_condition: true
			target:
			  entity_id: automation.mstr_bedroom_shutdown
		  - wait_for_trigger:
			  - trigger: state
				entity_id:
				  - automation.mstr_bedroom_shutdown
				to: "off"
			timeout:
			  hours: 0
			  minutes: 5
			  seconds: 0
		# IF Jeff is home then the THEN block above didn't execute, so 
		# insert the 10 minutes delay here before turnng off the niche spot.
		else:
		  - delay:
			  hours: 0
			  minutes: 10
			  seconds: 3
	```

# Future Approach

## Not A Long List of Lights

I'd like to have an automation that works like I have Evening Dim automation. Use a loop to drive through all the lights in the house; and somehow exclude those I don't want to get turned off.

## Use Areas?

My initial thinking is to use the Areas I have assigned to each light. Walk through each area of the house in turn. This way it looks 'natural' from the outside. And I can then, e.g., exclude the Master Bedroom if 'Jeff Away' = False.

But also then layer onto this certain specific exclusions - like, e.g., if any given light is labeled as a 'night light' then it stays on (or gets turned on if it isn't already on).

See the HA docs, [Areas](https://www.home-assistant.io/docs/configuration/templating/#areas)
