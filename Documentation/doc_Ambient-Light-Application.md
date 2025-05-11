# Purpose

Keep notes and documentation on how I am computing and handling the sensing of ambient light conditions and automations driven by ambient light sensors.

I am thinking of this as an "**Application**" vs just an automation. This is because it consists of multiple automations, helpers, and sensors. Collectively they work together to form an bigger thing that feels like an Application.

See also the section on Ambient Light in */Documentation/doc_diagrams.odg*.

### April 23, 2025

I have created two light sensors that are available in Home Assistant; via the ESPHome integration. These are both built from the Adafruit ESP32-S2 feather, with the Adafruit BH1750 Luminescence sensor attached via the I2C bus of the feather board. 

#### Sensors

At the moment I have two active sensors:

*Sensor-1*: This is mounted in the Kitchen, above the cabinets on the back wall 'looking' out the dining room windows. it controls only the lower cabinet LED lights.

*Sensor-2*: This is upstairs, mounted on the back wall 'looking' out the window. it controls the set of lights for upstairs, the downstairs living area, and the master bedroom.

# Design Goals and Notes

   1. **Create several *Ambient-Light_S#* Entities**. Create two to three entities, as 'sensor' objects, whose current state represents the current ambient light level in the locations where I have placed them. Use these entities to drive automatons that key off of ambient light. Locations I have in mind are:
	   1. Upstairs. I am thinking this one might be able to service as the general 'outdoors' light level sensor. Still TBD on this tho.
	   2. Kitchen Area.
	   3. Master Bedroom.
   2. **Light Sets**. Each sensor will be associated to a particular set of lights. That is the set of lights that will respond to that sensor's changes in it's *Reported States*. These light sets will be defined by adding specific Labels to each light that I want to have in the set for that sensor. The naming convention for these labels will be:
	   1. Lightset - Day-Dim | <area> \<area\> . The ' \<area\>' part will be something like 'Kitchen' or 'Living Room,' etc. The 'Day-Dim' part means this is the set of lights to come on when it is dim or dark during the day.
	   2. Lightset | Evening. 'Evening' means this is the set of lights to come on when Dim/Dark occur in the Evening or Night. Although note that as of April-2025 I have not incorporated the Evening lights on automation into the ambient light sensors. They are still set to come on based on the 'Daypart' sensor's state going to 'Evening.'
   3. **Other Notes / Goals**:
	   1. **Light Only**. Note that I intend these sensors to truly be for light levels. It should not be thought of as a time-of-day indicator. Don't use this for Bedtime, or Waking Up time, etc. I will create a separate template sensor for that.
		   1. 02-15-2025: I have done so, it is '`sensor.daypart`.'
	   2. **Code Abstraction**. One of the goals of using these sensor entities as the automation drivers and triggers is to be able to abstract the derivation of the estimated light levels 'behind the scenes' so that as I enhance and modify automations I won't need to modify all the automations that key off of those particular ambient light sensors. For example - allowing me to tweak the lumen level of, say 'Dim' and then not having to go into each automation that key's off of a dim level of light to change the value in each of those.
	   3. **Discrete States**. The purpose is to be able to trigger actions based on sensible changes in the light. This is not a sensor that reports a scalar level of brightness or lumens.
		   1. But I do intend to also make those scalar lumen levels available to an HA dashboard, if for no other reason so that I can use those values to calibrate the quantized states.
	   4. **The Reported States**. I'm not reporting scalar lumen values; rather I am reporting discrete quantized states that I can act on directly without having to 'translate' those inside of each individual automation. Note that as of 2/20/2025 I have implemented the reported quantized lux states using an HA Helper and an automation, and not on the device itself, so that I can more easily work out the design in a prototyping mode. Once I get it working to my satisfaction my intent is to implement those into the ESPHome device's config so that they are simply available as a sensor value coming from the sensor device. The quantized states of these ambient sensors are:
		   1. Bright
		   2. Dim
		   3. Dark
		   4. Night (or 'Black')
		   5. Unavailable <- This is the standard state HA reports when it literally cannot connect to a device, or otherwise is unable to compute or determine the device's, or template's, current state.
   4. **Challenge-A, Partly Cloudy**. In playing around so far I have noticed that the light level can flip around pretty rapidly based on cloud movement and/or cloud cover thickening and thinning out. Sometimes as rapidly as 5-10 minutes between state shifts. So here is how I am dealing with this:
	   1. First: A bit of smoothing. See the section below on that.
	   2. Second: Before turning the lights off due to the arrival of 'Bright' I insert a delay of one hour. If the quantized light level drops back down below 'Bright' while waiting for this hour to elapse I reset the timer and revert back to Dim Lights On, in Idle mode. But if the light level is still 'Bright' at the expiry of the timed delay I will proceed with turning the lights off. So the result is that if the brightness level is varying back and forth across the day then we just leave the lights on. Which does match what I'd normally do left to my own devices.
   5. **Challenge-B, Bright Lux vs Cloudy Lux**. I have noticed a curious thing. On 2/19/2025 it was a full day of full overcast and on/off snow showers. All across the day the lux readings bounced around between the mid300's and the low 3,000s. Yet the next day, while it was clear and sunny, clearly "Bright as day" with no need to turn lights on at all, I had readings ranging as low as the low 300's. So that's way too much overlap in lux readings vs what my human eye perceives as Bright vs Dim or Dark. **This needs further investigation**.

