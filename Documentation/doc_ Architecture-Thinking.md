Home assistant 

# Architectural Thinking 

## Automations

### Categories

Automations fall into the following categories: 

   1. **Actions**. Think of these a little bit like helper functions these should not have triggers but just the actions that represent a unit of work. These are called from Event automations. I have thought about subclassing these into atomic actions, versus compound actions. And maybe at some point if I get carried away enough with automating things it might be worth doing that. But for now I don't think it's worth it. However it's probably not a bad idea to keep the concept in mind when deciding how to divide the workup into actions. That is to say trying to maintain practical sense of modularity. 
	   1.    Actually, in UML an 'Action' is atomic ("...*an executable atomic computation that results in a change in the state of the model or the return of a value.*") and 'Activity' "...*Activity is an ongoing non-atomic execution within a state machine*." | source: [https://www.visual-paradigm.com/guide/uml-unified-modeling-language/about-state-diagrams/](https://www.visual-paradigm.com/guide/uml-unified-modeling-language/about-state-diagrams/)

   3. **Events.** These automations contain the triggers. These define the sort of events I am interested in having HA respond to. Events can call zero, one, or more actions as they response to the event. Zero is permissible because given other conditions, I.e the context around that event when it occurs, perhaps the appropriate response is to do nothing. For example, when normal bedtime arrives if the configuration flag for Jeff is home is true then don't do anything. If that flag is false then initiate the bedtime simulation routine. 

   4. **Sync**. These are standalone automations used to keep devices in sync. For example, keeping a plug-in device in sync with a wall switch. These could be called from either an event or an action. They can also have their own triggers. For example, sensing when either the plug or the wall switch has changed State and then setting the other device to match it. 

   5. **Other**. Maybe there are things that wouldn't fit into the above categories. 


## State Machine 

As an alternative to the above should I consider thinking of the whole schema as a big state machine? 

In this schema I would have *States*, which I assume would be represented by helpers; and there would be *State Transitions*, which would be the automations. 

In this schema there are still events and actions which is what caused the transitions to different states. But it might be cleaner to think of it as a state machine and that might affect how I organize things. 

## Sensors

I'm not sure what to say about sensors at this point. There are sensors Some of them are artificially created, in a sense, using a helper and an automation. Some of them are custom sensors that I create using the Template Helper, or coding them in yaml in VS Code. Some of them are defined in my little DIY ESPHome devices. Some of them just automatically appear as part of integrations built into HA, like the 'sun' object for instance.

Sensors, of course, are a primary source of triggers for automations.


