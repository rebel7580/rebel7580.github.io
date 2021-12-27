<!-- $Revision: 1.29 $ -->
<!-- $Date: 2021/07/22 21:55:08 $ -->
# MQTT Client Plug-in Help
{:.no_toc}

[Back to Projects Index](/index)

[Back to MQTT Index](/MQTT/MQTT_index)

  * Overview
    * Supported MQTT Topics
      * Standard and Custom Topics
      * Standard Command Topic
      * Standard State Topic
      * RESULT State Topic
      * Last Will and Testament Topic
      * Special "homevision" Topic
      * Special Counting Payload
    * Configuring Devices
      * Ext Devices Tab
      * Int Objects Tab
      * Settings Tab
    * Responding to External Device State Changes
    * Controlling Devices
      * Device/Object Display Area
      * Serial Control
      * NetIO
      * Sending Generic MQTT Messages
      * Custom Processing of Received Messages
        * Custom Commands
        * Triggers
      * mqttComm - Sending/Receiving MQTT Messages from/to Another Plug-in
      * Other Public Procedures Supplied/Called by the MQTT Plug-in
        * topicTemplate
        * mqttLog
        * mqttReady
      * MQTT Discovery for Home Assistant
{:toc}

## Overview
The MQTT Client Plug-in provides a client interface to MQTT for HomeVision.
Its main purposes are to control MQTT enabled devices via the HomeVision Schedule or NetIO and
to control HomeVision objects by MQTT sources.
The MQTT interface has three distinct functions:
* For "external" MQTT-enabled devices (e.g., Sonoff switches with Tasmota SW),
the MQTT Plug-in acts as an MQTT controller.
The plug-in PUBLISHES *command* topics to these devices
to control them
and SUBSCRIBES to *status* topics from these devices to track their state changes.
Changes are tracked by Actions such as setting Flags or Variables and running Macros based on that status.
* For "internal" objects defined in HomeVision
(such as X-10 modules, flags, inputs, etc.),
the MQTT Plug-in acts as a "proxy" for them, essentially making them appear to be MQTT-enabled.
For each selected object, the plug-in 
SUBSCRIBES to a *command* topic
so it can be controlled
and
PUBLISHES a *status* topic when it changes state.
* Generic MQTT topics can be sent from the HomeVision schedule via serial commands, from NetIO, or from custom plug-ins, independent of any configured devices.


While the client plug-in is designed to work easily with Tasmota based devices, using a similar topic and LWT structure,
other devices that follow different topic structures likely can be accommodated as well.
There is a lot of flexibility in the plug-in allowing support for many different situations.
<br>
<br>
**Note: To perform actions on internal objects, this plug-in uses the Actions Plug-in. The Actions Plug-in must be enabled to control internal objects.**
##  Supported MQTT Topics

### Standard and Custom Topics
Topics can be assigned to devices either as abbreviated ("Standard") topics or full topics.
Standard topics follow the Tasmota structure, so only the unique sub-topic portion need be entered. Default standard topics are indicated by enclosing the sub-topic with "<" and ">".
If a topic starts with a "<", the appropriate prefix is automatically added.
If a topic ends with a ">", the appropriate postfix is automatically added.
A ">" may be followed by optional index digits.
Prefixes and postfixes are defined in the **Settings Tab**.
Most of the examples to follow assume use of the defaults.
<br>
<br>
To provide additional flexibility in defining a system's topic structure, standard topics can have the prefix and postfix indicators in any position relative to the sub-topic.
The following are possible (where "T" is the sub-topic):
```
   Template        Example
      <T>       cmnd/T/POWER    --> Tasmota standard and MQTT plug-in default
      T<>       T/cmnd/POWER    
      <>T       cmnd/POWER/T
      >T<       POWER/T/cmnd
      T><       T/POWER/cmnd
      ><T       POWER/cmnd/T
```
To accommodate certain Tasmota devices that have multiple relays or switches, one or more digits can follow the ">" postfix index to specify the relay or switch.
For example, an external device of this type that sends out state reports like this:
```
    stat/FloorLamp/POWER2 ON
```
should be set up with this topic structure:
```
    <FloorLamp>2
```
Devices that send out reports for several "relays" should have separate entries in the **Ext Devices Tab** for each "relay".
<br>
<br>
Note 1:
While Tasmota indices are in the range of "1" through "32",
the plug-in allows any number of digits.
When using one of the templates where the topic follows a postfix with a relay index, care should be taken to avoid topics that start with a number, as the plug-in won't be able to tell where the index ends and the topic begins.
<br>
<br>Note 2:
Tasmota considers "POWER1" and "POWER" as the same and interchangeable.
However, the plug-in treats them as different, meaning "POWER" won't match a full topic containing "POWER1".
<br>
<br>
If a topic has only a "<" OR a ">", then only the appropriate
prefix OR postfix is automatically added.
Otherwise, the topic is used as-is without any additional portions prepended/appended to it.
Thus by not putting "<" and ">" in a topic,
any topic structure can be used.
However, the plug-in does not have built-in processing when receiving MQTT topics that don't follow the standard prefixes and postfixes. In these cases custom procedures need to be provided. See **Custom Processing of Received Messages**.
<br>
<br>
Most of the examples in the rest of this Help follow the Tasmota standard.

