
# Purpose

Keep track of some key things to know about in working with ESPHome devices and programming.

# Automation - Stop In Flight

There is no direct way to tell a running automation to stop. But is you use a two-step process you can do it. Turn it Off (i.e. 'disable' it) then turn it back on (i.e., 'endable' it):

Action `automation.turn_off` then follow with:
Action `automation.turn_on` 

This action disables the automation’s triggers, and optionally stops any currently active actions.

| Data attribute | Optional | Description                                                                        |
| -------------- | -------- | ---------------------------------------------------------------------------------- |
| entity_id      | no       | Entity ID of automation to turn off. Can be a list. none or all are also accepted. |
| stop_actions   | yes      | Stop any currently active actions (defaults to true)                               |

Source: [Automation actions](https://www.home-assistant.io/docs/automation/services/)

# Reboot Watchdog

ESPHome will automatically reboot periodically if no connection is made to its API. This helps in the event that there is an issue in the device’s network stack preventing it from being reachable on the network. You can adjust this behavior (or even disable automatic rebooting) using the `reboot_timeout` option in any of the following components:

   * WiFi Component
   * Native API Component
   * MQTT Client Component

Beware, however, that disabling the reboot timeout(s) effectively disables the reboot watchdog, so you will need to power-cycle the device if it proves to be/remain unreachable on the network.

# lambda Action

Templates (also known as lambdas) allow you to do almost anything in ESPHome. For example, if you want to only perform a certain automation if a certain complex formula evaluates to true, you can do that with templates. Let’s look at an example first:

``` yaml
binary_sensor:
  - platform: gpio
    name: "Cover End Stop"
    id: top_end_stop
cover:
  - platform: template
    name: Living Room Cover
    lambda: !lambda |-
      if (id(top_end_stop).state) {
        return COVER_OPEN;
      } else {
        return COVER_CLOSED;
      }
```

What’s happening here? First, we define a binary sensor (notably with id: top_end_stop) and then a template cover. (If you’re new to Home Assistant, a ‘cover’ is something like a window blind, a roller shutter, or a garage door.) The state of the template cover is controlled by a template, or “lambda”. In lambdas, you’re just writing C++ code and therefore the name lambda is used instead of Home Assistant’s “template” lingo to avoid confusion. Regardless, don’t let lambdas scare you just because you saw “C++” – writing lambdas is not that hard! Here’s a bit of a primer:

First, you might have already wondered what the lambda: !lambda |- part is supposed to mean. !lambda tells ESPHome that the following block is supposed to be interpreted as a lambda, or C++ code. Note that here, the lambda: key would actually implicitly make the following block a lambda, so in this context, you could also have written lambda: |-.

See [Automation ▸ Templates ](https://esphome.io/automations/templates#config-lambda)

