# How to Use Home Assistant Auto Discovery
{:.no_toc}

[Back to Projects](/index)

[Back to MQTT Index](/MQTT/MQTT_index)

* Overview
  * Built-in Method
    * Using the Built-in Method
  * Manual Method
  * Discovery Notes
  * HomeVision Object Discovery Details
    * Discovery Messages
      * X-10, Light Objects
      * Flags
      * Variables
      * Inputs
      * Outputs
      * IR
      * Macros, Scheduled Events, Periodic Events
      * Digital Temperature Sensors
      * Analog Inputs
      * Timers
      * HVAC
      * Device Class Note
{:toc}

# Overview
For systems where you want to expose a significant number of internal objects to Home Assistant, the MQTT plug-in provides an Auto Discovery feature that pushes discovery messages to Home Assistant, negating the need to enter each object's code into your <i>configuration.yaml</i>.
<br>
<br>
<b>Caution:</b>
Auto Discovery assumes that in MQTT Configuration "Settings", the "State Response" settings are all checked, and "Response uses OFF/ON" are all checked.
<i>Changing these settings will likely result in Discovered entities not working as expected.</i>
<br>
<br>
Auto Discovery does use the "Command Prefix", "State Prefix" and "Power Postfix" settings, and works with Object topics in any of the six standard topic templates: &lt;T&gt;, &gt;T&lt;, &lt;&gt;T, T&lt;&gt;, &gt;&lt;T, T&gt;&lt;.
## Built-in Method
The MQTT plug-in supports a User Interface for Discovery.
Because this is a feature that most users won't need, and for those who do use it, it is likely a one-time or seldom used feature, it is accessed by a normally "hidden" tab.
To activate the Discovery feature, open the MQTT configuration screen, hover your mouse anywhere on the screen, then press F4. You should see a new "Discovery" tab appear. Alternately pressing F4 will cause the Discovery tab to appear/disappear.
### Using the Built-in Method
* Select "Object Type". Select "All" to discover all objects (those that are in the "Int Objects" list, NOT all of your HomeVisionXL defined objects!)
The "Choose Object(s)" list is NOT populated if you select "All" here.
Note: If "All" is selected, only "flag" is included; "flag_b" is not.
* If you selected an object type and want to further limit discovery to a subset of those objects, click on each one you want (ctrl-click for selecting more than one.) Otherwise, select "All". If you click on "All" AND also select other objects, only the selected objects will be discovered, not "All".
* Select the discovery type: "Test", "Discover", UnDiscover".
* Check "NO ID in Name?" to not include the object ID in the entity name. Same as the "-noid" option in the manual method.
* Check "Underscore to space in Name?" to replace underscores with spaces in the entity name. Same as the "-nous" option in the manual method.
* Check "Retain Flag for HA published msgs?" to have Home Assistant set the retain flag for "command_topic" messages it sends. Same as the "-retain" option in the manual method.
* Check "Exclude from Device?" to NOT include the object's entity in Home Assistant in an object type device.
Same as the "-nodevice" option in the manual method.
* Click "Run" when all selections are complete. If you have the debug plug-in running, you will see messages for each object selected. This is particularly useful when "Test" is selected.

Discovery tab visibility as well as the selection of settings on the Discovery screen are **volatile**; they are not saved when the plug-in is shut down, unlike other plug-in settings.

## Manual Method

The MQTT Plug-in contains a public procedure to do discovery from withing other plug-ins. To use, the plug-in must have the following line:
<pre>
    hvImport haObjectDiscovery
</pre>
Syntax of discovery command:
<pre>
    hvObjectDiscovery {-noid} {-nous} {-nodevice} {-retain} <i>add</i> {<i>object_type</i> {<i>id1 id2 ...</i>}}
</pre>

