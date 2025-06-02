# Purpose

Describe the configuration I have created to handle the event 'Last Awake House Occupant is Heading Off to Bed Now'

NOTE: In this I do not need to account for Ali being home or not. I have the separate 'Set Ali Night Scene' button. So when she is home I press that one instead and leave it to her to use the Head-to-Bed button when she actually goes off to bed.

# Goals

   1. The event is triggered by pressing either of the 'Head to Bed' buttons on the kitchen or master bedroom keypads.
   2. When triggered all the non-night-lights are slowly turned off, in an order that is useful for making my way to bed. I.e., leaving the kitchen and hallway niche lights for last so that I can stop in the kitchen for some last minute whatever; and the hallway to adjust the thermostat and such.
   3. If triggered before the Evening Dim event has fired, or completed, this event will cancel or block that from happening for the rest of the day.
   4. If triggered while Daypart is 'Day' then it will also Deactivate all the Day-Dim lightsets for the rest of the day.
   5. Allow dual-use of the 'Head to Bed' keypad buttons. If either of those buttons are pressed at a time of day that is, as a practical matter, clearly not a heading off to bed event, then treat such a button press, instead, as a 'Turn off All the Lights During Daytime, But I'm Still Awake and Active' event. E.g., perhaps I'm leaving the house and simply want to turn all the lights off as I exit.

# Assumptions

   1. Evening and Night light-sets are structured the same as the Day-Dim light-sets are. The only real differences being:
    1. The 'sensors' which trigger them to turn on are not ambient light sensors, but the Daypart sensor (tho ideally I'd actually change this become the ambient light sensors as some point in the future).
    2. Once on, they don't turn off by any 'sensor' - they turn off, if at all, by the occurrence of other events. And thus after bring turned on their state just goes straight to 'Inactive' and not 'Idle.'
  2. All light-sets have their '*Deactivated*' state set back to '*Inactive*' upon the occurrence of the **Daypart** sensor's '*Morning*' event and not before then.

# Design

The Ambient Light / Day-Dim application is implicated in this. As defined in the above set of Goals the occurrence of this even will, at times, need to assert some control over either, or all, of the Day-Dim, Evening, and Night light-sets and their current states. 

However, due to the Assumptions listed above, and esp Assumption #2, I believe the following, quite simple, sequence will correctly manage any of the light-sets and properly accomplish the goals:

   1. FOR EACH [light-set]:
    1. IF light-set state NOT IN (Deactivated, Disabled, Deactivated, Inactive, Cancel In Process) THEN 
        1. invoke the 'Cancel' script for light-set.
     2. Wait for 'Cancel' script to complete (putting this here will automatically account for the condition that a cancel was already in process before we even initiated the Head-to-Bed sequence; as well as handing the situation where we just initiated a cancel in the prior step).
     3. IF light-set state NOT EQ 'Disabled' THEN set [light-set] state to 'Deactivated' 
   2. Run the *Nighttime Shutdown* automation. 

Note that I do not need to turn off any lights that were turned on for the light-set since the 'Head-to-Bed' action will turn all the house lights off anyway.

I think it is this simple.

# Revision History

## 2025-05-25
   * Fist articulation of the design.
   * 