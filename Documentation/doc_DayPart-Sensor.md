# Purpose

Keep notes and documentation on the custom template sensor that provides a state value for what part of the day we are in.

This sensor is: `sensor.daypart`

# Design Goals and Notes

   1. **Always Available Sensor**. Have available, within HA, an always available 'sensor' that reports what part of the 24 hour day cycle we are in at the moment.
   2. **Based on Sun Position**. To automatically account for seasonal variation use the sun object and base the sensor on the sun's current position above the horizon. So 'morning,' 'evening,' etc will be based on the current seasonal length of daylight.
   3. **Quantize Into Discrete States**. The sensor is to report out only the following set of values:
	   1. Day
	   2. Evening
	   3. Night
	   4. Morning
	   5. Unavailable <- This is the standard state HA reports when it literally cannot connect to a device, or otherwise is unable to compute or determine the device's, or template's, current state.

# Create as Custom Sensor

With help from chatGPT I was able to figure out how to create a custom template sensor.
### sun.sun Device

Note that the required sun.sun 'device' is an 'integration.' It was installed by default so I didn't have to manually install that integration. BUT I did have to find my way into it's settings and 'enable' the entities for Elevation and Azimuth. 

## Code (with comments)

See my companion document: **/Code/code_Daypart.yaml**  

| Condition | Elevation Trigger | Time Condition             | Conditions                                                             |
| --------- | ----------------- | -------------------------- | ---------------------------------------------------------------------- |
| Morning   | -6.0              | past Midnight, before Noon | ((> Morning AND <= Day) AND before-Noon)                               |
| Day       | 12.0              | top half                   | (> Day AND before-Noon)<br>OR<br>(> Evening AND after-Noon)            |
| Evening   | 5.0               | past Noon, before Midnight | ((<= Evening AND > Night) AND after-Noon)                              |
| Night     | -8.0              | bottom half                | (<= Night AND before-Midnight)<br>OR<br>(< Morning AND after-Midnight) |

## References

Home Assistant "**Template**" documentation page @ https://www.home-assistant.io/integrations/template/