### Standard Command Topic
The following command topic is supported (using default prefixes and postfixes) with various payloads:
```
        Full Topic     Payload
    cmnd/*topic*/POWER  ON
    cmnd/*topic*/POWER  ON *level**
    cmnd/*topic*/POWER  *level**
    cmnd/*topic*/POWER  OFF
    cmnd/*topic*/POWER  TOGGLE
    cmnd/*topic*/POWER  empty or "?"** 
```
*topic* is defined during device configuration.
<br>Payloads are case-insensitive for incoming messages.<br>
*level* is expressed as a percentage, 0-100.
For X-10 devices, it  is converted to the nearest discrete level supported.
Any status reports back will have the discrete level.
<br>
** "?" for those external entities that don't allow an empty payload.
<br>
<br>
The plug-in *publishes* command topics to external devices.
Usually, after receiving one of these commands, an external device will publish its state.
The plug-in receives this report by *subscribing* to the state topic. (See next.)
An empty payload requests the device to publish its state without changing the state of the device.
<br>
<br>
HomeVision objects *subscribe* to command topics so that external entities can control them.
For a full list of Actions based on the received topic and payload, see 
<!-- <a href="MQTT_Actions_int.html">Object Actions</a>
-->
[[Help: Object Actions|Help:-Object-Actions]]

