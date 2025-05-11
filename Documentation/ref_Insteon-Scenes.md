# Purpose

The doc provides an explanation of how Insteon scenes work.

# Notes RE Home Assistant

## Scene Not Working Right?

I had a situation where I added a plug in module to an existing scene in HA. After I did so that module turned off just fine with the scene was activated to 'turn off.' But that module would not turn on when the scene was activated to 'turn on.' Upon inspection of that device's All Links Database (ALdb) Data1 was set to '0' - meaning that it's 'on' value for scene 'ON' command was to actually go to OFF state. I edited that to a '1', saved the update to the device, and all was well.

My initial conjecture is that when adding devices to a scene the three data values will be set to the existing state the device happens to be in at the time of scene creation, or device addition. So this is something to watch for.

# How to Think About It

Note that the below, to me, implies that for any given Responding device they only react to a *unique combination of Controller-ID AND Group#*. Meaning that I could be a switch that 'Responds' to Controller-A, Group-25. But there could be a Controller-B that puts out a Group-25 as well. But if I don't have Controller-B in my All Link Database then I won't respond to Group-25 messages coming from it.

By HA Insteon Integration Developer, teharris1, May 2022:

Here is how to think about it:

1. Controllers send a broadcast message to all devices. The controller then sends direct messages to any known responders in order to make sure those responders heard the message. This is a fallback mechanism.
2. Responders listen for messages from a controller and respond accordingly.
3. The group number is important because a responder can be configured to respond differently based on the group that the controller triggers.
4. Data1, data2, and data3 are important because they hold the state information that the responder uses to know what state to go to when a specific controller triggers a group
5. Group 0 is special and is really only used between the modem and a device to allow the modem to control basic functions in a device like setting properties or writing to the ALDB of the device.

So as an example, let’s have a modem with address: 1A.2B.3C and devices AA.BB.CC as a dimmable light switch and DD.EE.FF is an on/off outlet with controllable top and bottom sockets.

Modem ALDB:

```
	Group  Target     Mode    Data1  Data2   Data3
	0      AA.BB.CC   Contr       0      0       0
	0      DD.EE.FF   Contr       0      0       0
	1      AA.BB.CC   Respd       0      0       0
	1      DD.EE.FF   Respd       0      0       0
	2      DD.EE.FF   Respd       0      0       0
	22     AA.BB.CC   Contr       0      0       0
	23     DD.EE.FF   Contr       0      0       0
```


AA.BB.CC ALDB:

```
	Group  Target     Mode    Data1  Data2   Data3
	0       1A.2B.3C  Respd       0      0       0
	1       1A.2B.3C  Contr       0      0       0
	22      1A.2B.3C  Respd     127     28       1
	1       DD.EE.FF  Contr       0      0       0
```


DD.EE.FF ALDB:

```
	Group  Target     Mode    Data1  Data2   Data3
	0      1A.2B.3C   Respd       0      0       0
	1      1A.2B.3C   Contr       0      0       0
	2      1A.2B.3C   Contr       0      0       0
	23     1A.2B.3C   Respd     255      0       1 
	1      AA.BB.CC   Respd     255      0       2
```

The dimmable switch AA.BB.CC has one “button” which is also known as group 1. The on/off outlet has two “buttons” or groups. The top is group 1 and the bottom is group 2. (By the way, an 8-button KPL has 8 groups).

So starting with the “default links” between the devices and the modem, each device is a responder to the modem at group 0. This means the device will respond to base-level commands from the controller such as reading and writing to the ALDB or properties of the device. The modem is then a responder to each of the buttons of that device (group/button 1 for AA.BB.CC and groups/buttons 1 & 2 for DD.EE.FF). This allows the modem to hear changes in the device that are manually made by a person at the device. These links are not strictly necessary but they do help with reliability. The modem will hear any message from any device that is in its ALDB but, as I said above, controllers send direct messages to any device they see as a responder. So when button 1 is pressed, the device sends a broadcast message that the button changed. Then they send a follow-up message to each device that they know is a responder to make sure the known responders saw the message.

In the links above, there are two scenes, 22 and 23. When the modem sends a “scene on” for group 22, device AA.BB.CC goes to an on level of 127 (aka 50%) with a ramp rate of 0.5 seconds for button 1. For any dimmable button: data1 is the on level, data2 is the ramp rate and data3 is the button that is affected.

When the modem sends a “scene on” for group 23, device DD.EE.FF goes to an on level of 255 (aka 100%) for button 2. For any on/off button: data1 is the on-level but since it is on/off only 0 and 255 are valid numbers, data2 not used and data3 is the button that is affected.

Finally, there is a link between AA.BB.CC button 1 and DD.EE.FF button 2. When the switch is turned on it sends a broadcast of group 1 on. Device DD.EE.FF hears that broadcast and turns button 2 on.

The reason there are groups 1 - 8 is because there are devices that have up to 8 buttons. I am not aware of any device with more than 8 buttons. It is common practice to start scenes above 10 or 20 in order to avoid confusion with the button groups. That is just convention, not technically important. You can have group 1 be a scene if you wanted to but it would make it very difficult to see if a device is in that group. Hopefully, that explains the links well enough.

**Source**: [Insteon Configuration Panel](https://community.home-assistant.io/t/insteon-configuration-panel/234145/269)
