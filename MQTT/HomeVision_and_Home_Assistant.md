<!-- $Revision: 1.9 $ -->
<!-- $Date: 2021/12/23 01:21:51 $ -->
<!-- <h1 id="tips-for-interfacing-homevision-with-home-assistant">Tips for Interfacing HomeVision with Home Assistant</h1> -->
# Tips for Interfacing HomeVision with Home Assistant
{:.no_toc}

[Back to Projects Index](/index)

[Back to MQTT Index](/MQTT/MQTT_index)

* Overview
  * Basics of the HomeVisionXL MQTT plug-in
    * How the MQTT plug-in Handles Internal Objects
    * How the MQTT plug-in Handles External Devices
  * MQTT Auto Discovery
  * Tips
    * Variable Options
    * Running Macros
    * Running Macros, Setting Flags and Variables and Executing Other Actions
    * Running Scheduled and Periodic Events
    * Running Other Objects
    * Timers
    * Refreshing HomeVision Objects
    * Retain Option for Objects
{:toc}
<!-- <h1 id="overview">Overview</h1> -->
# Overview
This help discusses ways to connect Home Assistant to your HomeVision controller running HomeVisionXL.
Emphasis is on using MQTT as the connecting method and assumes you have an MQTT broker running in your system.
With the versatility of the MQTT plug-in with respect to how many different ways you can use it to control your devices along with the complexity and power of Home Assistant, the possible combinations are almost endless.
This document tries to give a few of the more obvious solutions to common situations.

<!-- <h2 id="basics-of-the-homevisionxl-mqtt-plug-in">Basics of the HomeVisionXL MQTT plug-in</h2> -->
## Basics of the HomeVisionXL MQTT plug-in
First let's go over the basics. The MQTT plug-in provides support for monitoring and controlling of both HomeVision "internal" objects like x10, lights, vars, flags, etc. via MQTT, and "External" devices such as ESP8266 based products running
<a href="https://tasmota.github.io/docs/">Tasmota</a>
software.
This Help doesn't go into all the details of the MQTT plug-in. See
<a href="MQTT_Client_Plug-in">MQTT Help</a>
for that.
<!-- <h3 id="how-the-mqtt-plug-in-handles-internal-objects">How the MQTT plug-in Handles Internal Objects</h3> -->
### How the MQTT plug-in Handles Internal Objects

The MQTT plug-in exposes internal objects to the MQTT system by <i>publishing</i> STATE topics to report an object's state and by <i>subscribing</i> to COMMAND topics that can control the internal object. Only internal objects included in the MQTT plug-in's <b>Int Objects</b> configuration screen are exposed.

When an exposed internal object changes state, it produces one or two MQTT messages, depending on options chosen in "Settings". For example:

<pre>
    stat/topic/POWER ON
</pre> 

and/or

