# Purpose

Notes on my attempt to create an automation that will 'sense' if I have a morning alarm set on my Pixel phone. If so, then some number of minutes prior to that alarm going off I want to have the bedroom lights fade on somewhat like a sunrise.

# Method

## Expose Pixel 'Next Alarm' Sensor

After adding the pixel integration to HA, I enabled/exposed the 'next alarm' sensor entity. Don't fully remember how I did this, but I do know that you can go into the 'HA Companion' app on the Android phone, and in there somewhere is a place that lists all possible Android sensors and for each you can 'enable/disable' them.

## Determine Date Comparison Code

This was a devil to work out. Working with dates feels like a nightmare in HA. Do you have dates only? Time only? Both? Is it seconds since 'Unix Epoc'? Milliseconds? Do your entities provide an ISO formatted string? What about UTC vs local time zone?

So I have spent time in developer tools experimenting and scoping out the above for this use case. Here is what I learned and determined:

   1. '**sensor.pixel_8_pro_next_alarm**' has a base state of 'timestamp' that returns a time string in ISO format, UTC time-zone.
	   1. So, e.g., `{{states('sensor.pixel_8_pro_next_alarm')}}`  returns > the string: "2025-01-08T11:30:00+00:00" 
	   2. This sensor also has an additional attribute that will return a unix-like timestamp in milliseconds from Unix Epoc. This attribute is: 'Time in Milliseconds' in UTC
		   1. So, e.g., you can get this by using: `{{state_attr('sensor.pixel_8_pro_next_alarm', 'Time in Milliseconds') | int / 1000 }}`  returns > 1736335800
   2. I want to use UTC for the time checks because I may have an alarm set when I am in a different time zone. So I think I want a common, consistent, time-base for the the alarm's time compared to current time at home.
   3. I feel more comfortable using a unix timestamp number for this use case as it seems less complicated overall to me.
   4. To obtain a UTC unix timestamp from the next_alarm sensor, I use the 'Time in Milliseconds' attribute per the above.
   5. After obtaining the next_alarm timestamp I back off by 10 minutes. Which I can do by simply subtracting 10 minutes worth of milliseconds.
      1. This gets me that value: `{{(state_attr('sensor.pixel_8_pro_next_alarm', 'Time in Milliseconds') | int / 1000) - (10 * 60)}}` 
	      1. That is: Get the sensor time, in millis, as a integer. Divide by 1000 to convert the result into seconds. Then with that value of the number of seconds since Unix Epoc, subtract 10 minutes worth of seconds to get the time at which we want to activate the wake up automation.
   6. Now compare the above result to the current time's 'timestamp', and if there is a match, trigger the automation.
   7. This gets me the integer part of the current time, as a UTC timestamp: `{{utcnow().timestamp() | int }}`
   8. Here is the trigger test:
       1.  `{% set wake_time = (state_attr('sensor.pixel_8_pro_next_alarm', 'Time in Milliseconds') | int / 1000) - (10 * 60) %}`
         `{{ wake_time is not none and (utcnow().timestamp() | int) > wake_time}}`
   10. BUT the above will fail when no next_alarm has been set. It fails because the time value returned in that case will be the string 'none' which can't then be cast to an integer, nor used in the statements that follow. So I need to test for that condition first. Like this:
	   1. {% if states('sensor.pixel_8_pro_next_alarm') is none %} false
	     {% else %}
	        {% set wake_time = (state_attr('sensor.pixel_8_pro_next_alarm', 'Time in Milliseconds') | int / 1000) - (10 * 60) %}
	        {{ (utcnow().timestamp() | int) > wake_time}}
	     {% endif %}
	      1. Fails with: _ValueError: Template error: int got invalid input 'None' when rendering template '{% if states('sensor.pixel_8_pro_next_alarm') is none %} false {% else %} {% set wake_time = (state_attr('sensor.pixel_8_pro_next_alarm', 'Time in Milliseconds') | int / 1000) - (10 * 60) %} {{ (utcnow().timestamp() | int) > wake_time}} {% endif %}' but no default was specified_
	      2. So even if I try to first test for none it fails. 
      2. Apparently the whole template is evaluated prior to 'rendering' so you can't exclude an error condition by using a prior test within the same template.  

So - forget the use of the Unix Timestamp return values. Instead, go with ChatGPT's recommendation.
   1. `{% set alarm_time = as_datetime(states('sensor.pixel_8_pro_next_alarm')) %}`
      `{{ alarm_time is not none and utcnow() >= alarm_time - timedelta(minutes=10) }}`
   2. This seems to work fine actually.

**Also Note**: If the alarm goes off on the phone, the 'next_alarm' datetime almost immediately updates to either 'Unavailable' (none), the next alarm set if there is one.

**Also Note**: If you snooze the alarm on the phone the 'next_alarm' datetime almost immediately updates back to the current alarm's new time with the snooze added.

I also need to test for a few other conditions before kicking off the actions, so I do also have a conditions block. I only want the actions to fire off if it's still night out; plus I also supress the actions if the bedroom lights are already on. This latter is important I think because the trigger is going to be tested once every minute. So there is a 10 minute window, prior to the pixel alarm actually going off, that the trigger will evaluate to True; and thus we would fire off the actions once every minute for that 10 minutes.

So here is what I ended up with:

```yaml
	alias: Pixel Alarm | Bedroom Lights On
	description: ""
	triggers:
	  - trigger: template
	    value_template: >
	      {% set alarm_time = as_datetime(states('sensor.pixel_8_pro_next_alarm'))
	      %} {{ alarm_time is not none and utcnow() >= alarm_time -
	      timedelta(minutes=10) }}
	conditions:
	  - condition: state
	    entity_id: switch.mstrbed_all_on_off
	    state: "off"
	  - condition: sun
	    after: sunset
	    before: sunrise
	  - condition: state
	    entity_id: input_boolean.jeff_away
	    state: "off"
	actions:
	  - type: turn_on
	    device_id: 856ba2a3565d44d83b08c11d2c6bb5bf
	    entity_id: c033d419ddd2872dfcc0838b408e4785
	    domain: light
	  - action: scene.turn_on
	    metadata: {}
	    data: {}
	    target:
	      entity_id: switch.mstrbed_all_on_off
	mode: single 
```