* <i>-noid</i> is optional. If present, don't include the object ID (i.e.: A-10) at the beginning of the entity name. Default: include the id. The object ID is still used in <i>unique_id</i> and other identifiers to ensure unique entries there.
* <i>-nous</i> is optional. If present, replace underscores with spaces in the entity name. Default: include any underscores as-is.
* <i>-nodevice</i> is optional. If present, the object's entity in Home Assistant will NOT be included in an object type device. Default: Entity is included.
* <i>-retain</i> is optional. If present, Home Assistant will publish command_topic messages with the retain flag set. Default: retain flag is not set.
* <i>add</i> is required:
 <br>
 1 - for discovery,
 <br>
 0 - for object removal (sends a message asking for the object to be removed from the entity list.)
 <br>
 Any other number results in creating the message(s) and just displaying them in the debug plug-in, but NOT publishing.
 Good for testing that the procedure generates what you expect without actually sending anything.
* If <i>object_type</i> is not present, then it will discover (or remove) all objects (those that are in the "Int Objects" list, NOT all of your defined objects!).
Note: If "All" is selected, only "flag" is included; "flag_b" is not.
If you want to limit discovery or removal to one object type, put that type here. Valid built-in object_types are x10, light, var, flag, flag_b, input, output, analog, temp, ir, hvac, macro, se and pe.
Also works for other plug-ins that have discoverable devices and provide the proper interface to the Discovery procedures. (Caseta is one such plug-in.)
* If you want to further limit to only certain IDs within a type, list the IDs (the <b><i>numeric form</i></b>, NOT the letter/number form).


