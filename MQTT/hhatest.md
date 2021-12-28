# Test HomeVision with Home Assistant
<!-- $Revision: 1.9 $ -->
<!-- $Date: 2021/12/23 01:21:51 $ -->

[Back to Projects Index](/index)

[Back to MQTT Index](/MQTT/MQTT_index)

### Running Other Objects
The previous Tip was called out specifically because it can be used to add in macro capability for those who have run out of macro space.
However the idea can be extended to virtually all items in HomeVisionXL. Since the trigger simply sends whatever is there to HomeVisionXL's serial command processor, you can set triggers for anything the Action plug-in can do (or any other plug-in with its own defined serial commands), and, along with the trigger's capability to make run-time substitutions in a trigger string, Home Assistant may be able to trigger different things based on the payload sent.
In an extreme case, the trigger could be just "%M", in which case Home Assistant would put in the payload the complete trigger string to execute. See
[Triggers](/MQTT/MQTT_Client_Plug-in#Triggers) in [MQTT Help](/MQTT/MQTT_Client_Plug-in)
for more details.
### Refreshing HomeVision Objects
There may be instances (like restarting Home Assistant) where the current status of HomeVision objects is not reflected by Home Assistant. The MQTT plug-in provides a special topic to force all listed objects to report their status: 
<pre>
        Full Topic                        Payload
    cmnd/homevision/<i>object_type</i>/POWER    empty or "?"
or
    cmnd/homevision/POWER                empty or "?"
</pre>

```
test
```

If you have this issue, you may want to consider adding a button to send this message, or create an automation to issue the command at an appropriate time.
### Timers
HomeVision timers can be controlled via MQTT.
There is no simple "MQTT Discovery" defined in Home Assistant that is appropriate for Timers,
so the MQTT Plug-in's Discovery sets up two "sensor" entities,
one for the timer's state (Running, Stopped, Ringing)
and one for the current time (See Note below).
Time is reported in "HHH:MM:SS.hh" format.
<br>
<br>
The text "state" is appended to the name of the timer for the state entity,
and "time" is appended to the name of the timer for the time entity.
These allow tracking of Timer state and current time, but must be triggered by an MQTT stat message.
<br>
<br>
A Demonstration on how to implement a Timer GUI is as follows.
* Run MQTT Discovery for the desired timer. This creates two sensor entities. Use these in the GUI to show the Timer state and "current" timer value.
* Create the input_select helper for the timer's control states.
(Configuration->Helpers->Add Helper->Select).
This is used to specify what timer command to send.
Use the following values:
```
    Options:
        Load
        Start
        Stop
        Clear
```
* Create the input_text helper for the timer's set time. (Configuration->Helpers->Add Helper->Text).
This is used to specify what timer value to send.
Use the following values:
```
    Max: 12
    Min: 10
    Regex pattern: ^(?:(?:([01]?\\d|2[0-3]):)?([0-5]?\\d):)?([0-5]?\\d\\.)?([0-9][0-9]$
```
Once these are available, 
create a GUI using them.
There are probably many different ways to do this, but here is one.
<br>
<br>
<!--
  <img alt="HA Timer GUI.gif" src="https://github.com/rebel7580/img/blob/master/HA Timer GUI.gif?raw=true"/>
  <img alt="HA_Outside_Deco_Config" src="HA Timer GUI.gif">
-->
<br>
<br>
This GUI uses a grid card,
with the two timer entities, the two helpers and two buttons
(to get timer status and send commands to the timer).

Here is the yaml for the above implementation.
This was created using the GUI editor, which is a lot easier than writing in yaml, but is shown here to see the details.
``` yaml
type: grid
cards:
  - type: entity
    name: Wash Timer State
    entity: sensor.wash_timer_state
  - type: entity
    entity: sensor.wash_timer_time
    name: Wash Timer Status
  - type: entity
    entity: input_select.wash_timer_set
  - type: entity
    entity: input_text.wash_timer_time
    name: Wash Timer Time Set
  - type: button
    tap_action:
      action: call-service
      service: mqtt.publish
      service_data:
        topic: cmnd/WashTimer/POWER
        payload_template: |
          {% if is_state("input_select.wash_timer_set", "Load") %}
              load {{ states('input_text.wash_timer_time') }}
          {%-else %}
             {{ states("input_select.wash_timer_set") }}
          {% endif %}
      target: {}
    hold_action:
      action: none
    name: Update Timer
    show_icon: false
  - type: button
    tap_action:
      action: call-service
      service: mqtt.publish
      service_data:
        topic: cmnd/WashTimer/POWER
        payload: '?'
      target: {}
    hold_action:
      action: none
    show_icon: false
    name: Get Timer Status
columns: 2
square: false
```

some text

```
          {%-else %}
```

<b>Note:</b> 55555555555555 The timer's current time in the GUI does <i>not</i> update automatically, but only when receiving a "stat" update.
"stat" updates occur whenever a timer command (load, start,stop,clear or query - i.e., "?") is sent to the timer.

### Retain Option for Objects
You may also want to consider, as a possible alternative to the previous, whether to enable "retain" for objects that are tracked by Home Assistant.
This should allow Home Assistant to automatically pick up status via the MQTT broker's retained messages feature.
"Retain" is an option in the MQTT plug-in's <i>Configure Object</i> screen.
<br>
<br>
<b>DO NOT confuse</b> the above "retain" with the retain attribute in the Home Assistant discovery payload.
The retain in the Home Assistant discovery payload governs the retain setting of messages sent OUT by HomeAssistant, and is valid only for objects defined as switch, light or climate.
