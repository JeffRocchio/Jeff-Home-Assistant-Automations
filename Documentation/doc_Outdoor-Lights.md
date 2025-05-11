Last Updated: 1/23/2025
# Purpose

Document stuff related to my configuration for the outdoor lights.

# Interim Migration Notes

   1. The Insteon scene I created within HA - scene # 22 'Outdoor Lights | Front"' does not include the Kitchen keypad button (#4) yet since that device has not yet been brought into HA. Need to add that to the scene once I bring that keypad over.
   2. Need to add entry to the kitchen ALdb as a Responder, responding on it's button #4, to the master bedroom keypad, on group #4 once I bring the kitchen keypad over.

# Side Patio

There are two switches for the patio wall-mounted 'coach lights.' NOTE: The spotlight pair under the eve in the outer corner is not on any smartswitch at this time.

Those two switches, one in the dining area and the other in the master bedroom, are locally linked into a virtual '3-way.' This is accomplished by entries in each of the insteon switches' local All Links Database (ALdb), not in an HA automation or scene. 

I manually created these links in each of the insteon switches' All Links Database (ALdb) by using HA's Insteon integration's feature for being able to create ALdb links in specific devices.

# Driveway and Front Door

I have four switches that are involved in controlling the outdoor lights for the Driveway and Front Door. These are:

   - Switch by front door that locally controls the actual light outside the front door.
	   - Note that this controls only the local load. Turning this switch on/off will not trigger any other action.
   - Switch in Garage that locally controls the actual lights outside the garage door.
	   - Note that this controls only the local load. Turning this switch on/off will not trigger any other action.
   - Button #4 in the Kitchen keypad that will turn on/off both the Front Door and Garage outdoor lights.
	   - This is done locally, via entries in the All Links Database of the four devices.
	   - This keypad is configured, by it's ALdb, as a Controller, for group-4. The other three devices have entries in their ALdb as responders to this controller on group-4.  The other three devices have entries in their ALdb as responders to this controller on group-4.
	   - *INTENTION Once I bring the Kitchen keypad device into HA*: This keypad is also configured, by it's ALdb, as a Responder to the Master Bedroom Keypad, for group-4. It 'responds' by turning on/off it's button #4.
   - Button #4 in the Master Bedroom keypad that will turn on/off both the Front Door and Garage outdoor lights.
	   - Ditto for this switch as for the kitchen keypad button.

I manually created the above noted links in each of the insteon switches' All Links Database (ALdb) by using HA's Insteon integration's feature for being able to create ALdb links in specific devices.

So the above schema has configured those four switches to function in a totally local control mode, with no involvement by Home Assistant.

## Home Assistant Scene

To permit effect control of the grouping of Driveway and Front Door lights, and keep the two keypad button LEDs in sync, I also created an 'Insteon scene' in HA which I can call in an automation, or put on a dashboard.