"flag_b" generates a flag as a <i>binary_sensor</i>. "flag" does a <i>switch</i>.
As a binary_sensor, the flag is "read-only". As a switch, it can be read and written.
You can run discovery on a flag for both types (separately of course). Discovery in this case will generate different <i>unique_IDs</i> and <i>entity IDs</i> for each type. Depending on <i>-noid</i>, names may be the same, but these can be changed in the Home Assistant UI if desired.
There may be a need to have the same flag appear as both (although I don't have a use case for it!)
<br>
<br>
For sensor and binary_sensor objects, if "dc:<i>device_class</i>" is found in the HomeVisionXL Description field of the var, flag (flag_b), input, analog input, or temp (DTS),
a "device_class" attribute will be created with <i>device_class</i> as its value. No validation is made to make sure <i>device_class</i> is a valid "device_class" for the object, and may produce unknown behavior in Home Assistant if invalid.
<br>
<br>
Examples:
<br>
<br>
   Discover all objects:
<pre>
       hvObjectDiscovery 1
</pre>
   Discover some x-10 (a2, B6, C15):
<pre>
       hvObjectDiscovery 1 x10 1 21 46
</pre>
   Show all discovery messages, but don't publish:
<pre>
       hvObjectDiscovery 2
</pre>
   Remove all objects:
<pre>
       hvObjectDiscovery 0
</pre>
   Remove flag 10:
<pre>
       hvObjectDiscovery 0 flag 10
</pre>
   Discover only flag objects (as switches):
<pre>
       hvObjectDiscovery 1 flag
</pre>
   Discover only flag objects (as binary_sensors):
<pre>
       hvObjectDiscovery 1 flag_b
</pre>
   Discover more than one but less than your total object types. No ID or underscores in Name. 
   Add a new command line for each type:
<pre>
      hvObjectDiscovery -noid -nous 1 flag 10
      hvObjectDiscovery -noid -nous 1 x10 1 21 46
</pre>
## Discovery Notes

* Entities are created using the object's "Object Name", not its "topic".
* If neither "NO ID in Name?" (or <i>-noid</i>) nor "Underscore to space in Name?" (or <i>-nous</i>) are checked, the entity "name" is constructed from the object's "ID" and "Object Name".
<br>
Example: 
<pre>
    B-12 Garage_Back_Door
</pre>
The entity id will be:
<pre>
    light.b-12_garage_back_door
</pre>
* If "NO ID in Name?" is checked, the entity "name" and id are constructed from the object's "Object Name" only.
* If "Underscore to space in Name?" is checked, underscores are replaced by spaces in the entity "name".
Entity ids always replace spaces in the name with underscores, regardless of this setting.
<br>
Example ("NO ID in Name?"and "Underscore to space in Name?" both selected): 
<pre>
    Garage Back Door
</pre>
The entity id will be:
<pre>
    light.garage_back_door
</pre>
* If "Exclude from Device?" is NOT checked (or -nodevice is not present), for each object type, MQTT Discovery creates a separate "device" that groups all entities created for that type.
Device names will be "HomeVision {Object_type}". 
If "Exclude from Device?" is checked (or -nodevice is present), then the the entity is not included.
* Other plug-ins that support objects that can be discovered (like the caseta plug-in) can be accessed from the MQTT "Discovery" tab.
The plug-in must create two procedures: a "getDiscovDevices" procedure that returns a dict with a list of devices used to populate the "Object Type" and "Choose Objects" lists in the MQTT "Discovery" tab,
and a "haObjectDiscovery" procedure which is called when "Run Discovery" is clicked and creates the discovery message to Home Assistant for each object to be discovered.
The "haObjectDiscovery" procedure typically will call another procedure to process each individual object.
<br>
<br>
If a plug-in needs to use configuration values from the MQTT plug-in, use the "getDiscovConfig" procedure.
"getDiscovConfig" returns a dict with the following:
<pre>
    clientID
    cmndPrefix 
    statPrefix
    telePrefix
    pwrPostfix
    rstPostfix
    LWTPostfix 
</pre>
"clientID" can be used to create unique ids if necessary.
<br>
<br>
Example (as provided by the Caseta Plug-in): 
<pre>
    hvPublic getDiscovDevices
    proc getDiscovDevices {} {

        mk::loop row cfg.devices {
            lassign [mk::get $row id name] id name
            dict set caseta caseta $id [list "CA-$id" $name]
        }
        return $caseta
    }

    hvPublic haObjectDiscovery
    proc haObjectDiscovery {args} {
        ...
        sendDiscovery $noid $nous $retain $nodevice $add $id $name $topic $obj_type $dev_type $model $manuf
        ...
    }
    
    hvImport getDiscovConfig
    hvImport mqttComm
    proc sendDiscovery {args} {
        ...
        # get prefix/postfix, etc. info from MQTT, if needed.
        set mqttcfg {*}[getDiscovConfig]
        ...
        # create HA discovery topic and message
        ...
        # send discovery
        mqttComm -retain pub $hadiscoverytopic $message
        ...
    }
</pre>

## HomeVision Object Discovery Details

An MQTT discovery message is sent in the form:
<pre>
    homeassistant/{entity type}/{id}/config {payload}
</pre>
{entity type} is a Home Assistant defined entity.
<br>
{id} is a unique id constructed from the MQTT plug-in's client ID and the object's ID. This assures a unique ID even if for some (probably very unlikely) reason another HomeVisionXL MQTT is running in your system.
<br>
{payload} is the required Home Assistant definition of elements for the entity.
<br>
<br>
For each object type, MQTT Discovery can create a separate "device" that groups all entities created for that type.
 Device names will be "HomeVision {Object_type}"
<br>
<br>
The following entity types and devices are created according to the Object type:
<br>
<br>
<table>
 <tr>
  <th>HomeVision Object</th>
  <th>Entity Type</th>
  <th>Device Name</th>
 </tr>
 <tr>
  <td align="center">X-10, Light
  Dimming capable</td>
  <td align="center">light</td>
  <td>HomeVisionXL X10
  <br>HomeVisionXL Light</td>
 </tr>
 <tr>
  <td align="center">X-10:
  Appliance or switch module</td>
  <td align="center">switch</td>
  <td>HomeVisionXL X10</td>
 </tr>
 <tr>
  <td align="center">Flag</td>
  <td align="center">switch</td>
  <td>HomeVisionXL Flag</td>
 </tr>
 <tr>
  <td align="center">Flag_b</td>
  <td align="center">binary_sensor</td>
  <td>HomeVisionXL Flag_b</td>
 </tr>
 <tr>
  <td align="center">Variable</td>
  <td align="center">sensor</td>
  <td>HomeVisionXL Var</td>
 </tr>
 <tr>
  <td align="center">IR</td>
  <td align="center">switch</td>
  <td>HomeVisionXL Ir</td>
 </tr>
 <tr>
  <td align="center">Input</td>
  <td align="center">binary_sensor</td>
  <td>HomeVisionXL Input</td>
 </tr>
 <tr>
  <td align="center">Output</td>
  <td align="center">switch</td>
  <td>HomeVisionXL Output</td>
 </tr>
 <tr>
  <td align="center">Analog Input</td>
  <td align="center">sensor</td>
  <td>HomeVisionXL Analog</td>
 </tr>
 <tr>
  <td align="center">DTS (temp)</td>
  <td align="center">sensor</td>
  <td>HomeVisionXL Temp</td>
 </tr>
 <tr>
  <td align="center">HVAC</td>
  <td align="center">climate</td>
  <td>HomeVisionXL Hvac</td>
 </tr>
 <tr>
  <td align="center">Macro</td>
  <td align="center">switch</td>
  <td>HomeVisionXL Macro</td>
 </tr>
 <tr>
  <td align="center">Scheduled Event</td>
  <td align="center">switch</td>
  <td>HomeVisionXL Se</td>
 </tr>
 <tr>
  <td align="center">Periodic Event</td>
  <td align="center">switch</td>
  <td>HomeVisionXL Pe</td>
 </tr>
 <tr>
  <td align="center">Timer</td>
  <td align="center">sensor</td>
  <td>HomeVisionXL Timer</td>
 </tr>
 </table>

### Discovery Messages
Here are examples of what is sent to Home Assistant during discovery.
<br>
The JSON formatted payloads are expanded for readability.
<br>
Note: Actual discovery payloads may use abbreviated configuration variable names.
####  X-10, Light Objects
Defined as "light" entities.
{% raw %}
<pre>
homeassistant/light/HVXLb1d0d912ed315aad_C-9/config
{
        "unique_id": "HVXLb1d0d912ed315aad_C-9",
        "name": "C-9 Kitchen_Sink",
        "state_topic": "stat/KitchenSink/RESULT",
        "command_topic": "cmnd/KitchenSink/POWER",
        "payload_on": "ON",
        "payload_off": "OFF",
        "state_value_template": "{{ value_json.POWER }}",
        "brightness_command_topic": "cmnd/KitchenSink/POWER",
        "brightness_state_topic": "stat/KitchenSink/RESULT",
        "brightness_scale": 100,
        "brightness_value_template": "{{ value_json.Dimmer }}",
        "on_command_type": "brightness",
        "qos": 1,
        "retain": false,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_X10"
                ],
                "name": "HomeVision X10",
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
}
</pre>
{% endraw %}
If a device is defined as a appliance or switch module, it will be set up as a switch entity instead.
{% raw %}
<pre>
homeassistant/switch/HVXLb1d0d912ed315aad_C-11/config
{
        "unique_id": "HVXLb1d0d912ed315aad_C-11",
        "name": "C-9 Kitchen_Sink",
        "state_topic": "stat/Coffee/RESULT",
        "command_topic": "cmnd/Coffee/POWER",
        "payload_on": "ON",
        "payload_off": "OFF",
        "value_template": "{{ value_json.POWER }}",
        "qos": 1,
        "retain": false,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_X10"
                ],
                "name": "HomeVision X10",
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
}
</pre>
{% endraw %}
####  Flags
Defined as "switch" entities. In this case, the STATE RESULT payload uses "STATE" instead if "POWER". 
{% raw %}
<pre>
homeassistant/switch/HVXLb1d0d912ed315aad_FL-18/config
{
        "unique_id": "HVXLb1d0d912ed315aad_FL-18",
        "name": "FL-18 Trace_1",
        "state_topic": "stat/Trace1/RESULT",
        "value_template": "{{ value_json.STATE }}",
        "command_topic": "cmnd/Trace1/POWER",
        "payload_on": "Set",
        "payload_off": "Clear",
        "state_on": "Set",
        "state_off": "Clear",
        "qos": 1,
        "retain": false,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_Flag"
                ],
                "name": "HomeVision Flag",
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
}
</pre>
{% endraw %}
Alternatively, a flag can be defined as a binary_sensor (use "flag_b" as the object type). The flag would be read-only from the perspective of Home Assistant.
"device_class" is set if found in the input's "Description" field. Must be a valid sensor device class. See Note.
{% raw %}
<pre>
homeassistant/binary_sensor/HVXLb1d0d912ed315aad_FL-26b/config
{
        "unique_id": "HVXLb1d0d912ed315aad_FL-26b",
        "name": "FL-26b Gar_Dr_Report_OnTime",
        "state_topic": "stat/GarDrReportOnTime/RESULT",
        "value_template": "{{ value_json.STATE }}",
        "payload_on": "Set",
        "payload_off": "Clear"
        "device_class: "garage_door",
        "qos": 1,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_Flag_b"
                ],
                "name": "HomeVision Flag_b",
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
}
</pre>
{% endraw %}
#### Variables
Defined as "sensor" entities. 
"device_class" is set if found in the input's "Description" field. Must be a valid sensor device class. See Note.
See <i>"Variable Options"</i> in the <i>"Tips"</i> section of