## Smoothing

### As of 2-19-2025 - Prototyping:

I do have a light sensor physical device integrated into HA, using ESPHome. It is pointing out the upstairs window. And I am driving a 'daytime dim / overcast' automation from it. I have noticed that despite my attempt to smooth out reported lux level variation by using the moving average 'filter' in the device's ESPHome .yaml config it is still flipping between 'bright' and 'dim' every 5-30 minutes based on how clouds are moving across the sky. And, mind you, on this day it is solidly overcast, so we are not talking about summer thunderheads where it can go from bright sun to dark sky very quickly. To the naked eye the light level variations are barely noticeable. 

I think the moving average smoothing is still a good idea in the device's code. But I need to add something to this.

First, note that as of right now I have implemented the reported quantized lux states using an HA Helper and an automation, and not on the device itself, so that I can more easily work out the design in a prototyping mode. Once I get it working to my satisfaction I will move the implementation into the ESPHome device's config.

# Application Architecture and Implementation

## Design

   1. The design consists of a combination of Labels, Helpers, Automations and scripts.
   2. I have constructed the Automations and Scripts to be usable by any light-set so that they are not replicated for each light-set.
	   1.  This is accomplished by passing in a key for the light-set to be controlled; and where necessary using a hard-coded lookup map to set internal variables as needed for the given light-set. This is nice, but it does mean that adding/removing any light-sets will require modifying those lookup maps in each Automation and Script. At some future point it might be worth investigating the possibility of using a text Helper to store a C-like data 'struct' that could then contain the needed data so that it is defined in one, and only one, place.
		   1. Turns out you can't do it; but only because the max length of text that can be stored in a text helper is 255 characters. Too bad, because otherwise there is actually a provision for reading JSON text and marshalling it into yaml / jinja variables. Per CharGPT anyway.

## Documentation

See the LibreOffice Draw document:  */Documentation/doc_diagrams.odg*.


### Naming Conventions

#### Labels

   * **Light-Set - Day-Dim | \<area\>**. The light sets that each sensor controls will be defined by applying these labels to each light that is to be in the set.
	   * NOTE: There is a challenge when 'Bright' occurs for a given light-set. While is it Dim or Dark a human may have turned on additional lights that are not in the defined light-set. I handle this by using Rooms to turn off lights upon 'Bright' state occurring. 
   * **App | Light Set**. Used to mark the collection of entities that constitute this Application.

#### Helpers

