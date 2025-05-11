# Purpose

Provide ready references to Home Assistant documentation that I believe to be essential, or even just quite helpful.
## State and state object

Devices are represented in Home Assistant as [entities](https://www.home-assistant.io/docs/configuration/entities_domains/). The state of an entity (for example, if a light is on, at 50% brightness in orange) can be shown on the dashboard or be used in automations. This page looks at the concepts _state_, _state object_, and _entity state attribute_.

NOTE: Be somewhat aware of the info in this document first, I think.

## Entities and domains

Your devices are represented in Home Assistant as entities. Entities are the basic building blocks to hold data in Home Assistant. An entity represents a sensor[](https://www.home-assistant.io/integrations/sensor/), actor, or function in Home Assistant. Entities are used to monitor physical properties or to control other entities. An entity is usually part of a device or a service[](https://www.home-assistant.io/docs/scripts/perform-actions/). Entities have [states](https://www.home-assistant.io/docs/configuration/state_object/) and [state attributes](https://www.home-assistant.io/docs/configuration/state_object/#about-entity-state-attributes).

All your entities are listed in the entities table, under [**Settings** > **Devices & services** > **Entities**](https://my.home-assistant.io/redirect/entities).

Contains a long list of links to "entity domains"  or so-called  _building block integrations_ or _entity integrations_. This list could be a key reference list.

@ [Entities and Domains](https://www.home-assistant.io/docs/configuration/entities_domains/)

## Entity ID vs 'unique_id'

The unique_id one assigns in a custom template sensor or entity is NOT the same as the 'Entity ID' you see in the user interface.

unique_id is used internally in HA for housekeeping and is never actually exposed to the user in the interface.

The Entity ID is generated from the '- name:' string given in the template. Per my question about this in the HA forum. Edwin D's reply:

"*The name is what is used to generate the entity id. The entity id is what is used in automations, scripts and scenes. The unique id is not shown. It is used internally to link customization data to the entity. So nothing is truncated, you just did not put v1 in the name (which can be a nice friendly name, HA will sluggify it for the entity id).*"

See [Truncation of ‘_v1’ at End of unique_id?](https://community.home-assistant.io/t/truncation-of-v1-at-end-of-unique-id/827285)