[[Tips for interfacing HomeVision with Home Assistant.|Tips-for-Interfacing-HomeVision-with-Home-Assistant]]
<!--
<a href="HomeVision and Home Assistant.html">Tips for interfacing HomeVision with Home Assistant.</a>
-->
for other uses of variables.
{% raw %}
<pre>
homeassistant/sensor/HVXLb1d0d912ed315aad_VA-145/config
{
        "unique_id": "HVXLb1d0d912ed315aad_VA-145",
        "name": "VA-145 Battery Level",
        "state_topic": "stat/BatteryLevel/RESULT",
        "value_template": "{{ value_json.STATE }}",
        "device_class": "battery",
        "qos": 1,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_Var"
                ],
                "name": "HomeVision Var,
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
 }
</pre>
{% endraw %}
#### Inputs
Defined as "binary Sensor" entities. "payload_on" and "payload_off" are defined as the text in HomeVisionXL's <i>Input Port Summary's</i> "Low State Label" and "High State Label", respectively.  "device_class" is set if found in the input's "Description" field. Must be a valid binary_sensor device class. See Note.
Binary sensors do not have a "retain" option.
{% raw %}
<pre>
homeassistant/binary_sensor/HVXLb1d0d912ed315aad_IB-5/config
{
        "unique_id": "HVXLb1d0d912ed315aad_IB-5",
        "name": "IB-5 Door1",
        "state_topic": "stat/Door1/RESULT",
        "value_template": "{{ value_json.STATE }}",
        "payload_on": "Open",
        "payload_off": "Closed",
        "device_class": "garage_door",
        "qos": 1,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_Input"
                ],
                "name": "HomeVision Input",
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
}
</pre>
{% endraw %}
#### Outputs
Defined as "switch" entities. "state_on" and "state_off" are defined as the text in HomeVisionXL's <i>Output Port Summary's</i> "High State Label" and "Low State Label", respectively. Note: this is opposite of inputs!
{% raw %}
<pre>
homeassistant/switch/HVXLb1d0d912ed315aad_OA-3/config
{
        "unique_id": "HVXLb1d0d912ed315aad_OA-3",
        "name": "OA-3 Sprinkler_Enable",
        "state_topic": "stat/SprinklerEnable/RESULT",
        "value_template": "{{ value_json.STATE }}",
        "command_topic": "cmnd/SprinklerEnable/POWER",
        "payload_on": "high",
        "payload_off": "low",
        "state_on": "Enabled",
        "state_off": "Disabled",
        "qos": 1,
        "retain": false,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_Output"
                ],
                "name": "HomeVision Output",
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
}
</pre>
{% endraw %}
#### IR
Defined as "switch" entities. Number of repeats is set to 1. 
Because IR objects have no real "on/off" state, IR switches will show as Off, and when clicked to On, will shortly return back to the Off position, since the state message payload will not match.
{% raw %}
<pre>
homeassistant/switch/HVXLb1d0d912ed315aad_IR-152/config
{
        "unique_id": "HVXLb1d0d912ed315aad_IR-152",
        "name": "IR-152 Candles_on",
        "state_topic": "stat/Candleson/RESULT",
        "value_template": "{{ value_json.STATE }}",
        "command_topic": "cmnd/Candleson/POWER",
        "payload_on": "ON 1",
        "payload_off": "Off 1",
        "state_on": "ON",
        "state_off": "Unknown",
        "qos": 1,
        "retain": false,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_Ir"
                ],
                "name": "HomeVision Ir",
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
}
</pre>
{% endraw %}
IR entities can also be used as Buttons, which makes a little more sense UI-wise. These can be added manually in the UI, which will create a button object like this:
{% raw %}
<pre>
      - type: button
        tap_action:
          action: toggle
        entity: switch.ir_119_comcast_nav_up
        show_name: false
        name: UP
        show_icon: true
        icon: 'mdi:arrow-up-box'
        icon_height: 50px
        hold_action:
          action: none