The below set of 'Helpers' I am thinking of as variables. Things whose values I can change in code (i.e., automations). I initially tried to use Attributes in a custom sensor template to accomplish this; but after much effort, and a session with ChatGPT I determined that, by design, it is not possible to dynamically set/change the value of those attributes. Thus the use of Helpers.

   * **Ambient S\<#\> | Quantized**. A light 'sensor' which controls a labeled light-set. I.e.:
	   * *Ambient S1 | Quantized* = Sensor that controls the Kitchen light-set.
	   * *Ambient S2 | Quantized* = Sensor that controls the Living Area light-set.
	   * *Ambient S9 | Quantized* = Dummy sensor used for application dev and testing.
   * **Ambient S\<#\> | Light-Set Label**. Text helper that contains a string which is the text of the Light-Set Label that this sensor number controls.
   * **Light-Set\<area\> | Off Areas**. Text helper that contains a list of the Rooms to be turned off when 'Bright' happens for the light-set.
   * **Light-Set - \<area\> | Timer**. Timer helper used to delay the turning off of lights upon transition from Dim/Dark to Bright.
   * **Light-Set - \<area\> | State**. A helper (variable) which stores the current state of the relevant light-set. See the state-transition diagram in  */Documentation/doc_diagrams.odg*. Possible states are:
	   * **Inactive**. Nothing within HA is in-process or pending with regard to the light-set in question, but the light-set is 'armed' and will respond to its associated ambient light sensor.
	   * **In Process - ON**. HA is currently turning the light set ON.
	   * **Idle**. The light set has, at some time in the recent past, been turned on. That action has been completed. So the lights are just sitting there, on, with no other actions by HA in process or pending. Note that anyone in the house could be manually turning lights on/off during this time. My current thinking is that I don't need to account for this. BUT I do want the ability for a human to manually set the state of this helper to 'Inactive' in order to have the effect of manually cancelling the (future) automated light set turning off process (for reasons the human would know that the computers and sensors cannot know).
	   * **Pending - OFF**. Ambient light has transitioned to 'Bright' (from any other dimmer level), so we are hypothesizing that it's ok to turn the light set off now. However, we want to wait awhile just to make sure we don't have clouds coming and going, causing us to be constantly turning the light set on/off at too frequent intervals. This is meant to work in conjunction with the *Dim-Bright Transition Counter*.
	   * **In Process - OFF**. HA is currently turning the light set OFF.
	   * **Cancel In Process**. Light-set is in some active state, but we wish to cancel it even tho it's controlling ambient light sensor isn't reporting 'Bright' light. Might be initiated by a human; or might be called as part of some other triggering event (e.g., we have transitioned into Evening while it is Dim outside).
	   * **Deactivated**. When in this state HA does not perform any automated actions on the light set. My intention is that this state be set, manually, by a Human. This state will transition back to *Inactive* during the middle of the night so that the light-set is automatically rearmed for the next day.
	   * **Disabled**. When in this state HA does not perform any automated actions on the light set. My intention is that this state be set, manually, by a Human.  This state is permanent until a Human sets it back to 'Inactive' manually.
   * **Light-Set - \<area\> | Operational State**. A helper (variable) which stores what I am calling the "Operation State" of the light-set. I am using this to initiate the *cancel*, *Deactivated*, and *Disabled* states of the light-set. And also it's reactivation. Possible states are:
	   * **Activate**. Rearm the light-set. This will move the light-set's state back into *Inactive*.
	   * **Cancel**. Stop any in-flight activity of the light-set and move it back to the *Inactive* state. NOTE: This will leave any lights that are currently on, on.
	   * **Deactivate**. Stop any in-flight activity of the light-set and move it's state to *Deactivated*. NOTE: This will leave any lights that are currently on, on.
	   * **Disable**. Stop any in-flight activity of the light-set and move it's state to *Disabled*. NOTE: This will leave any lights that are currently on, on.

#### Dependency Helpers

   * **Daypart**. A custom sensor that reports what part of the day we are in, based on the elevation of the Sun. Possible states are Morning, Day, Evening, and Night. Used for conditional tests in the application's automations.
   * **Light Set | All OFF | In Progress**. Set by:
	   * Automation `Light Set | All Off` 
   * **Light Quantization Config Values**. The set of configuration settings to define the lux values I consider to be Bright, Dim, Dark and Night. NOTE that my intention is to eventually encode these directly on the ESP32 devices so that the quantized values just get reported as the primary sensor state. For now it is handy to be able to play around with these values as I figure it all out. These helpers are:
	   * **Ambient-Light_Dark Lux**. Type: Input_number.
	   * **Ambient-Light_Dim Lux**. Type: Input_number.
	   * **Ambient-Light_Night Lux**. Type: Input_number.



## References

See chatGPT chat session: [Timer Automation in HA](https://chatgpt.com/c/67b60323-13c0-8013-8cb1-7f013944e6f5) 

Home Assistant "**Template**" documentation page @ https://www.home-assistant.io/integrations/template/