### Standard State Topic
For each external device, the plug-in *subscribes* to the following topic (assuming "<*topic*> in the "Topic" field):
```
        Full Topic                     Payload
    stat/*topic*/POWER*x*           OFF/ON/ON *level*/*level*
```
When a POWER topic is received, the plug-in looks for a message of "on" or "off" and takes action according to the device's settings in the **Ext Devices Tab**.
For a full list of Actions based on the received topic and payload, see
<!-- <a href="MQTT_Actions_ext.html">External Device Actions</a> -->
[[Help: External Device Actions|Help:-External-Device-Actions]]
.
<br>
<br>
When an internal object changes state, the plug-in will *publish* a state message to indicate the object's new state.
### RESULT State Topic
Certain devices, i.e., those running Tasmota software, usually produce both at POWER state message and a RESULT state message with the POWER info in a JSON formatted payload.
In fact, RESULT state messages can be sent from the device when many other actions occur.
<br>
<br>
To receive such messages, they must explicitly be entered as a device in the **Ext Devices Tab** with a *custom command* defined to process the message.
```
          Topic                         Payload
    stat/*topic*/RESULT               a JSON string
```
### Last Will and Testament Topic
For each external device with a *standard topic*
that has "Subscribe to Last Will and Testament" checked,
the plug-in automatically *subscribes* to
a *Last Will and Testament* topic as follows:
```
        Full Topic     
      tele/*topic*/LWT
```
If received, the plug-in looks for a message of "online" or "offline" and attempt to display this message in the **Ext Devices Tab**.
<br>
<br>
If the topic is not in the form of a standard topic,
the "Subscribe to Last Will and Testament" checkbox has no effect.
<br>
<br>
LWT is not supported for internal objects.
### Special "homevision" Topic
The MQTT plug-in automatically subscribes to:
```
        Full Topic
    cmnd/homevision/#
```
When a message is received in the form:
```
        Full Topic                        Payload
    cmnd/homevision/*object_type*/POWER    empty or "?"
    cmnd/homevision/POWER               empty or "?"
```
for each defined object matching *object_type*
(one of
x10, light, var, flag, hvac, temp, analog, input or output)
or for all object types if not specified, the plug-in will *publish* a state message for each object to indicate its current state.
*Only those objects specified in the **Int Objects Tab** are reported*.
This feature can be used by an MQTT entity to quickly sync up with HomeVision object states.
<br>
<br>
The only valid Payloads are empty or "?".
This topic cannot be used to *control* objects.
### Special Counting Payload
The MQTT plug-in has special handling to provide a "counting" feature.
Certain MQTT messages will *increment* selected variable(s) each time the message is received. To activate this feature,
create either an external device or internal object with either one or two variables selected.
Then, in the "Count Payload Text" of the *Settings Tab*, enter the text that will indicate "Count".
For external devices, receiving a "stat" or "tele" "POWER" message that has its first word in the payload matching "Count Payload Text" will cause the selected variable(s) to increment.
For internal objects, a "cmnd" "Power" message will do the same.
<br>
<br>
For example, if "Count Payload Text" is set to "CYCLE", and an internal object is created with Var-100 and Var-101 assigned to it but re-named "Watchdog",
then when the following is received:
```
    cmnd/Watchdog/POWER CYCLE
```
Var-100 and Var-101 are incremented.
<br>
<br>
Note: Other messages that would set variables assigned to this topic will continue to do so.
This can be used to advantage; sending a "cmnd" "POWER" message with a payload of "0" would "reset" the variables so subsequent "Count" messages would start from zero.
## Configuring Devices
### Ext Devices Tab
This tab contains a list of supported external devices.
*Device Name* is the name of the device for use by serial and NetIO commands.
It must be unique among both external and internal device names.
See below for Name rules.
*Topic* is the topic used for publishing and subscribing.
See below for Topic rules.
* The *Flag/Var* column shows the Flag (FL-#) or Variable (VA-#) assigned to the device.
If none is assigned, a "-" will show.
* The *Macro* column shows the Macro(s) assigned to the device in the form {on macro#}/{off macro#}.
If no macro is assigned, a "-" will show. 
* The *State* column will usually show "On" or "Off", depending on the reported state of the device.
If the payload was an number, its value, potentially masked and/or truncated, will be displayed.
When the state has not (yet) been reported, a "-" will show.
* For each device, its row will be displayed in RED text if the device is reported as "off line" by an LWT message.
However, if an "on Line" LWT or a POWER state message is received,
the row will display in BLACK text.
* Rows can be sorted in ascending or descending order by *Device Name* or *Topic* by clicking on the column header. 

*New/Edit/Delete External Devices*
* To enter a new device, click the "New" button, and start with the *Topic*.
  * Topics are case-sensitive!
  * A "Standard" topic, for which prefix and postfix substitution is performed (see **Settings Tab**),
is indicated by enclosing it in "<" and ">"
(or one of the variations mentioned previously).
E.g., "<*topic*>".
*This should be the default method for specifying topics.*
To help with this, a new device will have "<>" already inserted in the topic field.
(Delete or modify as needed.)
  * Topics can contain multiple levels, indicated by "/", but should not start or end with a "/".
  * A topic with *default* processing of received messages should not contain "#" or "+" MQTT wildcards, since it is used both to subscribe and publish and wildcards are not allowed in published topics.
  * A topic with *custom* processing of received messages MAY contain "#" or "+" MQTT wildcards, since it will only be subscribed to.
Use of wildcards should adhere to the MQTT standards.
However, care should be taken when using them. It's probably a bad idea to have "#" as a topic, as it will subscribe to EVERYTHING!
  *Device Names*, not topics, are used by NetIO and serial commands to publish commands to MQTT devices.
If the "topic" is simple and descriptive, it can be copied to the *Name* field.
If the "topic" is multi-level, *Name* must be simpler.
*Name* cannot contain "<", ">", "/" or spaces.
Alphanumeric and the underscore are the only allowed characters.
*Device Name* cannot be "pub", "sub" or "unsub"; these are reserved keywords for sending generic MQTT messages.
  * There is some validation of the *Topic* and *Device Name* fields to enforce the above rules and to avoid name duplication, but it may not be perfect.
  * For each device, an HV Flag or Variable can be assigned. 
Assigning a Flag or Variable is optional.
There is no checking to make sure a Flag or Variable is not used more than once.
  * If a variable is selected, two additional options are available.
    * "Use Variable as Flag" treats the variable as a flag with its value being either a "0" (off) or "1" (on). If a value is received instead of "on" or "off", if it is "odd" it represents "on" and if "even" it represents "off". The variable will be set  to "0" or "1" accordingly.
    * If "Use Variable as Flag" is NOT selected, then "Use Two Variables"
can be selected.
If it is, any value received in a stat message will be written to two variables as a 2-byte number. I.e., the LSB will be written to the specified Variable, and the MSB written to the specified Variable + 1.
If the option is not selected, the received value is written as a 1-byte number to the specified Variable.
<br>
<br>
Note: When "Use Variable as Flag" is not selected, Payloads with "on" or "off" do not cause any Actions, except for updating the "State" display.

  * Macros can be assigned for "On" and "Off" states.
The same macro can be assigned to both states, or different macros can be assigned.
A macro can be assigned to just one of the states, leaving the other set to "None".
  * To have custom processing of received messages or triggers for this topic, select the appropriate mode and enter command/triggers.
See **Custom Processing of Received Messages** for details.
  * Check "Subscribe to Last Will and Testament" to subscribe standard topics to LWT.
If the topic is not in the form of a standard topic,
this selection has no effect.
  * Choose whether to log MQTT messages sent or received by this device.
  * Click "OK" to save edits, or "Cancel" to discard them.

* Devices can be edited by clicking the "Edit" button.
This brings up the same window used for adding a new device.
* Use the "Delete" button to delete a device.
When a device is deleted, the plug-in unsubscribes to any topics related to that device.
* When "Done" is clicked, devices with standard topics are subscribed to their state and LWT full topics.
Devices with custom topics are subscribed only to the topic as-is.
*Click "Done" for changes to become effective!*


### Int Objects Tab
This tab contains a list of supported internal objects.
These can be any of: X-10, Custom Lights, Flags, Variables, Inputs, Outputs, IR, Digital Temperature Sensors, Analog Inputs, or HVAC.
Add only those objects that need to be visible to or acted on by the MQTT network.
* *ID* is the internal object ID.
X-10 object IDs show with their A-P house/unit code format.
Input and Output object IDs have an "I" or "O" prepended to their usual A-Q codes to distinguish them from X-10 ids.
Other objects show using a "Fake" code of the first two letters of their standard HV object type name. For example, Custom Lights show as "LI".
* *Object Name* is the name of the object for use by serial and NetIO commands.
It must be unique among both external and internal names. It can be the same as the topic, but cannot contain "<", ">", "/" or spaces.
Alphanumeric and the underscore are the only allowed characters.
*Object Name* cannot be "pub", "sub" or "unsub"; these are reserved keywords for sending generic MQTT messages.
**N.B.:** HomeVision allows duplicate names among objects, so some modification to the suggested default names will be necessary to achieve uniqueness in the object lists.
* *Topic* is the topic used for publishing and subscribing.
Topic rules are the same as external devices with default processing (i.e., no MQTT wildcards).
* The *State* column will show the reported state of the object.
The auto reporting feature for objects must be turned on, or the HomeVision schedule must explicitly send updates for this to work.
When the state has not (yet) been reported, a "-" may show.
Variables always show a "-" for the state.
* *Level* shows the reported object level or value as appropriate.
X-10 and Light objects show their level in percent, while Variables show their value.
* Rows can be sorted in ascending or descending order by *ID*, *Object Name* or *Topic* by clicking on the column header.
When sorting by ID, object types are always grouped together and sorted within those groups by ID.

*New/Edit/Delete Internal Objects*

* To enter a new object, Click the "New" button and start by selecting an object from the drop-down list.
Objects included in the list are those that have been checked in the
"Object Type List Enable" section of the **Int Object Tab**.
  * If a variable is selected, two additional options are available.
    * "Use Variable as Flag" treats the variable as a flag with its value being either a "0" (off) or "1" (on). If a value is received instead of "on" or "off", if it is "odd" it represents "on" and if "even" it represents "off". The variable will be set  to "0" or "1" accordingly.
    * If "Use Variable as Flag" is NOT selected, then "Use Two Variables" can be selected. If it is, any value received in a stat message will be written to two variables as a 2-byte number. I.e., the LSB will be written to the specified Variable, and the MSB written to the specified Variable + 1. If the option is not selected, the received value is written as a 1-byte number to the specified Variable.
  * If an X-10 object is selected, then choose a Model.
Model is used to determine how level changes are done.
  * Clicking "Copy Object to Topic, Name" will populate the "Topic" and "Name" fields with an appropriate version of the object name.
Some validation is done on the "Topic" and "Name" fields; the "OK" button will be grayed out if either field fails validation. 
The "Topic" and "Name" fields can be modified, keeping in mind the rules above for topics and object/device names.
<br>
<br>
More than one object can have the same topic. This allows a single incoming command to control several objects.
However, note that if a single topic is assigned to multiple objects, when ANY of those objects changes state, the same status message is published so that any external client that might be subscribed to the status message won't be able to tell which object changed state.
<br>
<br>
An exception to this is if different index numbers are appended to the topic, in which case each will act and report independently.
  * Choose if object status messages should be sent with the "retained" flag set.
The retain flag will cause most brokers to "remember" the status and send it to any client that later subscribes to the status topic.
  * Choose whether to log MQTT messages sent or received by this device.
* Objects can be edited by clicking the "Edit" button. This brings up the same window used for adding a new object.
* Use the "Delete" button to delete an object. When a object is deleted, the plug-in unsubscribes to any topics related to that device.
* When "Done" is clicked, objects with standard topics are subscribed to their command full topics.Objects with custom topics are subscribed only to the topic as-is.
*Click "Done" for changes to become effective!*

### Settings Tab

* Prefixes and Postfixes can be set for "standard" topics.
The defaults conform to the Tasmota structure.
They can be changed if necessary, but
avoid Prefixes and Postfixes that have digits at the beginning or end.
* If desired, enter the "Count" payload text.
Leave blank if not used.
* "MQTT Broker web/IP Address" defaults to "localhost" which should work if the MQTT broker is on the same (Linux) computer as HomeVisionXL.
If it doesn't work, or the broker and HomeVisionXL are on different computers,
change it to the MQTT broker's domain or explicit IP address.
* "MQTT Broker Port" defaults to "1883" and probably won't need to change.
* QOS values for Publish and Subscribe messages can be set for brokers that do not support the defaults of QOS 1 for Publish and QOS 2 for Subscribe.
* If a username and password is used to log into the broker, check the "Use Username/Password" box and then fill in the username and password.
* Select the type(s) of state responses desired.
*Valid only for internal objects!*
<br>
"Power" enables a standard response like this:
```
        Full Topic                     Payload
    stat/*topic*/POWER*x*               OFF/ON/ON *level*/*level*
```
"RESULT" enables a response like this, if "Dimming" is unchecked:
```
        Full Topic                     Payload
    stat/*topic*/RESULT          {"POWER*x*":"OFF/ON/ON *level*/*level*"}
```
or like this if "Dimming" is checked:
```
        Full Topic                     Payload
    stat/*topic*/RESULT          {"POWER*x*":"OFF/ON","Dimmer":*level*}*
```
* Note: If set to "Response uses 0-100" (see next item), the POWER value will still be "OFF" if 0 and "ON" if otherwise.
<br>
One or both of "Power" or "RESULT" *must* be selected.
If both are *unchecked*, "Power" will be re-selected automatically.
"Dimming" has no effect if "Power" only is selected.
* Select the format of device state responses for "stat" MQTT messages.
*Valid only for internal objects!*
Responses can be either the "OFF"/"ON"/"ON *level*" format or strictly "*level*", i.e., 0-100.
Response format can be set separately for appliance modules (which normally would be best set to "OFF/ON") and all others (standard X-10, PCS, Custom Lights, etc.).
* If logging is desired, check "Create log file".
Doing so will display an entry box for the folder for the logs.
Log files will be created in this folder.
Enter the folder name directly, or click the "..." button to browse to or create it.
To keep file sizes to a reasonable length, a new file is created each day there is logging information.
The file name is "MQTTLogYYMMDD", with an extension determined by "Log File Extension".
If "Log File Extension" is empty, the file will have a ".txt" extension if HomeVisionXL is running in Windows,
and no extension if running in Linux.
  * Only messages for devices/objects that have their "Log sent messages" or "Log received messages" options checked are logged.
This includes messages sent via the "right-click" menu, serial or NetIO.
  * "Subscribe" and "Unsubscribe" messages are not logged.
  * Received messages to "cmnd/homevision/#" are always logged.
However, responses to this command are determined by the affected objects' settings.

* "Netio string", "Serial string prefix string", and "Serial string terminator character(s)" are set to reasonable defaults and probably don't need to be changed, except in the rare case that they conflict with other plug-ins.

## Responding to External Device State Changes
Refer to <!-- <a href="MQTT_Actions_ext.html">External Device Actions</a> --> [[Help: External Device Actions|Help:-External-Device-Actions]]
for responses to received messages.
<br>
<br>
Note that an assigned Flag or Variable is always updated before the macro is run so the macro can take advantage of the new value.
## Controlling Devices

### Device/Object Display Area
Right-clicking on a line in the Device/Object Display Area
will bring up a menu from which "On", "Off", "Toggle", "State", and "Set to" can be selected.
For internal objects, one or more of these items may be grayed out if not appropriate for the object type.

For external devices, none are grayed out as there is no knowledge of the capabilities of the device. Some or all of these items may not work based on what the device can do (or not do).
<br>
<br>
When selected, for standard topics, the corresponding *command* topic is published. (See **Standard Command Topics** above).
For other non-standard topics, the right-click items may or may not make sense.
<br>
<br>
For external devices, this typically will make its way through the MQTT broker to the external device which would then act on the command.
For external device topics that are not standard, or at least missing the "<", the topic will be published "as-is", with the appropriate payload.
Since the plug-in will have subscribed to this topic, the message will be sent right back to the plug-in from the broker.
<br>
<br>
For internal objects, 
the corresponding command topic is published in the same manner as external devices.
(Some object types "interpret" the selection depending on its capability. For example, clicking "On" for a flag translates to SET, while "Off" translates to CLEAR.)
*No action is taken directly on the internal object.*
Only a cmnd messages is sent out.
However, since the internal object has *subscribed* to the command topic,
the MQTT broker will send it right back to the plug-in,
which will then set the object to the requested state.
If the command topic causes an object state change, the plug-in completes the sequence by publishing a state topic showing the new state.
Here's a sequence chart example that might make it more clear:
```    HomeVision              Plug-in                       MQTT Broker

                     Right-click X-10 object "Den"
                         and click "Toggle":

                        cmnd/den/POWER TOGGLE     --->
                                                  <--- cmnd/den/POWER TOGGLE
                   <--- HV cmd to toggle Den
    sends X-10 cmd
     to toggle Den
                   ---> stat/den/POWER            --->
```

### Serial Control
MQTT devices can be controlled within a schedule via serial commands which take the form:
```
    mqtt: *device_name/object_name* *command*;
```
For example, to toggle a device *named* "sonoff1",
the serial command would be:
```
    mqtt: sonoff1 toggle;
```
"mqtt:" is whatever the "Serial string prefix" is defined as.
";' is whatever the "Serial string terminator character(s)" is defined as.
"toggle" or "2", "on" or "1", "off" or "0", and "state" are allowed.
X-10 and Custom Lights can be set to a level by using "on *level*". E.g.,
```
    mqtt: den on 50;
```
The device *name* is case-insensitive when matching a device *name* in the Device list.
The *command* portion is sent as-is, case-wise.
<br>
<br>
Note: Device *names* are limited to alphas, numbers and the underscore.
<br>Note: While serial commands in the schedule can change the state of internal objects as well as external devices, it may make more sense to just set the internal object directly in the schedule.
In either case, the plug-in will publish a state change topic if the state actually changed.
### NetIO
MQTT devices can be controlled via NetIO using a "netioaction" command in the NetIO application.
For example, to have a button set up to toggle a device *named* "sonoff1",
the button's *sends* attribute would be set to:
```
    sends:  netioaction mqtt sonoff1 toggle
```
"mqtt" is whatever the "Netio string" is defined as.
"toggle" or "2", "on" or "1", "off" or "0", and "state" are allowed.
<br>
<br>
The device *name* is case-insensitive when matching a device *name* in the Device list.
The *command* portion is sent as-is, case-wise.
<br>
<br>
The state of the device can get retrieved by:
```
    reads:  get mqtt sonoff1
```
The *get* command will return the object's state string, depending on the state of the device.
<br>
<br>
Note: This is a "convenience" command, since it actually just reads the state of the Flag or Variable associated with an external device,
or the state of an internal object.
For external devices, if no flag or variable are associated, an empty string is returned.
If the device's state has not yet been reported, the last value of the Flag or Variable will be returned.
<br>
<br>
Also note that this command is essentially the same as the more direct gets:
```
    reads:  get var state *var#*
```
or
```
    reads:  get flag state *flag#*
```
or
```
    reads:  get x10 state *id#*
```
except that no NetIO Custom Returns processing is done.
So the direct object gets are probably better to use then the MQTT versions.

### Sending Generic MQTT Messages

In addition to the external and internal device/object specific MQTT messages,
the plug-in allows generic MQTT messages that may or may not be related to any configured device.
This gives freedom to send anything necessary without tying messages to specific configured devices.
Generic messages can be sent by either NetIO or serial commands.
<br>
<br>
Generic MQTT messages can be sent within a schedule via serial commands and take the form:
```
    mqtt: pub *topic* {*payload*};
    mqtt: sub|unsub *topic* {*callback*};
```
For "pub" commands, if the last word in the payload is "retain",
the command is sent with the MQTT retain flag set.
<br>
<br>
For sub and unsub, if *callback* is given,
then a public callback proc must exist somewhere.
<br>
<br>
If *callback* is not present, then the serial command essentially does nothing.
<br>
<br>
Examples:
```
    mqtt: sub tele/somedevice/state mycb;
    mqtt: unsub tele/somedevice/state mycb;
    mqtt: pub "cmnd/living room/POWER" some payload info;
    mqtt: pub "cmnd/living room/POWER" some payload info retain;
```
Note: If a topic has spaces, the entire topic should be enclosed in double-quotes or braces {}.
<br>
Note: Double-quotes or braces are NOT necessary for any spaces in the payload portion.
<br>
<br>
Generic MQTT messages can be sent via NetIO using a "netioaction" command in the NetIO application in a similar way as serial commands.
<br>
<br>
Examples:
```
    sends:  netioaction mqtt sub tele/somedevice/state mycb
    sends:  netioaction mqtt unsub tele/somedevice/state mycb
    sends:  netioaction mqtt pub {cmnd/living room/POWER} some payload info
    sends:  netioaction mqtt pub {cmnd/living room/POWER} some payload info retain
```
Note: For NetIO, a topic with spaces must be enclosed by braces {}.
Double-quotes are not allowed due to the way NetIO handles arguments of the netioaction command.

### Custom Processing of Received Messages
**Note: This section applies to external devices only.**
<br>
<br>
Sometimes a topic may not fit the standard forms supported by the plug-in, or the actions taken (setting flags, variables and running macros)
may not be powerful enough.
There are two methods that provide more advanced processing:
* Custom commands - Create a plug-in and define a command to run when a topic is received;
* Triggers - Send trigger strings to HomeVisionXL or plug-ins.


#### Custom Commands
To have custom processing of received messages for a topic,
click "Command" and enter a procedure name in the "Command" field.
The procedure becomes the callback proc for this topic.
<br>
<br>
Note: If "Command" is selected,
the "Flag/Var", "On Macro" and "Off Macro" fields are ignored.
However, the action command can be used in the procedure to manipulate flags, vars, macros, etc..
<br>
<br>
**CAUTION:
*There is little validation of the name of the custom procedure.
Care should be taken to avoid any standard TCL procedure names
as inadvertently using an existing procedure name may result in abnormal behavior!
**
<br>
<br>
The procedure must be defined in another enabled plug-in, 
and must be made public via the "hvPublic" command.
A plug-in can contain several different procedures that are called by different messages.
<br>
<br>
The procedure will be called like this:
```
    procedure_name *topic* *payload* *retain*
```
Example:
<br>
<br>
Suppose we have a device (named BathHumidity) running Tasmota software with a AM2301 temperature/humidity sensor attached , and we want to turn on a ventilator fan (named BathFan) when the humidity gets high (>70) and turn it off when it is low (<55).
We would get periodic MQTT tele reports like this:
```

    tele/BathHumidity/SENSOR {
        "Time":"2020-05-20T15:13:25",
        "Switch2":"OFF",
        "AM2301": {
              "Temperature": 75.0,
              "Humidity": 82.0
        },
        "TempUnit": "F"
    }

```
(The JSON payload is expanded here for readability.)
<br>
<br>
In the **Ext Devices Tab**, add an external device for the humidity sensor with a full topic of
```
    tele/BathHumidity/SENSOR
```
The MQTT plug-in will explicitly subscribe to this topic instead of the normal standard state topics.
Click "Command" and set the *Command* entry field to "humid".
Click "OK" to save this entry.
<br>
<br>
Add another external device for the fan with a standard topic
```
    <BathFan>
```
so that it will report status
in the standard way, like this:
```
    stat/BathFan/POWER ON
```
Click "Command" and set the *Command* entry field to "bathfan".
Click "OK" to save this entry.
<br>
<br>
Click "Done" for changes to be effective!
<br>
<br>
Create a plug-in containing the following:
``` tcl
    tcl::tm::path add [file dirname [info script]]
    package require json 1.0

    hvImport mqttComm
    
    hvPublic humid
    proc humid {topic payload} {
        global fanStatus
        if {$payload eq ""} return
        if {[catch {::json::json2dict $payload} status]} return

        set humidity [dict get $status AM2301 Humidity]
        if {$humidity > 70 && !$fanStatus} {
            mqttComm pub "cmnd/BathFan/POWER" on
        }
        if {$humidity < 55 && $fanStatus} {
            mqttComm pub "cmnd/BathFan/POWER" off
        }
    }
    
    hvPublic bathfan
    proc bathfan {topic payload} {
        global fanStatus
        if {[lindex [split $topic "/"] end] eq "POWER"} {
            if {$payload eq "ON"} {
                set fanStatus 1
            } elseif {$payload eq "OFF"} {
                set fanStatus 0
            }
        }
    }
```
(See next Section for a description of "mqttComm".)
<br>
<br>
Procedure *humid* is called whenever a humidity status message is received. It processes the JSON status message to get the humidity and turns the fan on or off depending on the humidity level.
<br>
<br>
Procedure *bathfan* is called whenever a state message from the fan switch is received. It tracks the state of the fan so *humid* only turns the fan on/off if it is not already.
In reality, *bathfan* isn't necessary, as turning on the fan while it is already on does no harm. It's here mainly as an example of how to set up a command.
#### Triggers
For those not comfortable with creating custom procedures, Triggers are a way to get a little more processing power without coding.
Works well for sequential actions that don't require decision making.
<br>
<br>
Instead of a command procedure, enter serial trigger strings to perform actions. A serial trigger string can be one or more "built-in" HomeVisionXL strings and/or serial strings supported by existing plug-ins.
<br>
<br>
Note: Like "Command",
the "Flag/Var", "On Macro" and "Off Macro" fields are ignored.
However, use the action command in trigger(s)
to manipulate flags, vars, macros, etc..
<br>
<br>
To use Triggers, select "Trigger". 
Once selected, the "Topic Type" options are available. 
<br>
<br>
Selecting "Standard" is used with standard topics
(generally, those of the form <topic>),
so "stat" messages with payloads of "on", "off","on {level}" and "{level}" are available.
If "All" is selected, then only one trigger entry is provided, and this is run regardless of the payload.
If "On/Off" is selected, there are separate entries for the "on" and "off" states. However, one of the two fields can be empty, which means that nothing will happen for messages received with that state.
A payload of "0" or "on 0" implies "off".
<br>
<br>
Selecting "Custom" should be used for topics that don't adhere to the "standard" topic model, and hence can't be guaranteed an "on/off" payload.
Only one trigger entry is available and is run regardless of the payload contents.
<br>
<br>
The following special character strings will cause substitutions for every occurrence in a trigger string(s).
(Note: %X, %P, %E, %L don't make sense for "Custom" trigger strings, but, if used, will result in a "0" being substituted.)
* **%X** Substitute a received 0-100 level scaled to 0-16 (standard **X**10).
* **%P** Substitute a received 0-100 level scaled to 0-31 (**P**cs).
* **%E** Substitute a received 0-100 level scaled to 0-63 (dir**E**ct to level).
* **%L** Substitute a received 0-100 **L**evel unscaled.
* **%O** Substitute the t**O**pic.
* **%M** Substitute the entire payload  (**M**essage) unmodified.
* **%m** Substitute the entire payload  (**m**essage), substituting single quotes for double quotes.
* **%D** Substitute the current **D**ate in the form YYYYMMDD.
* **%T** Substitute the current **T**ime in the form HH:MM:SS.
<br>
Examples:
<br>
<br>
Play a sound file and run a program when the topic is received.
This example shows how two separate "built-in" triggers can work together in one trigger string. 
(Assumes that the sound and program functions are working.)
<br>

```
Options: Custom. 

Trigger:
    Play wav file "alarm"  Run program "sendtext"
```
<br>
Write a daily log file. (This may be redundant, as the MQTT plug-in has logging capability in other forms, but shows what can be done.)
```
Options: Standard, All.
 
Trigger:
    write to file "filename%D" "%T: %O:%M"
```
<br>
<br>
Write a daily log file, where the payload may have double quotes in it (i.e., a JSON string).
In that case, the double quotes must be converted to single quotes to avoid confusing the "write to file" processing.
```
Options: Standard, All.
 
Trigger:
    write to file "filename%D" "%T: %O:%m"
```
<br>
*The following examples require action plug-in version 1.3 or greater.*
<br>
<br>
Turn on/off other lights to same level as triggering device.
<br>
```
Options: Standard, On/Off. 

On Trigger:
    action: x10 pcslevel c12 %P, x10 level a3 %X;

Off Trigger:
    action: x10 off c12, x10 off a3;
```
<br>
Tune a TV to a channel using IR.
(Select cable input to TV, then tune a channel with a .5 second delay between digits.)
<br>
```
Options: Standard, On/Off. 

On Trigger:
    action: ir transmit 28 1,ir transmit 114 1,wait for 500,ir transmit 116 1;

Off Trigger:
    {empty}
```
<br>
Tune a TV to a channel on a streaming service.
(Turn on TV, .5 second delay for TV to turn on, select streaming device input, then select streaming channel.)
This example shows how two separate plug-in triggers (action and roku) can work together in one trigger string.
<br>
```
Options: Standard, On/Off. 

On Trigger:
    action: ir transmit 2 1,wait for 500,ir transmit 26 1; roku: 13;

Off Trigger:
    {empty}
```
Note: *roku* is a custom plug-in not generally available.

### mqttComm - Sending/Receiving MQTT Messages from/to Another Plug-in

When total control for MQTT message processing is needed,
this method takes "Custom Commands" a step farther.
A plug-in can be essentially independent of processing that is done in the MQTT plug-in, except for using it as a MQTT transport mechanism.
<br>
<br>
Other plug-ins can interface with the MQTT plug-in via the *mqttComm* command.
The calling plug-in should import the command via:
``` tcl
    hvImport mqttComm
```
The *mqttComm* command has the following formats:
``` tcl
    mqttComm {-log} sub|unsub <*topic*>|*topic* *callback* 
    mqttComm {-exactstat -nodim -log -retain} stat|cmnd <*topic*>|*topic* {*payload*}
    mqttComm {-log -retain} pub *topic* {*payload*}
```

* Note: In previous versions (< 1.76), "type", the name of the calling plug-in, was the first argument. This has been deprecated. However, for backward compatibility, new versions of the command will allow (and ignore) a "type" argument.
* -log: Log the command. This argument is optional.
* -retain: The sent message is retained by the broker. This argument is optional.
* -exactstat: Sends a POWER stat response regardless of power/result settings.
* -nodim: Don't include Dimming in a RESULT response.
* *topic*: Can either be enclosed in "<>" (or any of the other standard forms) or not. If a standard form is used, MQTT's standard prefixes and the Power post-fix will be added to create a full topic. Otherwise, a topic with neither "<" nor ">" is used as-is, without adding the standard pre- and post-fixes, essentially creating generic MQTT messages. If the topic contains spaces, the topic along with any "<" or ">", should be enclosed in double-quotes.
* *payload*: The MQTT message payload to send and is valid only for "cmnd", "stat" and "pub" actions. Double-quotes or braces are NOT necessary for any spaces in the payload portion.
* *callback*: The name of a proc in the calling plug-in that will process the subscribed-to incoming message and is valid only for "sub" and "unsub". It will be called like this:
``` tcl
    *callback fulltopic payload retain*
```
 * *fulltopic*: the complete topic, including any prefix or postfix.
 * *payload*: the message payload.
 * When a new subscription is made by a client, all retained messages that match the full topic are reported with *retain* set to 1. Any messages matching the full topic that are subsequently received by the broker are reported with a value of 0.


See below for further details on the use of callbacks!
<br>
<br>
A "stat" or "cmnd", when used with a topic with neither "<" nor ">", ignores the "stat" or "cmnd" and is the same as using "pub".
<br>
<br>
A "pub" or "cmnd", when used with a standard topic, is the same as using "cmnd".
<br>
<br>
When the standard form is used with "sub" or "unsub", a "cmnd" pre-fix is assumed.
To subscribe or unsubscribe to a "stat" type topic, use the topic without the <.
Example: To subscribe to a stat message:
``` tcl
    mqttComm sub stat/utilityroom> cascallback
```
which will subscribe to
```
    stat/utilityroom/POWER
```
with "cascallback" as the callback proc.
<br>
<br>**Caution:**
Make sure when subscribing to a topic that the topic is unique, especially compared to *external devices* and *internal objects*.
The MQTT client allows multiple, different callback procs to be assigned to the same topic (via different subscriptions) resulting in all of the procs being called when the common topic arrives.
This could be useful, but normally it will cause unexpected behavior.
<br>
<br>
The mqttComm proc returns an empty string if mqtt is not ready.
This can be used to trigger retries until MQTT is ready.
<br>
<br>
The mqttComm proc returns 0 if the type, action or topic is missing or NULL,
or if there is no payload for a cmnd or stat command.
<br>
<br>
The mqttComm proc returns 1 for a successful operation.
<br>
<br>
Examples:
``` tcl
    mqttComm sub <utilityroom> cascallback       subscribes to cmnd/utilityroom/POWER with callback cascallback
    mqttComm unsub stat/utilityroom/power mycallback  unsubscribes to stat/utilityroom/power with callback mycalback
    mqttComm cmnd <utilityroom> on               publishes cmnd/utilityroom/POWER on
    mqttComm stat <utilityroom> on               publishes stat/utilityroom/POWER on and/or
                                                           stat/utilityroom/RESULT {"POWER":"ON","Dimming":100}
    mqttComm -nodim stat <utilityroom> on        publishes stat/utilityroom/POWER on and/or
                                                           stat/utilityroom/RESULT {"POWER":"ON"}
    mqttComm -exactstat stat <utilityroom> on    publishes stat/utilityroom/POWER on 
    mqttComm stat "cmnd/living room/state" on    publishes cmnd/living room/state on
    mqttComm pub "cmnd/living room/state" on     publishes cmnd/living room/state on
```
<br>
<br>
*Callbacks with the "sub" form of mqttComm*
<br>
<br>
When something subscribes to a topic, the assumption is that some action should be taken when a message with that topic arrives.
So how is that handled when a plug-in uses *mqttComm* to subscribe to a topic?
<br>
<br>
This is best explained by example.
Let's assume a plug-in wants to subscribe to a topic:
``` tcl
    mqttComm sub <utilityroom> cascb     
```
This results in a subscription to
```
    cmnd/utilityroom/POWER
```
When received, this topic means either report back the current device state (with an empty or "?" payload),
or change the device state based on the payload.
That means that when the MQTT plug-in receives the message, it must communicate with the subscribing plug-in to complete the response.
It does this by calling the callback proc. Let's say some external entity wants to know the state of utilityroom.
When the mqtt plug-in received the appropriate message, it calls the callback proc like this:
``` tcl
    cascb cmnd/utilityroom/POWER "?" 0
```
A proc "cascb" must be defined in the calling plug-in to handle the message.
In this case, it should eventually use another mqttComm command to send back status, maybe like this:

``` tcl
    mqttComm stat <utilityroom> On 50   --->   stat/utilityroom/POWER ON 50
```
There are a couple of different approaches to handling subscribed topics. Which is better depends on the goals of the plug-in.
<br>
<br>
If the plug-in is going to handle a few very specific fulltopics,
a separate callback for each might be the best way to handle them.
<br>
<br>
If it needs to handle a number of very similar fulltopics, one callback that parses the fulltopic might be best.
<br>
<br>
Either approach will work.

### Other Public Procedures Supplied/Called by the MQTT Plug-in

#### topicTemplate

To help with parsing full topics,
the mqtt plug-in makes public a helper proc that will split a full topic (the topic returned to the callback) into its parts and provide other info about the full topic.
<br>
<br>
The calling plug-in should import the command via:
``` tcl
    hvImport topicTemplate
```
The *topicTemplate* command has the following format:
``` tcl
    topicTemplate *topic*
```
*topic*: fulltopic received by the callback.

topicTemplate returns a list containing a dict with the following key/values:
<br>
<br>

  | Key | Value |
  | :---: | --- |
  | template | One of <T>, >T<, <>T, T<>, ><T, T><, <T, >T, T>, T<, T.| 
  | pre pos | Prefix position 1-3, -1 if no prefix.| 
  | pre type | Prefix string, "" if no prefix.| 
  | topic pos | Sub-topic position 1-3| 
  | topic name | Sub-topic string| 
  | topic type | "s" (standard-contains both < and >), "ns" (non-standard) otherwise.| 
  | post pos | Postfix position 1-3, -1 if no postfix.| 
  | post type | Postfix string, "" if no prefix.| 
  | post ntype | Post type with "Relay" number striped off.| 
  | post index | "Relay" number or "[0-9]*" if none, or "" if no postfix.| 
  | match | Regexp expression to match the full topic.| 

<br>
<br>
There are some details and cautions concerning the use of this helper proc.
Contact the author for further support.

#### mqttLog

Puts an entry into the current MQTT log file.
<br>
<br>
The calling plug-in should import the command via:
``` tcl
    hvImport mqttLog
```
The *mqttlog* command has the following format:
``` tcl
    mqttLog *string* {*color*}
```
* string*: String to log.
*color*: If "debug" is imported into the calling proc,
will send string to the debug plug-in in *color*, Default: red.

#### mqttReady

This proc is *called* by the MQTT plug-in to indicate whether it is connected or disconnected to the MQTT broker (and hence ready to take mqttComm calls).
The using plug-in should make the command public via:
``` tcl
    hvPublic mqttReady
```
When the MQTT connection changes, the MQTT plug-in calls *mqttReady* like this:
``` tcl
    mqttReady *status*
```
*status* is a dict  of either {state connected} or {state disconnected reason *reason*}.
Possible values for *reason* are:
```
    0 Normal disconnect
    1 Unacceptable protocol version
    2 Identifier rejected
    3 Server unavailable
    4 Bad user name or password
```

Using this proc is only necessary to make sure the plug-in's subscriptions are re-done automatically in the case the connection to the broker going down and then recovers (with a {state connected} return).
<br>
<br>
Reasons "1", "2", and "4" are fatal and need to be corrected before a connection can be made.
<br>
<br>
To use, define an mqttReady proc to respond to the connected and/or disconnected states.
Typical use:
``` tcl
    hvPublic mqttReady
    proc mqttReady {status} {
    
        if {[dict get $status state] eq "connected"} {
            #code to run when MQTT is ready.
        } else {
            #code to run when MQTT is not ready.
        }
    }
```

### MQTT Discovery for Home Assistant

[Tips for Interfacing HomeVision with Home Assistant](https://github.com/rebel7580/MQTT-Plug-in-For-HomeVisionXL/wiki/Tips-for-Interfacing-HomeVision-with-Home-Assistant)
<br>
[How to Use the MQTT Plug-in's Home-Assistant Auto Discovery](HomeVision_Discovery_How-to)