</pre>
{% endraw %}
#### Macros, Scheduled Events, Periodic Events
Defined as "switch" entities. 
Because these objects have no real "on/off" state, switches will show as Off, and when clicked to On, will shortly return back to the Off position, since the state message payload will not match.
{% raw %}
<pre>
homeassistant/switch/HVXLb1d0d912ed315aad_MA-10/config
{
        "unique_id": "HVXLb1d0d912ed315aad_MA-10",
        "name": "MA-10 Good Night",
        "state_topic": "stat/GoodNight/RESULT",
        "value_template": "{{ value_json.STATE }}",
        "command_topic": "cmnd/GoodNight/POWER",
        "payload_on": "Run",
        "payload_off": "Off",
        "state_on": "ON",
        "state_off": "Unknown",
        "qos": 1,
        "retain": false,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_Macro"
                ],
                "name": "HomeVision Macro",
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
}
</pre>
{% endraw %}
Like IR entities, Macros, Scheduled Events and Periodic Events can also be used as Buttons. See IR above for details.
#### Digital Temperature Sensors
Defined as "sensor" entities. "device_class" is set if found in the input's "Description" field. If no device class is found in the Description, then "device_class" is set to "temperature". Must be a valid sensor device class. See Note. If the "device_class" word is followed by a "-F" or "-C", "unit_of_measurement" is set to "F" or "C" accordingly. Otherwise it is set to the value specified in HomeVisionXL's Settings->Temperature Scale.
Sensors do not have a "retain" option.
{% raw %}
<pre>
homeassistant/sensor/HVXLb1d0d912ed315aad_TE-0/config
{
        "unique_id": "HVXLb1d0d912ed315aad_TE-0",
        "name": "TE-0 Digital_temp_sensor_0",
        "state_topic": "stat/Digitaltempsensor0/RESULT",
        "value_template": "{{ value_json.STATE }}",
        "unit_of_measurement": "F",
        "device_class": "temperature",
        "qos": 1,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_Temp"
                ],
                "name": "HomeVision Temp",
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
}
</pre>
{% endraw %}
#### Analog Inputs
Defined as "sensor" entities. 
"device_class" is set if found in the input's "Description" field. Must be a valid sensor device class. See Note.
{% raw %}
<pre>
homeassistant/sensor/HVXLb1d0d912ed315aad_AN-0/config
{
        "unique_id": "HVXLb1d0d912ed315aad_AN-0",
        "name": "AN-0 Analog_Input_0",
        "state_topic": "stat/AnalogInput0/RESULT",
        "value_template": "{{ value_json.STATE }}",
        "device_class": "battery",
        "qos": 1,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_Analog"
                ],
                "name": "HomeVision Analog",
                "model": "HomeVision",
                "manufacturer": "CustomSolutions",
        }
 }