<pre>
    stat/topic/RESULT {"POWER":"ON","Dimming":"100}
</pre>

An internal object can be controlled by sending an MQTT message like this:

<pre>
    cmnd/topic/POWER ON 50
</pre>

<!-- <h3 id="how-the-mqtt-plug-in-handles-external-devices">How the MQTT plug-in Handles External Devices</h3> -->
### How the MQTT plug-in Handles External Devices
The MQTT plug-in can track the state of and control external devices by <i>subscribing</i> to STATE topics and <i>publishing</i> command messages. Only external devices included in the MQTT plug-in's "Ext Devices" configuration screen are monitored.

When an external device changes state, it typically produces one or two MQTT messages, depending on device settings:

<pre>
    stat/topic/POWER ON
</pre> 

or

<pre>
    stat/topic/RESULT {"POWER":"ON","Dimming":"100}
</pre>

The MQTT plug-in can respond to these STATE messages in a number of ways, such as changing flags or variables, running macros, executing procedures, or any combination of these. 
<br>
<br>
An external device can be controlled by sending an MQTT message like this:

<pre>
    cmnd/topic/POWER ON 50
</pre>

It's important to keep in mind that the above are examples, and there is significant variation in how all this is done, which can be taken advantage of when interfacing with Home Assistant. For Home Assistant, this means that "virtual" external devices can be created and used to cause actions in HomeVision.
<!-- <h4 id="virtual-external-devices">Virtual External Devices</h4> -->
#### Virtual External Devices
A "virtual" external device is a external device configuration entry in the MQTT Plug-in's <b>Ext Devices</b> tab that has <i>no corresponding physical device</i>.
But, since it will have a topic, it can be accessed by any MQTT entity, and in turn it can run the Flag, Variable or Commands set up in that "virtual" external device's configuration entry.

In the <i>Tips</i> section to follow, there are several examples where this capability is used.
<!-- <h2 id="mqtt-auto-discovery">MQTT Auto Discovery</h2> -->
## MQTT Auto Discovery
For systems where you want to expose a significant number of internal objects to Home Assistant, the MQTT plug-in provides an Auto Discovery feature that pushes discovery messages to Home Assistant, negating the need to enter each object's code into your <i>configuration.yaml</i>.
<br>
<br>
Valid object types are x10, light, var, var_n, flag, flag_b, input, output, analog, temp, ir, macro, se, pe, timer and hvac.
Other plug-ins that support objects that can be discovered can add object types.
<br>
<br>
You can find out more about how to run Discovery here:
<a href="HomeVision_Discovery_How-to">How to Use the MQTT Plug-in's Home Assistant Auto Discovery</a>

<!-- <h2 id="tips">Tips</h2> -->
## Tips
<!-- <h3 id="variable-options">Variable Options</h3> -->
### Variable Options
Variables are supported as either sensor (var) or number (var_n) entities, or both.

Straightforward, bi-directional control of variables from the Home Assistant UI is supported via the MQTT Discovery number entity (use var_n).

They can be included in discovery as read-only sensors (use var).
(The MQTT Plug-in itself allows both reading and writing of variables.)

<!-- <h4 id="variable-example">Variable Example</h4> -->
#### Variable Example
<b>Note: variables as number entities, which appear in the Home Assistant UI as sliders, is a recent addition to discovery. The following example, devised before number entities were available to MQTT discovery, is a technique to essentially provide the same functionality with a variable sensor. As such, it is kept here for reference as some of the techniques included may be useful.</b>

For applications other than sensors, it is possible to include HomeVision variables as part of automations, so application-specific configurations could be done.

For a simple application of a variable, 
create a slider that takes its value from a State message from HomeVisionXL and transmits back a new value if the slider is manually changed.
This can be done by creating an "input number" entity (to create the slider) along with two automations. 
<br>
<br>
For each variable you want to expose in Home Assistant:
<ul>
<li>First create the input_number entity for the variable you want to set/read.
</li><li>Create an automation to <i>get</i> the value of the HomeVision variable.
</li><li>Create an automation to <i>set</i> the value of the HomeVision variable.
</li></ul>
You do not need to create both automations if you only want to either just read or just set the variable.
<br>
<br>
The easiest way to create the input_number entity in Home Assistant is Configuration->Helpers->Add Helper->Number.
<br>
<br>
There are blueprints for the two automations. You can get them into Home Assistant by Configuration->Blueprints->Import Blueprints and use the following URIs:
<br>
<br>
<a href="http://github.com/rebel7580/Home-Assistant/blob/master/get_variable.yaml">https://github.com/rebel7580/Home-Assistant/blob/master/get_variable.yaml</a>
<br>
<a href="http://github.com/rebel7580/Home-Assistant/blob/master/set_variable.yaml">https://github.com/rebel7580/Home-Assistant/blob/master/set_variable.yaml</a>
<br>
<br>
If you wish to enter directly in YAML, here is an example.
You will need to replicate these three sets of code for each variable you want to support, making sure you change the Variable names and topics.
{% raw %}
``` yaml
# Example configuration.yaml using 'input_number' in an automation
input_number:
  var_145:
    name: Var 145 Slider
    min: 1
    max: 255
    step: 1
    unit_of_measurement: step
```
{% endraw %}
The automations should be in <i>automation.yaml</i> or after the <code>automation: !include automations.yaml</code> line in <i>configuration.yaml</i>.
{% raw %}
``` yaml
# This automation script runs when a value is received via MQTT
# It sets the value slider on the GUI.
automation myvars:
  - alias: Set var 145
    trigger:
      platform: mqtt
      topic: 'stat/Var145/RESULT'
    action:
      service: input_number.set_value
      data:
        entity_id: input_number.var_145
        value: "{{ trigger.payload_json.STATE }}"

# This second automation script runs when the slider is moved.
# It publishes its value to the same MQTT topic as myvars.
  - alias: slider moved
    trigger:
      platform: state
      entity_id: input_number.var_145
    action:
      service: mqtt.publish
      data:
        topic: 'cmnd/Var145/POWER'
        payload: "{{ states('input_number.var_145') | int }}"

```
{% endraw %}

<!-- <h3 id="running-macros-setting-flags-and-variables-and-executing-other-actions">Running Macros, Setting Flags and Variables and Executing Other Actions</h3> -->
### Running Macros, Setting Flags and Variables and Executing Other Actions
There are at least three "categories" into which interactions with HomeVision and Home Assistant can be grouped.
<ul>
<li>Direct control of a Discovered object via its HomeAssistant entity.
</li><li>Control using multiple objects via created entities in yaml.
</li><li>Control using a "virtual" external device as an intermediary for complex actions.
</li></ul>
Most of the following Tips can be used with almost all of the Internal Objects, not just the ones shown in the examples.

<!-- <h4 id="simple-control-of-discovered-objects">Simple Control of Discovered Objects</h4> -->
#### Simple Control of Discovered Objects
Discovered Objects like X-10 lights, flags, outputs, macros, IR, etc. appear in Home Assistant as appropriate entities, which in most instances can be integrated directly into your Home Assistant GUI and used without very much additional work.
Here's a screenshot of a simple "entities card" with a selection of discovered light and switch entities, with some tweaks to the icons used.

<p align="center">
<img alt="MasterBath" src="MasterBath.gif">
</p>

<!-- <h4 id="using-a-toggle-macro-and-a-tracking-flag">Using a Toggle Macro and a Tracking Flag</h4> -->
#### Using a Toggle Macro and a Tracking Flag

Home Assistant automatically configures macros as switch entities (as in the previous section).
This is the easiest way to run a macro.
<br>
<br>
But what if you want a button that runs a macro, and also show the result of that action?
You can't do it directly via the macro, since it doesn't report a useful state (as macros have none).
You need another entity  for that.
<br>
<br>
For example, you have a macro ("Toggle Garage Door 1") that toggles your garage door, and an input ("Door1") that indicates the state (Open/Closed) of the door.
You'd like the state/icon of the button to show the door's state.

There are (at least) two ways to do this.
The first is probably the best way, as it does not require manual modification of the <i>configuration.yaml</i> file. The second is the "original" method in these Tips. Both start with this step:
<ul>
<li>Add the "Door1" input and "Toggle Garage Door 1" macro to the <b>Int Objects</b> list.
Run MQTT discovery for at least the "Door1" input binary sensor, but it won't hurt if you do for both, so you can use them in Home Assistant for other reasons, as well as in this case.
</li></ul>
<b>Method 1:</b>
<br>
<ul>
<li>Create a button in the HA GUI. Use the GUI editor, but here is the corresponding yaml:
</li></ul>
{% raw %}
``` yaml
type: button
tap_action:
  action: call-service
  service: mqtt.publish
  service_data:
    topic: cmnd/ToggleGarageDoor1/POWER
    payload: Run
  target: {}
entity: binary_sensor.ib_5_door1
icon: mdi:garage
icon_height: 50px
name: Garage Door 1
```
{% endraw %}

Done!

If you want "dynamic" icons, leave "icon:" blank, and define a device class for the binary sensor.
In this example, you would use a device class of "garage_door".
See <b>Device Class Note</b> at the end of [How to Use Home Assistant Auto Discovery](/MQTT/HomeVision_Discovery_How-to#device-class-note)
<br><br>
<b>Method 2:</b>
<br>
<ul><li>Manually add the following to your configuration.yaml
(HA Core 2022.6 and later):
</li></ul>
{% raw %}
``` yaml
mqtt:
  switch:
    - name: "MA_GarageDoor1"
      unique_id: "MA_GarageDoor1"
      state_topic: "stat/Door1/POWER"
      command_topic: "cmnd/ToggleGarageDoor1/POWER"
      payload_on: "ON"
      payload_off: "ON"
      state_on: "Open"
      state_off: "Closed"
      qos: 1
```
{% endraw %}
Version deprecated in HA Core 2022.6 (usable until removed in 2022.9):
{% raw %}
``` yaml
- switch:
  - platform: mqtt
    unique_id: "MA_GarageDoor1"
    name: "MA_GarageDoor1"
    state_topic: "stat/Door1/POWER"
    command_topic: "cmnd/ToggleGarageDoor1/POWER"
    payload_on: "ON"
    payload_off: "ON"
    state_on: "Open"
    state_off: "Closed"
    qos: 1
```
{% endraw %}

<ul><li>Create a button in the HA GUI. Use the GUI editor, but here is the corresponding yaml:
</li></ul>
{% raw %}
``` yaml
    type: button
    entity: switch.ma_garagedoor1
    icon: 'mdi:garage'
    name: Ron's Garage Door
```
{% endraw %}

When the button is pressed, the macro will run to toggle the door, and the input will report back the door's position, which will be reflected in the button's state and/or icon.
Since the button's state is controlled by the Door1 state, it can be "On" and "Off".
If "On", the button when press will send the "off" payload,
so both command payloads are set to "ON" since that's what the macro expects.

Same comment as in Method 1 applies for "dynamic" icons.

<!-- <h4 id="using-different-macros-for-on-and-off-via-a-virtual-external-device">Using Different Macros for ON and OFF via a Virtual External Device</h4> -->
#### Using Different Macros for ON and OFF via a Virtual External Device

While macros can be run directly by configuring them in the <b>Int Objects</b> screen and running MQTT Discovery,
there may be situations where more flexibility is needed.
For example,
you want to run different macros when sending an "on" or "off" from the same topic.
<br>
<br>
The handling of external devices has significant capabilities for executing actions within HomeVisionXL. When a MQTT <i>State</i> message is received by the MQTT plug-in, a number of actions can be done.
We can use these capabilities to enable Home Assistant to do these actions. The way to do this is to create a "virtual" external device, i.e., one defined in the "Ext Devices" MQTT configuration screen, but does not actually physically exist. Instead, we will program Home Assistant to be that "device".

First, in the MQTT Configuration <i>Ext Devices</i> Tab, set up a virtual external device with an appropriate descriptive topic and the ON and Off macros you want to run defined in the On and Off macro fields.
<br>
<br>
<p align="center">
<img alt="HA_Outside_Deco_Config" src="HA_Outside_Deco_Config.png">
</p>
<br>
<br>
Next set up a switch in Home Assistant's <i>configuration.yaml</i> Since this is an external device, you can't use MQTT discovery.
(HA Core 2022.6 and later):
{% raw %}
``` yaml
mqtt:
  switch:
    - name: "HA_Outside_Deco"
      unique_id: "HA_Outside_Deco"
      state_topic: "stat/HA_Outside_Deco/POWER"
      command_topic: "stat/HA_Outside_Deco/POWER"
      payload_on: "ON 1"
      payload_off: "OFF"
      state_on: "ON 1"
      state_off: "OFF"
      qos: 1
```
{% endraw %}
Version deprecated in HA Core 2022.6 (usable until removed in 2022.9):
{% raw %}
``` yaml
- switch:
  - platform: mqtt
    unique_id: "HA_Outside_Deco"
    name: "HA_Outside_Deco"
    state_topic: "stat/HA_Outside_Deco/POWER"
    command_topic: "stat/HA_Outside_Deco/POWER"
    payload_on: "ON 1"
    payload_off: "OFF"
    state_on: "ON 1"
    state_off: "OFF"
    qos: 1
```
{% endraw %}
Note that <i>command_topic</i> and the <i>state_topic</i> are the same. Since HomeVisionXL responds to state messages to execute actions, Home Assistant must send a "stat" message to the MQTT plug-in. Home Assistant also will listen to the <i>state_topic</i>, so will hear its own message and ensure that the switch is in the correct state.

Lastly, add in a switch to your UI.
<!-- <h4 id="using-different-macros-for-on-and-off-and-a-tracking-flag-via-a-virtual-external-device">Using Different Macros for ON and OFF and a Tracking Flag via a Virtual External Device</h4> -->
#### Using Different Macros for ON and OFF and a Tracking Flag via a Virtual External Device

There are potentially two shortcomings of the previous solution:  It requires a "virtual" external device, and, more importantly,  there is no way keep the "state" of the switch in sync with what is happening.
If, for instance, the switch is used to run a macro to turn something(s) on, but the "off" macro is executed from some other place, the switch won't change and hence will not display the actual "state". 

We can resolve the second concern by defining the "stat" parameters in the yaml switch to reflect the object used to track "state".
Let's assume we have a Flag "OutsideDecoState" which tracks whether the outside decorations are on or not. Discover it as a binary sensor.

Use the same "virtual" external devices before, but include the binary sensor in the yaml:

(HA Core 2022.6 and later):
{% raw %}
``` yaml
mqtt:
  switch:
    - name: "HA_Outside_Deco"
      unique_id: "HA_Outside_Deco"
      state_topic: "stat/OutsideDecoState/POWER"
      command_topic: "stat/HA_Outside_Deco/POWER"
      payload_on: "ON 1"
      payload_off: "OFF"
      state_on: "Set"
      state_off: "Clear"
      qos: 1
```
{% endraw %}
Version deprecated in HA Core 2022.6 (usable until removed in 2022.9):
{% raw %}
``` yaml
- switch:
  - platform: mqtt
    unique_id: "HA_Outside_Deco"
    name: "HA_Outside_Deco"
    state_topic: "stat/OutsideDecoState/POWER"
    command_topic: "stat/HA_Outside_Deco/POWER"
    payload_on: "ON 1"
    payload_off: "OFF"
    state_on: "Set"
    state_off: "Clear"
    qos: 1
```
{% endraw %}

<!-- <h4 id="using-different-macros-for-on-and-off-and-a-tracking-flag-without-a-virtual-external-device">Using Different Macros for ON and OFF and a Tracking Flag without a Virtual External Device</h4> -->
#### Using Different Macros for ON and OFF and a Tracking Flag without a Virtual External Device

To simplify further, the "virtual" external device can be eliminated by using a <i>Template Switch</i>.
This method uses the binary sensor to set the switch state and provides for having different topics for the "on" and "off" actions.

{% raw %}

``` yaml
- switch:
  - platform: template
    switches:
      outsidedeco:
        unique_id: "outsidedecotmplsw"
        value_template: "{{ is_state('binary_sensor.OutsideDecoState', 'on') }}"
        turn_on:
          service: mqtt.publish
          data:
            topic: 'cmnd/OutsideDecorationsOn/POWER'
            payload: 'Run'
        turn_off:
          service: mqtt.publish
          data:
            topic: 'cmnd/OutsideDecorationsOff/POWER'
            payload: 'Run'
```
{% endraw %}

You can add additional switches as needed under "switches".

<!-- <h4 id="using-triggers-for-different-onoff-complex-actions-with-a-virtual-external-device">Using Triggers for Different On/Off Complex Actions with a Virtual External Device</h4> -->
#### Using Triggers for Different On/Off Complex Actions with a Virtual External Device

This one shows how to use standard ON/OFF triggers to execute actions more complex than just one macro each for ON and OFF. It uses a serial command to the <i>action plug-in</i> to run the macros, then a serial command to <i>custom plug-in</i> to control a candles object.
<br>
<br>
<p align="center">
  <img alt="HA_Indoor_Deco_Config" src="HA_Indoor_Deco_Config.png">
</p>
<!-- <h4  id="using-a-single-trigger-for-complex-actions-with-a-virtual-external-device">Using a Single Trigger for Complex Actions with a Virtual External Device</h4> -->
#### Using a Single Trigger for Complex Actions with a Virtual External Device

This one shows using a single trigger (independent of ON/OFF) to send a series of IR commands. 
<br>
<br>
<p align="center">
<img alt="HA_NBC_Config" src="HA_NBC_Config.png">
</p>
<br>
<br>
The compete trigger command (obscured in the screen shot) is:
<pre>
    action: ir transmit 28 1,ir transmit 114 1,wait for 500,
            ir transmit 116 1,wait for 500,ir transmit 110 1;
</pre>
In this example, IR is used, but with a virtual external device to allow for more complex command execution. Consequently, a switch should be added manually to your <i>configuration.yaml</i>. Here is what the switch code would look like
(HA Core 2022.6 and later):

{% raw %}
``` yaml
mqtt:
  switch:
    - name: "IR_HA_NBC"
      unique_id: IR_HA_NBC
      state_topic: "stat/HA_NBC/POWER"
      command_topic: "stat/HA_NBC/POWER"
      payload_on: "ON 1"
      payload_off: "Off 1"
      state_on: "ON"
      state_off: "Unknown"
      qos: 1
```
{% endraw %}
Version deprecated in HA Core 2022.6 (usable until removed in 2022.9):
{% raw %}
``` yaml
switch:
  - platform: mqtt
    unique_id: IR_HA_NBC
    name: "IR_HA_NBC"
    state_topic: "stat/HA_NBC/POWER"
    command_topic: "stat/HA_NBC/POWER"
    payload_on: "ON 1"
    payload_off: "Off 1"
    state_on: "ON"
    state_off: "Unknown"
    qos: 1
```
{% endraw %}
Next, create a button using the state_topic and payload_on in MQTT Publish service.

<!-- <h3 id="running-scheduled-and-periodic-events">Running Scheduled and Periodic Events</h3> -->
### Running Scheduled and Periodic Events
The MQTT plug-in provides for direct execution of Scheduled and Periodic Events, similar to Macros, by defining them in <b>Int Objects</b> configuration.
<br>
<br>
You also can execute a Scheduled Event or Periodic Event by using the trigger option in a virtual external device.
It may be useful to run these after being triggered by some Home Assistant event, in addition to what HomeVision's schedule is doing.
They also can be used simply as additional macro space, <i>especially if put in the "Disabled" state</i>, where they normally won't run via a schedule or period.
<br>
<br>
To do this, in a virtual external device's "Configure Device" screen, select "Trigger" and set the "Trigger:" field to something like this:

<pre>
    action: se run 3;
</pre>

which will run Scheduled Event #3.
<!-- <h3 id="running-other-objects">Running Other Objects</h3> -->
### Running Other Objects
The previous Tip was called out specifically because it can be used to add in macro capability for those who have run out of macro space.
However the idea can be extended to virtually all items in HomeVisionXL. Since the trigger simply sends whatever is there to HomeVisionXL's serial command processor, you can set triggers for anything the Action plug-in can do (or any other plug-in with its own defined serial commands), and, along with the trigger's capability to make run-time substitutions in a trigger string, Home Assistant may be able to trigger different things based on the payload sent.
In an extreme case, the trigger could be just "%M", in which case Home Assistant would put in the payload the complete trigger string to execute. See
<i>Triggers</i> in
<a href="MQTT_Client_Plug-in">MQTT Help</a>
for more details.
<!-- <h3 id="timers">Timers</h3> -->
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
<ul>
<li>Run MQTT Discovery for the desired timer. This creates two sensor entities. Use these in the GUI to show the Timer state and "current" timer value.</li>
<li>Create the input_select helper for the timer's control states.
(Configuration->Helpers->Add Helper->Select).
This is used to specify what timer command to send.
Use the following values:
</li></ul>
{% raw %}
``` yaml
    Options:
        Load
        Start
        Stop
        Clear
```
{% endraw %}
<ul>
<li>Create the input_text helper for the timer's set time. (Configuration->Helpers->Add Helper->Text).
This is used to specify what timer value to send.
Use the following values:
</li>
</ul>
{% raw %}
``` yaml
    Max: 12
    Min: 10
    Regex pattern: ^(?:(?:([01]?\\d|2[0-3]):)?([0-5]?\\d):)?([0-5]?\\d\\.)?([0-9][0-9]$
```
{% endraw %}

Once these are available, 
create a GUI using them.
There are probably many different ways to do this, but here is one.
<br>
<br>
<p align="center">
<img alt="HA_Outside_Deco_Config" src="HA Timer GUI.gif">
</p>
<br>
<br>
This GUI uses a grid card,
with the two timer entities, the two helpers and two buttons
(to get timer status and send commands to the timer).

Here is the yaml for the above implementation.
This was created using the GUI editor, which is a lot easier than writing in yaml, but is shown here to see the details, especially for the Update payload, which needs to be entered in the payload template in the GUI editor as shown in the yaml.
{% raw %}
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
{% endraw %}

<b>Note:</b> The timer's current time in the GUI does <i>not</i> update automatically, but only when receiving a "stat" update.
"stat" updates occur whenever a timer command (load, start,stop,clear or query - i.e., "?") is sent to the timer.

<!-- <h3 id="generic-homevision-actions">Generic Homevision Actions</h3> -->
### Generic Homevision Actions
This feature allows you to send a set of commands just like those allowed in an external device <i>trigger</i>.
Since this method is not supported in MQTT discovery, It would need to be used in configuration.yaml or in a GUI-based construction, like a button.

Instead of the method used in 
<a href="#using-a-single-trigger-for-complex-actions-with-a-virtual-external-device">Using a Single Trigger for Complex Actions with a Virtual External Device</a>,
Create a button and use
<pre>
    Topic:       cmnd/homevision/action
    Payload:     action: ir transmit 28 1,ir transmit 114 1,
                 wait for 500,ir transmit 116 1,wait for 500,
                 ir transmit 110 1;
</pre>
No "virtual" external device needed.

<!-- <h4 id="generic-homevision-actions-versus-virtual-devices">Generic HomeVision Actions Versus Virtual Devices</h4>-->
#### Generic HomeVision Actions Versus Virtual Devices
Which one is better? 
It depends on how you want to access the functionality that either method provides.
<ul>
<li>
If you want to execute complex actions (like in the above example) from more than one place ( e.g., Home Assistant, Node-Red), then it may make more sense to use a virtual device. Otherwise, the generic method would require the same sequence of actions to be duplicated at every source of the command. This could complicate managing any changes that might be needed as you would have to remember to update all occurrences of the action sequence.
</li>
<li>
Virtual devices also keep the action sequence information "closer" to the place where it is actually used (in HomeVision), potentially easier to manage.
</li>
<li>
If you are only concerned about access from one place (i.e., Home Assistant),
using the generic HomeVision action method may simplify implementation in both HomeVision and Home Assistant. No "extra" external device would be required in the MQTT plug-in. The command most likely could be implemented directly in the Home Assistant UI, with little to no manual changes in the configuration.yaml file.
A virtual device would usually require a switch entity in the configuration.yaml file.
</li>
</ul>
<!-- <h3 id="refreshing-homevision-objects">Refreshing HomeVision Objects</h3>-->
### Refreshing HomeVision Objects
There may be instances (like restarting Home Assistant) where the current status of HomeVision objects is not reflected by Home Assistant. The MQTT plug-in provides a special topic to force all listed objects to report their status: 
<pre>
        Full Topic                        Payload
    cmnd/homevision/<i>object_type</i>/POWER    empty or "?"
or
    cmnd/homevision/POWER                empty or "?"
</pre>

If you have this issue, you may want to consider adding a button to send this message, or create an automation to issue the command at an appropriate time.
<br>
<br>
Here's one possible implementation of an automation that is triggered 1 minute after Home Assistant starts, to make sure that the state of all HomeVision objects gets updated in Home Assistant.

The automation is probably most easily created using the Home Assistant Automation visual editor, but here is the resulting yaml:

{% raw %}
``` yaml
alias: Refresh HA for HVXL Objects at Restart
description: 1 Min after HA start, trigger status from HVXL.
trigger:
  - platform: homeassistant
    event: start
condition: []
action:
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - service: mqtt.publish
    data:
      topic: cmnd/homevision/POWER
      payload: '?'
mode: single
```
{% endraw %}

<!-- <h3 id="retain-option-for-objects">Retain Option for Objects</h3> -->
### Retain Option for Objects
You may also want to consider, as a possible alternative to the previous, whether to enable "retain" for objects that are tracked by Home Assistant.
This should allow Home Assistant to automatically pick up status via the MQTT broker's retained messages feature.
"Retain" is an option in the MQTT plug-in's <i>Configure Object</i> screen.
<br>
<br>
<b>DO NOT confuse</b> the above "retain" with the retain attribute in the Home Assistant discovery payload.
The retain in the Home Assistant discovery payload governs the retain setting of messages sent OUT by HomeAssistant, and is valid only for objects defined as switch, light or climate.