</pre>
{% endraw %}
#### Timers
Defined as <i>two</i> "sensor" entities,
one for the timer's state (Running, Stopped, Ringing)
and one for the current time.
Time is reported in "HHH:MM:SS.hh" format.
The text "state" is appended to the name of the timer for the state entity,
and "time" is appended to the name of the timer for the time entity.
"device_class" is set if found in the input's "Description" field.
Must be a valid sensor device class,
although "device_class" may not make sense for timers. See Note.
{% raw %}
<pre>
homeassistant/sensor/HVXLb1d0d912ed315aad_TI_8/config
{
          "unique_id":"HVXLb1d0d912ed315aad_TI_8",
          "name":"Wash Timer state",
          "state_topic":"stat/WashTimer/RESULT",
          "value_template":"{{ value_json.STATE }}"
          "qos":1,
          "device": {
                  "identifiers": [
                          "HVXLb1d0d912ed315aad_Timer"
                  ],
                  "name":"HomeVisionXL Timer",
                  "model": "HomeVision",
                  "manufacturer": "CustomSolutions",
          }
}
</pre>
{% endraw %}

{% raw %}
<pre>
homeassistant/sensor/HVXLb1d0d912ed315aad_TI_8t/config
{
          "uniq_id":"HVXLb1d0d912ed315aad_TI_8t",
          "name":"Wash Timer time",
          "state_topic":"stat/WashTimer/RESULT",
          "value_template":"{{ value_json.TIME }}"}!
          "qos":1,
          "device": {
                  "identifiers": [
                          "HVXLb1d0d912ed315aad_Timer"
                  ],
                  "name":"HomeVisionXL Timer",
                  "model": "HomeVision",
                  "manufacturer": "CustomSolutions",
          }
}
</pre>
{% endraw %}
#### HVAC
Defined as "climate" entities. This is for HVAC native support for RCS TX-15 type thermostats. Assumes F.
While the HVAC returns modes in Title case, the climate entity requires them in lower case. The mode_, hold_, and fan_ state_templates convert them to lower case. When Home Assistant sends commands to the HVAC, it sends in lower case, but the MQTT plug-in is case-insensitive on receive.
<br>
<br>
When HVAC status is sent from HomeVisionXL, "Fan" reports either "Auto" or "On". The "Auto" here conflicts with the mode ("HVAC") of "Auto". Since the fan keywords used to set the fan mode are "fanauto" and "fanon", these are defined as the <i>Fan_modes</i> items, and the <i>fan_mode_state_template</i> prepends "fan" to what it receives (either "Auto" or "On").
{% raw %}
<pre>
homeassistant/climate/HVXLb1d0d912ed315aad_HV-1/config
{
        "unique_id": "HVXLb1d0d912ed315aad_HV-1",
        "name": "HV-1 Zone1",
        "modes": [
                "auto",
                "off",
                "cool",
                "heat"
        ],
        "fan_modes": [
                "fanauto",
                "fanon"
        ],
        "hold_modes": [
                "run",
                "hold"
        ],
        "mode_command_topic": "cmnd/Zone1/POWER",
        "hold_command_topic": "cmnd/Zone1/POWER",
        "temperature_command_topic": "cmnd/Zone1/POWER",
        "fan_mode_command_topic": "cmnd/Zone1/POWER",
        "current_temperature_topic": "stat/Zone1/RESULT",
        "current_temperature_template": "{{ value_json.Temperature }}",
        "mode_state_topic": "stat/Zone1/RESULT",
        "mode_state_template": "{{ value_json.HVAC|lower }}",
        "hold_state_topic": "stat/Zone1/RESULT",
        "hold_state_template": "{{ value_json.Control|lower }}",
        "temperature_state_topic": "stat/Zone1/RESULT",
        "temperature_state_template": "{{ value_json.Setpoint }}",
        "fan_mode_state_topic": "stat/Zone1/RESULT",
        "fan_mode_state_template": "{{ \\\"fan\\\" + value_json.Fan|lower }}",
        "temperature_unit": "F",
        "precision": 1.0,
        "max_temp": 99,
        "min_temp": 55,
        "qos": 1,
        "retain": false,
        "device": {
                "identifiers": [
                        "HVXLb1d0d912ed315aad_Hvac"
                ],
                "name": "HomeVision Hvac",
                "model": "TX-15",
                "manufacturer": "RCS",
        }
}
</pre>
{% endraw %}

#### Device Class Note
For sensors and binary sensors, "device_class" can be set by placing the following somewhere in the Object's Description field in HomeVisionXL.
(Other text is allowed in the Description as well.)
<pre>
   dc:<i>device_class</i>
</pre>
where <i>device_class</i> is a valid device class name for the sensor or binary sensor. No checking is done to make sure the string is a valid device class. 
