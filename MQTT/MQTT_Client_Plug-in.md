<!-- $Revision: 1.29 $ -->
<!-- $Date: 2021/07/22 21:55:08 $ -->
<body>
<h1>MQTT Client Plug-in<h1>
<h2>Overview</h2>
The MQTT Client Plug-in provides a client interface to MQTT for HomeVision.
Its main purposes are to control MQTT enabled devices via the HomeVision Schedule or NetIO and
to control HomeVision objects by MQTT sources.
The MQTT interface has three distinct functions:
<ul>
<li>
For "external" MQTT-enabled devices (e.g., Sonoff switches with Tasmota SW),
the MQTT Plug-in acts as an MQTT controller.
The plug-in PUBLISHES <i>command</i> topics to these devices
to control them
and SUBSCRIBES to <i>status</i> topics from these devices to track their state changes.
Changes are tracked by Actions such as setting Flags or Variables and running Macros based on that status.
<li>
For "internal" objects defined in HomeVision
(such as X-10 modules, flags, inputs, etc.),
the MQTT Plug-in acts as a "proxy" for them, essentially making them appear to be MQTT-enabled.
For each selected object, the plug-in 
SUBSCRIBES to a <i>command</i> topic
so it can be controlled
and
PUBLISHES a <i>status</i> topic when it changes state.
<li>
Generic MQTT topics can be sent from the HomeVision schedule via serial commands, from NetIO, or from custom plug-ins, independent of any configured devices.
</ul>
While the client plug-in is designed to work easily with Tasmota based devices, using a similar topic and LWT structure,
other devices that follow different topic structures likely can be accommodated as well.
There is a lot of flexibility in the plug-in allowing support for many different situations.
<br>
<br>
<b>Note: To perform actions on internal objects, this plug-in uses the Actions Plug-in. The Actions Plug-in must be enabled to control internal objects.</b>
<h2> Supported MQTT Topics</h2>
<h3>Standard and Custom Topics</h3>
Topics can be assigned to devices either as abbreviated ("Standard") topics or full topics.
Standard topics follow the Tasmota structure, so only the unique sub-topic portion need be entered. Default standard topics are indicated by enclosing the sub-topic with "&lt;" and "&gt;".
If a topic starts with a "&lt;", the appropriate prefix is automatically added.
If a topic ends with a "&gt;", the appropriate postfix is automatically added.
A "&gt;" may be followed by optional index digits.
Prefixes and postfixes are defined in the <b>Settings Tab</b>.
Most of the examples to follow assume use of the defaults.
<br>
<br>
To provide additional flexibility in defining a system's topic structure, standard topics can have the prefix and postfix indicators in any position relative to the sub-topic.
The following are possible (where "T" is the sub-topic):
<pre>
   Template        Example
      &lt;T&gt;       cmnd/T/POWER    --&gt; Tasmota standard and MQTT plug-in default
      T&lt;&gt;       T/cmnd/POWER    
      &lt;&gt;T       cmnd/POWER/T
      &gt;T&lt;       POWER/T/cmnd
      T&gt;&lt;       T/POWER/cmnd
      &gt;&lt;T       POWER/cmnd/T
</pre>
To accommodate certain Tasmota devices that have multiple relays or switches, one or more digits can follow the "&gt;" postfix index to specify the relay or switch.
For example, an external device of this type that sends out state reports like this:
<pre>
    stat/FloorLamp/POWER2 ON
</pre>
should be set up with this topic structure:
<pre>
    &lt;FloorLamp&gt;2
</pre>
Devices that send out reports for several "relays" should have separate entries in the <b>Ext Devices Tab</b> for each "relay".
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
If a topic has only a "&lt;" OR a "&gt;", then only the appropriate
prefix OR postfix is automatically added.
Otherwise, the topic is used as-is without any additional portions prepended/appended to it.
Thus by not putting "&lt;" and "&gt;" in a topic,
any topic structure can be used.
However, the plug-in does not have built-in processing when receiving MQTT topics that don't follow the standard prefixes and postfixes. In these cases custom procedures need to be provided. See <b>Custom Processing of Received Messages</b>.
<br>
<br>
Most of the examples in the rest of this Help follow the Tasmota standard.

<h3>Standard Command Topic</h3>
The following command topic is supported (using default prefixes and postfixes) with various payloads:
<pre>
        Full Topic     Payload
    cmnd/<i>topic</i>/POWER  ON
    cmnd/<i>topic</i>/POWER  ON <i>level</i>*
    cmnd/<i>topic</i>/POWER  <i>level</i>*
    cmnd/<i>topic</i>/POWER  OFF
    cmnd/<i>topic</i>/POWER  TOGGLE
    cmnd/<i>topic</i>/POWER  empty or "?"** 
</pre>
<i>topic</i> is defined during device configuration.
<br>Payloads are case-insensitive for incoming messages.<br>
* <i>level</i> is expressed as a percentage, 0-100.
For X-10 devices, it  is converted to the nearest discrete level supported.
Any status reports back will have the discrete level.
<br>
** "?" for those external entities that don't allow an empty payload.
<br>
<br>
The plug-in <i>publishes</i> command topics to external devices.
Usually, after receiving one of these commands, an external device will publish its state.
The plug-in receives this report by <i>subscribing</i> to the state topic. (See next.)
An empty payload requests the device to publish its state without changing the state of the device.
<br>
<br>
HomeVision objects <i>subscribe</i> to command topics so that external entities can control them.
For a full list of Actions based on the received topic and payload, see 
<!-- <a href="MQTT_Actions_int.html">Object Actions</a>
-->
[[Help: Object Actions|Help:-Object-Actions]]

<h3>Standard State Topic</h3>
For each external device, the plug-in <i>subscribes</i> to the following topic (assuming "&lt;<i>topic</i>&gt; in the "Topic" field):
<pre>
        Full Topic                     Payload
    stat/<i>topic</i>/POWER<i>x</i>           OFF/ON/ON <i>level</i>/<i>level</i>
</pre>
When a POWER topic is received, the plug-in looks for a message of "on" or "off" and takes action according to the device's settings in the <b>Ext Devices Tab</b>.
For a full list of Actions based on the received topic and payload, see
<!-- <a href="MQTT_Actions_ext.html">External Device Actions</a> -->
[[Help: External Device Actions|Help:-External-Device-Actions]]
.
<br>
<br>
When an internal object changes state, the plug-in will <i>publish</i> a state message to indicate the object's new state.
<h3>RESULT State Topic</h3>
Certain devices, i.e., those running Tasmota software, usually produce both at POWER state message and a RESULT state message with the POWER info in a JSON formatted payload.
In fact, RESULT state messages can be sent from the device when many other actions occur.
<br>
<br>
To receive such messages, they must explicitly be entered as a device in the <b>Ext Devices Tab</b> with a <i>custom command</i> defined to process the message.
<pre>
          Topic                         Payload
    stat/<i>topic</i>/RESULT               a JSON string
</pre>
<h3>Last Will and Testament Topic</h3>
For each external device with a <i>standard topic</i>
that has "Subscribe to Last Will and Testament" checked,
the plug-in automatically <i>subscribes</i> to
a <i>Last Will and Testament</i> topic as follows:
<pre>
        Full Topic     
      tele/<i>topic</i>/LWT
</pre>
If received, the plug-in looks for a message of "online" or "offline" and attempt to display this message in the <b>Ext Devices Tab</b>.
<br>
<br>
If the topic is not in the form of a standard topic,
the "Subscribe to Last Will and Testament" checkbox has no effect.
<br>
<br>
LWT is not supported for internal objects.
<h3>Special "homevision" Topic</h3>
The MQTT plug-in automatically subscribes to:
<pre>
        Full Topic
    cmnd/homevision/#
</pre>
When a message is received in the form:
<pre>
        Full Topic                        Payload
    cmnd/homevision/<i>object_type</i>/POWER    empty or "?"
    cmnd/homevision/POWER               empty or "?"
</pre>
for each defined object matching <i>object_type</i>
(one of
x10, light, var, flag, hvac, temp, analog, input or output)
or for all object types if not specified, the plug-in will <i>publish</i> a state message for each object to indicate its current state.
<i>Only those objects specified in the <b>Int Objects Tab</b> are reported</i>.
This feature can be used by an MQTT entity to quickly sync up with HomeVision object states.
<br>
<br>
The only valid Payloads are empty or "?".
This topic cannot be used to <i>control</i> objects.
<h3>Special Counting Payload</h3>
The MQTT plug-in has special handling to provide a "counting" feature.
Certain MQTT messages will <i>increment</i> selected variable(s) each time the message is received. To activate this feature,
create either an external device or internal object with either one or two variables selected.
Then, in the "Count Payload Text" of the <i>Settings Tab</i>, enter the text that will indicate "Count".
For external devices, receiving a "stat" or "tele" "POWER" message that has its first word in the payload matching "Count Payload Text" will cause the selected variable(s) to increment.
For internal objects, a "cmnd" "Power" message will do the same.
<br>
<br>
For example, if "Count Payload Text" is set to "CYCLE", and an internal object is created with Var-100 and Var-101 assigned to it but re-named "Watchdog",
then when the following is received:
<pre>
    cmnd/Watchdog/POWER CYCLE
</pre>
Var-100 and Var-101 are incremented.
<br>
<br>
Note: Other messages that would set variables assigned to this topic will continue to do so.
This can be used to advantage; sending a "cmnd" "POWER" message with a payload of "0" would "reset" the variables so subsequent "Count" messages would start from zero.
<h2>Configuring Devices</h2>
<h3>Ext Devices Tab</h3>
This tab contains a list of supported external devices.
<ul>
<li>
<i>Device Name</i> is the name of the device for use by serial and NetIO commands.
It must be unique among both external and internal device names.
See below for Name rules.<li>
<i>Topic</i> is the topic used for publishing and subscribing.
See below for Topic rules.
<li>
The <i>Flag/Var</i> column shows the Flag (FL-#) or Variable (VA-#) assigned to the device.
If none is assigned, a "-" will show.
<li>
The <i>Macro</i> column shows the Macro(s) assigned to the device in the form {on macro#}/{off macro#}.
If no macro is assigned, a "-" will show. 
<li>
The <i>State</i> column will usually show "On" or "Off", depending on the reported state of the device.
If the payload was an number, its value, potentially masked and/or truncated, will be displayed.
When the state has not (yet) been reported, a "-" will show.
<li>
For each device, its row will be displayed in RED text if the device is reported as "off line" by an LWT message.
However, if an "on Line" LWT or a POWER state message is received,
the row will display in BLACK text.
<li>
Rows can be sorted in ascending or descending order by <i>Device Name</i> or <i>Topic</i> by clicking on the column header. 
</ul>
<i>New/Edit/Delete External Devices</i>
<ul>
<li>
To enter a new device, click the "New" button, and start with the <i>Topic</i>.
<ul>
<li>
Topics are case-sensitive!
<li>
A "Standard" topic, for which prefix and postfix substitution is performed (see <b>Settings Tab</b>),
is indicated by enclosing it in "&lt;" and "&gt;"
(or one of the variations mentioned previously).
E.g., "<<i>topic</i>>".
<i>This should be the default method for specifying topics.</i>
To help with this, a new device will have "<>" already inserted in the topic field.
(Delete or modify as needed.)
<li>
Topics can contain multiple levels, indicated by "/", but should not start or end with a "/".
<li>
A topic with <i>default</i> processing of received messages should not contain "#" or "+" MQTT wildcards, since it is used both to subscribe and publish and wildcards are not allowed in published topics.
<li>
A topic with <i>custom</i> processing of received messages MAY contain "#" or "+" MQTT wildcards, since it will only be subscribed to.
Use of wildcards should adhere to the MQTT standards.
However, care should be taken when using them. It's probably a bad idea to have "#" as a topic, as it will subscribe to EVERYTHING!

<li>
<i>Device Names</i>, not topics, are used by NetIO and serial commands to publish commands to MQTT devices.
If the "topic" is simple and descriptive, it can be copied to the <i>Name</i> field.
If the "topic" is multi-level, <i>Name</i> must be simpler.
<i>Name</i> cannot contain "&lt;", "&gt;", "/" or spaces.
Alphanumeric and the underscore are the only allowed characters.
<i>Device Name</i> cannot be "pub", "sub" or "unsub"; these are reserved keywords for sending generic MQTT messages.
<li>
There is some validation of the <i>Topic</i> and <i>Device Name</i> fields to enforce the above rules and to avoid name duplication, but it may not be perfect.
<li>
For each device, an HV Flag or Variable can be assigned. 
Assigning a Flag or Variable is optional.
There is no checking to make sure a Flag or Variable is not used more than once.
<li>
If a variable is selected, two additional options are available.
<ul>
<li>
"Use Variable as Flag" treats the variable as a flag with its value being either a "0" (off) or "1" (on). If a value is received instead of "on" or "off", if it is "odd" it represents "on" and if "even" it represents "off". The variable will be set  to "0" or "1" accordingly.
<li>
If "Use Variable as Flag" is NOT selected, then "Use Two Variables"
can be selected.
If it is, any value received in a stat message will be written to two variables as a 2-byte number. I.e., the LSB will be written to the specified Variable, and the MSB written to the specified Variable + 1.
If the option is not selected, the received value is written as a 1-byte number to the specified Variable.
<br>
<br>
Note: When "Use Variable as Flag" is not selected, Payloads with "on" or "off" do not cause any Actions, except for updating the "State" display.
</ul>
<li>
Macros can be assigned for "On" and "Off" states.
The same macro can be assigned to both states, or different macros can be assigned.
A macro can be assigned to just one of the states, leaving the other set to "None".
<li>
To have custom processing of received messages or triggers for this topic, select the appropriate mode and enter command/triggers.
See <b>Custom Processing of Received Messages</b> for details.
<li>
Check "Subscribe to Last Will and Testament" to subscribe standard topics to LWT.
If the topic is not in the form of a standard topic,
this selection has no effect.
<li>
Choose whether to log MQTT messages sent or received by this device.
<li> Click "OK" to save edits,
or "Cancel" to discard them.
</ul>
<li>
Devices can be edited by clicking the "Edit" button.
This brings up the same window used for adding a new device.
<li>
Use the "Delete" button to delete a device.
When a device is deleted, the plug-in unsubscribes to any topics related to that device.
<li>
When "Done" is clicked, devices with standard topics are subscribed to their state and LWT full topics.
Devices with custom topics are subscribed only to the topic as-is.
<i>Click "Done" for changes to become effective!</i>
</ul>

<h3>Int Objects Tab</h3>
This tab contains a list of supported internal objects.
These can be any of: X-10, Custom Lights, Flags, Variables, Inputs, Outputs, IR, Digital Temperature Sensors, Analog Inputs, or HVAC.
Add only those objects that need to be visible to or acted on by the MQTT network.
<ul>
<li>
<i>ID</i>
is the internal object ID.
X-10 object IDs show with their A-P house/unit code format.
Input and Output object IDs have an "I" or "O" prepended to their usual A-Q codes to distinguish them from X-10 ids.
Other objects show using a "Fake" code of the first two letters of their standard HV object type name. For example, Custom Lights show as "LI".
<li>
<i>Object Name</i> is the name of the object for use by serial and NetIO commands.
It must be unique among both external and internal names. It can be the same as the topic, but cannot contain "&lt;", "&gt;", "/" or spaces.
Alphanumeric and the underscore are the only allowed characters.
<i>Object Name</i> cannot be "pub", "sub" or "unsub"; these are reserved keywords for sending generic MQTT messages.
<b>N.B.:</b> HomeVision allows duplicate names among objects, so some modification to the suggested default names will be necessary to achieve uniqueness in the object lists.
<li>
<i>Topic</i> is the topic used for publishing and subscribing.
Topic rules are the same as external devices with default processing (i.e., no MQTT wildcards).
<li>
The <i>State</i> column will show the reported state of the object.
The auto reporting feature for objects must be turned on, or the HomeVision schedule must explicitly send updates for this to work.
When the state has not (yet) been reported, a "-" may show.
Variables always show a "-" for the state.
<li>
<i>Level</i> shows the reported object level or value as appropriate.
X-10 and Light objects show their level in percent, while Variables show their value.
<li>
Rows can be sorted in ascending or descending order by <i>ID</i>, <i>Object Name</i> or <i>Topic</i> by clicking on the column header.
When sorting by ID, object types are always grouped together and sorted within those groups by ID.
</ul>
<i>New/Edit/Delete Internal Objects</i>
<ul>
<li>
To enter a new object, Click the "New" button and start by selecting an object from the drop-down list.
Objects included in the list are those that have been checked in the
"Object Type List Enable" section of the <b>Int Object Tab</b>.
<ul>
<li>
If a variable is selected, two additional options are available.
<ul>
<li>
"Use Variable as Flag" treats the variable as a flag with its value being either a "0" (off) or "1" (on). If a value is received instead of "on" or "off", if it is "odd" it represents "on" and if "even" it represents "off". The variable will be set  to "0" or "1" accordingly.
<li>
If "Use Variable as Flag" is NOT selected, then "Use Two Variables"
can be selected.
If it is, any value received in a stat message will be written to two variables as a 2-byte number. I.e., the LSB will be written to the specified Variable, and the MSB written to the specified Variable + 1.
If the option is not selected, the received value is written as a 1-byte number to the specified Variable.
</ul>
<li>
If an X-10 object is selected, then choose a Model.
Model is used to determine how level changes are done.
<li>
Clicking "Copy Object to Topic, Name" will populate the "Topic" and "Name" fields with an appropriate version of the object name.
Some validation is done on the "Topic" and "Name" fields; the "OK" button will be grayed out if either field fails validation. 
The "Topic" and "Name" fields can be modified, keeping in mind the rules above for topics and object/device names.
<br>
<br>
More than one object can have the same topic. This allows a single incoming command to control several objects.
However, note that if a single topic is assigned to multiple objects, when ANY of those objects changes state, the same status message is published so that any external client that might be subscribed to the status message won't be able to tell which object changed state.
<br>
<br>
An exception to this is if different index numbers are appended to the topic, in which case each will act and report independently.
<li>
Choose if object status messages should be sent with the "retained" flag set.
The retain flag will cause most brokers to "remember" the status and send it to any client that later subscribes to the status topic.
<li>
Choose whether to log MQTT messages sent or received by this device.
</ul>
<li>
Objects can be edited by clicking the "Edit" button.
This brings up the same window used for adding a new object.
<li>
Use the "Delete" button to delete an object.
When a object is deleted, the plug-in unsubscribes to any topics related to that device.
<li>
When "Done" is clicked, objects with standard topics are subscribed to their command full topics.
Objects with custom topics are subscribed only to the topic as-is.
<i>Click "Done" for changes to become effective!</i>
</ul>

<h3>Settings Tab</h3>
<ul>
<li>
Prefixes and Postfixes can be set for "standard" topics.
The defaults conform to the Tasmota structure.
They can be changed if necessary, but
avoid Prefixes and Postfixes that have digits at the beginning or end.
<li>
If desired, enter the "Count" payload text.
Leave blank if not used.
<li>
"MQTT Broker web/IP Address" defaults to "localhost" which should work if the MQTT broker is on the same (Linux) computer as HomeVisionXL.
If it doesn't work, or the broker and HomeVisionXL are on different computers,
change it to the MQTT broker's domain or explicit IP address.
<li>
"MQTT Broker Port" defaults to "1883" and probably won't need to change.
<li>
QOS values for Publish and Subscribe messages can be set for brokers that do not support the defaults of QOS 1 for Publish and QOS 2 for Subscribe.
<li>
If a username and password is used to log into the broker, check the "Use Username/Password" box and then fill in the username and password.
<li>
Select the type(s) of state responses desired.
<i>Valid only for internal objects!</i>
<br>
"Power" enables a standard response like this:
<pre>
        Full Topic                     Payload
    stat/<i>topic</i>/POWER<i>x</i>               OFF/ON/ON <i>level</i>/<i>level</i>
</pre>
"RESULT" enables a response like this, if "Dimming" is unchecked:
<pre>
        Full Topic                     Payload
    stat/<i>topic</i>/RESULT          {"POWER<i>x</i>":"OFF/ON/ON <i>level</i>/<i>level</i>"}
</pre>
or like this if "Dimming" is checked:
<pre>
        Full Topic                     Payload
    stat/<i>topic</i>/RESULT          {"POWER<i>x</i>":"OFF/ON","Dimmer":<i>level</i>}*
</pre>
* Note: If set to "Response uses 0-100" (see next item), the POWER value will still be "OFF" if 0 and "ON" if otherwise.
<br>
One or both of "Power" or "RESULT" <i>must</i> be selected.
If both are <i>unchecked</i>, "Power" will be re-selected automatically.
"Dimming" has no effect if "Power" only is selected.
<li>
Select the format of device state responses for "stat" MQTT messages.
<i>Valid only for internal objects!</i>
Responses can be either the "OFF"/"ON"/"ON <i>level</i>" format or strictly "<i>level</i>", i.e., 0-100.
Response format can be set separately for appliance modules (which normally would be best set to "OFF/ON") and all others (standard X-10, PCS, Custom Lights, etc.).
<li>
If logging is desired, check "Create log file".
Doing so will display an entry box for the folder for the logs.
Log files will be created in this folder.
Enter the folder name directly, or click the "..." button to browse to or create it.
To keep file sizes to a reasonable length, a new file is created each day there is logging information.
The file name is "MQTTLogYYMMDD", with an extension determined by "Log File Extension".
If "Log File Extension" is empty, the file will have a ".txt" extension if HomeVisionXL is running in Windows,
and no extension if running in Linux.
<ul>
<li>
Only messages for devices/objects that have their "Log sent messages" or "Log received messages" options checked are logged.
This includes messages sent via the "right-click" menu, serial or NetIO.
<li>
"Subscribe" and "Unsubscribe" messages are not logged.
<li>
Received messages to "cmnd/homevision/#" are always logged.
However, responses to this command are determined by the affected objects' settings.
</ul>
<li>
"Netio string", "Serial string prefix string", and "Serial string terminator character(s)" are set to reasonable defaults and probably don't need to be changed, except in the rare case that they conflict with other plug-ins.
</ul>
<h2>Responding to External Device State Changes</h2>
Refer to <!-- <a href="MQTT_Actions_ext.html">External Device Actions</a> --> [[Help: External Device Actions|Help:-External-Device-Actions]]
for responses to received messages.
<br>
<br>
Note that an assigned Flag or Variable is always updated before the macro is run so the macro can take advantage of the new value.
<h2>Controlling Devices</h2>
<h3>Device/Object Display Area</h3>
Right-clicking on a line in the Device/Object Display Area
will bring up a menu from which "On", "Off", "Toggle", "State", and "Set to" can be selected.
For internal objects, one or more of these items may be grayed out if not appropriate for the object type.

For external devices, none are grayed out as there is no knowledge of the capabilities of the device. Some or all of these items may not work based on what the device can do (or not do).
<br>
<br>
When selected, for standard topics, the corresponding <i>command</i> topic is published. (See <b>Standard Command Topics</b> above).
For other non-standard topics, the right-click items may or may not make sense.
<br>
<br>
For external devices, this typically will make its way through the MQTT broker to the external device which would then act on the command.
For external device topics that are not standard, or at least missing the "&lt;", the topic will be published "as-is", with the appropriate payload.
Since the plug-in will have subscribed to this topic, the message will be sent right back to the plug-in from the broker.
<br>
<br>
For internal objects, 
the corresponding command topic is published in the same manner as external devices.
(Some object types "interpret" the selection depending on its capability. For example, clicking "On" for a flag translates to SET, while "Off" translates to CLEAR.)
<i>No action is taken directly on the internal object.</i>
Only a cmnd messages is sent out.
However, since the internal object has <i>subscribed</i> to the command topic,
the MQTT broker will send it right back to the plug-in,
which will then set the object to the requested state.
If the command topic causes an object state change, the plug-in completes the sequence by publishing a state topic showing the new state.
Here's a sequence chart example that might make it more clear:
<pre>    HomeVision              Plug-in                       MQTT Broker

                     Right-click X-10 object "Den"
                         and click "Toggle":

                        cmnd/den/POWER TOGGLE     ---&gt;
                                                  &lt;--- cmnd/den/POWER TOGGLE
                   &lt;--- HV cmd to toggle Den
    sends X-10 cmd
     to toggle Den
                   ---&gt; stat/den/POWER            ---&gt;
</pre>

<h3>Serial Control</h3>
MQTT devices can be controlled within a schedule via serial commands which take the form:
<pre>
    mqtt: <i>device_name/object_name</i> <i>command</i>;
</pre>
For example, to toggle a device <i>named</i> "sonoff1",
the serial command would be:
<pre>
    mqtt: sonoff1 toggle;
</pre>
"mqtt:" is whatever the "Serial string prefix" is defined as.
";' is whatever the "Serial string terminator character(s)" is defined as.
"toggle" or "2", "on" or "1", "off" or "0", and "state" are allowed.
X-10 and Custom Lights can be set to a level by using "on <i>level</i>". E.g.,
<pre>
    mqtt: den on 50;
</pre>
The device <i>name</i> is case-insensitive when matching a device <i>name</i> in the Device list.
The <i>command</i> portion is sent as-is, case-wise.
<br>
<br>
Note: Device <i>names</i> are limited to alphas, numbers and the underscore.
<br>Note: While serial commands in the schedule can change the state of internal objects as well as external devices, it may make more sense to just set the internal object directly in the schedule.
In either case, the plug-in will publish a state change topic if the state actually changed.
<h3>NetIO</h3>
MQTT devices can be controlled via NetIO using a "netioaction" command in the NetIO application.
For example, to have a button set up to toggle a device <i>named</i> "sonoff1",
the button's <i>sends</i> attribute would be set to:
<pre>
    sends:  netioaction mqtt sonoff1 toggle
</pre>
"mqtt" is whatever the "Netio string" is defined as.
"toggle" or "2", "on" or "1", "off" or "0", and "state" are allowed.
<br>
<br>
The device <i>name</i> is case-insensitive when matching a device <i>name</i> in the Device list.
The <i>command</i> portion is sent as-is, case-wise.
<br>
<br>
The state of the device can get retrieved by:
<pre>
    reads:  get mqtt sonoff1
</pre>
The <i>get</i> command will return the object's state string, depending on the state of the device.
<br>
<br>
Note: This is a "convenience" command, since it actually just reads the state of the Flag or Variable associated with an external device,
or the state of an internal object.
For external devices, if no flag or variable are associated, an empty string is returned.
If the device's state has not yet been reported, the last value of the Flag or Variable will be returned.
<br>
<br>
Also note that this command is essentially the same as the more direct gets:
<pre>
    reads:  get var state <i>var#</i>
</pre>
or
<pre>
    reads:  get flag state <i>flag#</i>
</pre>
or
<pre>
    reads:  get x10 state <i>id#</i>
</pre>
except that no NetIO Custom Returns processing is done.
So the direct object gets are probably better to use then the MQTT versions.

<h3>Sending Generic MQTT Messages</h3>

In addition to the external and internal device/object specific MQTT messages,
the plug-in allows generic MQTT messages that may or may not be related to any configured device.
This gives freedom to send anything necessary without tying messages to specific configured devices.
Generic messages can be sent by either NetIO or serial commands.
<br>
<br>
Generic MQTT messages can be sent within a schedule via serial commands and take the form:
<pre>
    mqtt: pub <i>topic</i> {<i>payload</i>};
    mqtt: sub|unsub <i>topic</i> {<i>callback</i>};
</pre>
For "pub" commands, if the last word in the payload is "retain",
the command is sent with the MQTT retain flag set.
<br>
<br>
For sub and unsub, if <i>callback</i> is given,
then a public callback proc must exist somewhere.
<br>
<br>
If <i>callback</i> is not present, then the serial command essentially does nothing.
<br>
<br>
Examples:
<pre>
    mqtt: sub tele/somedevice/state mycb;
    mqtt: unsub tele/somedevice/state mycb;
    mqtt: pub "cmnd/living room/POWER" some payload info;
    mqtt: pub "cmnd/living room/POWER" some payload info retain;
</pre>
Note: If a topic has spaces, the entire topic should be enclosed in double-quotes or braces {}.
<br>
Note: Double-quotes or braces are NOT necessary for any spaces in the payload portion.
<br>
<br>
Generic MQTT messages can be sent via NetIO using a "netioaction" command in the NetIO application in a similar way as serial commands.
<br>
<br>
Examples:
<pre>
    sends:  netioaction mqtt sub tele/somedevice/state mycb
    sends:  netioaction mqtt unsub tele/somedevice/state mycb
    sends:  netioaction mqtt pub {cmnd/living room/POWER} some payload info
    sends:  netioaction mqtt pub {cmnd/living room/POWER} some payload info retain
</pre>
Note: For NetIO, a topic with spaces must be enclosed by braces {}.
Double-quotes are not allowed due to the way NetIO handles arguments of the netioaction command.

<h3>Custom Processing of Received Messages</h3>
<b>Note: This section applies to external devices only.</b>
<br>
<br>
Sometimes a topic may not fit the standard forms supported by the plug-in, or the actions taken (setting flags, variables and running macros)
may not be powerful enough.
There are two methods that provide more advanced processing:
<ul>
<li> 
Custom commands - Create a plug-in and define a command to run when a topic is received;
<li>
Triggers - Send trigger strings to HomeVisionXL or plug-ins.
</ul>
<h4>Custom Commands</h4>
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
<b>CAUTION:
<i>There is little validation of the name of the custom procedure.
Care should be taken to avoid any standard TCL procedure names
as inadvertently using an existing procedure name may result in abnormal behavior!
</i></b>
<br>
<br>
The procedure must be defined in another enabled plug-in, 
and must be made public via the "hvPublic" command.
A plug-in can contain several different procedures that are called by different messages.
<br>
<br>
The procedure will be called like this:
<pre>
    procedure_name <i>topic</i> <i>payload</i> <i>retain</i>
</pre>
Example:
<br>
<br>
Suppose we have a device (named BathHumidity) running Tasmota software with a AM2301 temperature/humidity sensor attached , and we want to turn on a ventilator fan (named BathFan) when the humidity gets high (&gt;70) and turn it off when it is low (&lt;55).
We would get periodic MQTT tele reports like this:
<pre>

    tele/BathHumidity/SENSOR {
        "Time":"2020-05-20T15:13:25",
        "Switch2":"OFF",
        "AM2301": {
              "Temperature": 75.0,
              "Humidity": 82.0
        },
        "TempUnit": "F"
    }

</pre>
(The JSON payload is expanded here for readability.)
<br>
<br>
In the <b>Ext Devices Tab</b>, add an external device for the humidity sensor with a full topic of
<pre>
    tele/BathHumidity/SENSOR
</pre>
The MQTT plug-in will explicitly subscribe to this topic instead of the normal standard state topics.
Click "Command" and set the <i>Command</i> entry field to "humid".
Click "OK" to save this entry.
<br>
<br>
Add another external device for the fan with a standard topic
<pre>
    &lt;BathFan&gt;
</pre>
so that it will report status
in the standard way, like this:
<pre>
    stat/BathFan/POWER ON
</pre>
Click "Command" and set the <i>Command</i> entry field to "bathfan".
Click "OK" to save this entry.
<br>
<br>
Click "Done" for changes to be effective!
<br>
<br>
Create a plug-in containing the following:
<pre>
    tcl::tm::path add [file dirname [info script]]
    package require json 1.0

    hvImport mqttComm
    
    hvPublic humid
    proc humid {topic payload} {
        global fanStatus
        if {$payload eq ""} return
        if {[catch {::json::json2dict $payload} status]} return

        set humidity [dict get $status AM2301 Humidity]
        if {$humidity &gt; 70 && !$fanStatus} {
            mqttComm pub "cmnd/BathFan/POWER" on
        }
        if {$humidity &lt; 55 && $fanStatus} {
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
</pre>
(See next Section for a description of "mqttComm".)
<br>
<br>
Procedure <i>humid</i> is called whenever a humidity status message is received. It processes the JSON status message to get the humidity and turns the fan on or off depending on the humidity level.
<br>
<br>
Procedure <i>bathfan</i> is called whenever a state message from the fan switch is received. It tracks the state of the fan so <i>humid</i> only turns the fan on/off if it is not already.
In reality, <i>bathfan</i> isn't necessary, as turning on the fan while it is already on does no harm. It's here mainly as an example of how to set up a command.
<h4>Triggers</h4>
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
(generally, those of the form &lt;topic&gt;),
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
<dl>
<dt>%X</dt>
<dd>Substitute a received 0-100 level scaled to 0-16 (standard <b>X</b>10).</dd>
<dt>%P</dt>
<dd>Substitute a received 0-100 level scaled to 0-31 (<b>P</b>cs).</dd>
<dt>%E</dt>
<dd>Substitute a received 0-100 level scaled to 0-63 (dir<b>E</b>ct to level).</dd>
<dt>%L</dt>
<dd>Substitute a received 0-100 <b>L</b>evel unscaled.</dd>
<dt>%O</dt>
<dd>Substitute the t<b>O</b>pic.</dd>
<dt>%M</dt>
<dd>Substitute the entire payload  (<b>M</b>essage) unmodified.
<dt>%m</dt>
<dd>Substitute the entire payload  (<b>m</b>essage), substituting single quotes for double quotes.
<dt>%D</dt>
<dd>Substitute the current <b>D</b>ate in the form YYYYMMDD.</dd>
<dt>%T</dt>
<dd>Substitute the current <b>T</b>ime in the form HH:MM:SS.</dd>
</dl>
<br>
Examples:
<br>
<br>
Play a sound file and run a program when the topic is received.
This example shows how two separate "built-in" triggers can work together in one trigger string. 
(Assumes that the sound and program functions are working.)
<br>
<pre>
Options: Custom. 

Trigger:
    Play wav file "alarm"  Run program "sendtext"
</pre>
<br>
Write a daily log file. (This may be redundant, as the MQTT plug-in has logging capability in other forms, but shows what can be done.)
<pre>
Options: Standard, All.
 
Trigger:
    write to file "filename%D" "%T: %O:%M"
</pre>
<br>
<br>
Write a daily log file, where the payload may have double quotes in it (i.e., a JSON string).
In that case, the double quotes must be converted to single quotes to avoid confusing the "write to file" processing.
<pre>
Options: Standard, All.
 
Trigger:
    write to file "filename%D" "%T: %O:%m"
</pre>
<br>
<i>The following examples require action plug-in version 1.3 or greater.</i>
<br>
<br>
Turn on/off other lights to same level as triggering device.
<br>
<pre>
Options: Standard, On/Off. 

On Trigger:
    action: x10 pcslevel c12 %P, x10 level a3 %X;

Off Trigger:
    action: x10 off c12, x10 off a3;
</pre>
<br>
Tune a TV to a channel using IR.
(Select cable input to TV, then tune a channel with a .5 second delay between digits.)
<br>
<pre>
Options: Standard, On/Off. 

On Trigger:
    action: ir transmit 28 1,ir transmit 114 1,wait for 500,ir transmit 116 1;

Off Trigger:
    {empty}
</pre>
<br>
Tune a TV to a channel on a streaming service.
(Turn on TV, .5 second delay for TV to turn on, select streaming device input, then select streaming channel.)
This example shows how two separate plug-in triggers (action and roku) can work together in one trigger string.
<br>
<pre>
Options: Standard, On/Off. 

On Trigger:
    action: ir transmit 2 1,wait for 500,ir transmit 26 1; roku: 13;

Off Trigger:
    {empty}
</pre>
Note: <i>roku</i> is a custom plug-in not generally available.

<h3>Sending/Receiving MQTT Messages from/to Another Plug-in</h3>
When total control for MQTT message processing is needed,
this method takes "Custom Commands" a step farther.
A plug-in can be essentially independent of processing that is done in the MQTT plug-in, except for using it as a MQTT transport mechanism.
<br>
<br>
Other plug-ins can interface with the MQTT plug-in via the <i>mqttComm</i> command.
The calling plug-in should import the command via:
<pre>
    hvImport mqttComm
</pre>
The <i>mqttComm</i> command has the following formats:
<pre>
    mqttComm {-log} sub|unsub &lt;<i>topic</i>&gt;|<i>topic</i> <i>callback</i> 
    mqttComm {-exactstat -nodim -log -retain} stat|cmnd &lt;<i>topic</i>&gt;|<i>topic</i> {<i>payload</i>}
    mqttComm {-log -retain} pub <i>topic</i> {<i>payload</i>}
</pre>
<ul>
<li>Note: In previous versions (&lt; 1.76), "type", the name of the calling plug-in, was the first argument.
This has been deprecated.
However, for backward compatibility,
new versions of the command will allow (and ignore) a "type" argument.
<li>-log: Log the command.
This argument is optional.
<li>-retain: The sent message is retained by the broker.
This argument is optional.
<li>-exactstat: Sends a POWER stat response regardless of power/result settings.
<li>-nodim: Don't include Dimming in a RESULT response.
<li><i>topic</i>: Can either be enclosed in "&lt;&gt;" (or any of the other standard forms) or not. If a standard form is used, MQTT's standard prefixes and the Power post-fix will be added to create a full topic.
Otherwise, a topic with neither "&lt;" nor "&gt;" is used as-is, without adding the standard pre- and post-fixes, essentially creating generic MQTT messages.
If the topic contains spaces, the topic along with any "&lt;" or "&gt;",
should be enclosed in double-quotes.
<li><i>payload</i>: The MQTT message payload to send and is valid only for "cmnd", "stat" and "pub" actions.
Double-quotes or braces are NOT necessary for any spaces in the payload portion.
<li><i>callback</i>: The name of a proc in the calling plug-in that will process the subscribed-to incoming message and is valid only for "sub" and "unsub". It will be called like this:
<pre>
    <i>callback fulltopic payload retain</i>
</pre>
<ul>
<li><i>fulltopic</i>: the complete topic, including any prefix or postfix.
<li><i>payload</i>: the message payload.
<li>When a new subscription is made by a client, all retained messages that match the full topic are reported with <i>retain</i> set to 1. Any messages matching the full topic that are subsequently received by the broker are reported with a value of 0.

</ul>
</ul>
See below for further details on the use of callbacks!
<br>
<br>
A "stat" or "cmnd", when used with a topic with neither "&lt;" nor "&gt;", ignores the "stat" or "cmnd" and is the same as using "pub".
<br>
<br>
A "pub" or "cmnd", when used with a standard topic, is the same as using "cmnd".
<br>
<br>
When the standard form is used with "sub" or "unsub", a "cmnd" pre-fix is assumed.
To subscribe or unsubscribe to a "stat" type topic, use the topic without the &lt;.
Example: To subscribe to a stat message:
<pre>
    mqttComm sub stat/utilityroom> cascallback
</pre>
which will subscribe to
<pre>
    stat/utilityroom/POWER
</pre>
with "cascallback" as the callback proc.
<br>
<br><b>Caution:</b>
Make sure when subscribing to a topic that the topic is unique, especially compared to <i>external devices</i> and <i>internal objects</i>.
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
<pre>
    mqttComm sub &lt;utilityroom&gt; cascallback       subscribes to cmnd/utilityroom/POWER with callback cascallback
    mqttComm unsub stat/utilityroom/power mycallback  unsubscribes to stat/utilityroom/power with callback mycalback
    mqttComm cmnd &lt;utilityroom&gt; on               publishes cmnd/utilityroom/POWER on
    mqttComm stat &lt;utilityroom&gt; on               publishes stat/utilityroom/POWER on and/or
                                                           stat/utilityroom/RESULT {"POWER":"ON","Dimming":100}
    mqttComm -nodim stat &lt;utilityroom&gt; on        publishes stat/utilityroom/POWER on and/or
                                                           stat/utilityroom/RESULT {"POWER":"ON"}
    mqttComm -exactstat stat &lt;utilityroom&gt; on    publishes stat/utilityroom/POWER on 
    mqttComm stat "cmnd/living room/state" on    publishes cmnd/living room/state on
    mqttComm pub "cmnd/living room/state" on     publishes cmnd/living room/state on
</pre>
<br>
<br>
<i>Callbacks with the "sub" form of mqttComm</i>
<br>
<br>
When something subscribes to a topic, the assumption is that some action should be taken when a message with that topic arrives.
So how is that handled when a plug-in uses <i>mqttComm</i> to subscribe to a topic?
<br>
<br>
This is best explained by example.
Let's assume a plug-in wants to subscribe to a topic:
<pre>
    mqttComm sub &lt;utilityroom&gt; cascb     
</pre>
This results in a subscription to
<pre>
    cmnd/utilityroom/POWER
</pre>
When received, this topic means either report back the current device state (with an empty or "?" payload),
or change the device state based on the payload.
That means that when the MQTT plug-in receives the message, it must communicate with the subscribing plug-in to complete the response.
It does this by calling the callback proc. Let's say some external entity wants to know the state of utilityroom.
When the mqtt plug-in received the appropriate message, it calls the callback proc like this:
<pre>
    cascb cmnd/utilityroom/POWER "?" 0
</pre>
A proc "cascb" must be defined in the calling plug-in to handle the message.
In this case, it should eventually use another mqttComm command to send back status, maybe like this:

<pre>
    mqttComm stat &lt;utilityroom&gt; On 50   ---&gt;   stat/utilityroom/POWER ON 50
</pre>
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
<h3>Other Public Procedures Supplied/Called by the MQTT Plug-in</h3>

<b>topicTemplate</b>
<br>
<br>
To help with parsing full topics,
the mqtt plug-in makes public a helper proc that will split a full topic (the topic returned to the callback) into its parts and provide other info about the full topic.
<br>
<br>
The calling plug-in should import the command via:
<pre>
    hvImport topicTemplate
</pre>
The <i>topicTemplate</i> command has the following format:
<pre>
    topicTemplate <i>topic</i>
</pre>
<ul>
<li><i>topic</i>: fulltopic received by the callback.
</ul>
topicTemplate returns a list containing a dict with the following key/values:
<br>
<br>
<table border="0" cellspacing="10">
 <tr>
  <th>Key</th>
  <th>Value</th>
 </tr>
 <tr>
  <td align="center">template</td>
  <td>One of &lt;T&gt;, &gt;T&lt;, &lt;&gt;T, T&lt;&gt;, &gt;&lt;T, T&gt;&lt;, &lt;T, &gt;T, T&gt;, T&lt;, T.</td>
 </tr>
 <tr>
  <td align="center">pre pos</td>
  <td>Prefix position 1-3, -1 if no prefix.</td>
 </tr>
 <tr>
  <td align="center">pre type</td>
  <td>Prefix string, "" if no prefix.</td>
 </tr>
 <tr>
  <td align="center">topic pos</td>
  <td>Sub-topic position 1-3</td>
 </tr>
 <tr>
  <td align="center">topic name</td>
  <td>Sub-topic string</td>
 </tr>
 <tr>
  <td align="center">topic type</td>
  <td>"s" (standard-contains both &lt; and &gt;), "ns" (non-standard) otherwise.</td>
 </tr>
 <tr>
  <td align="center">post pos</td>
  <td>Postfix position 1-3, -1 if no postfix.</td>
 </tr>
 <tr>
  <td align="center">post type</td>
  <td>Postfix string, "" if no prefix.</td>
 </tr>
 <tr>
  <td align="center">post ntype</td>
  <td>Post type with "Relay" number striped off.</td>
 </tr>
 <tr>
  <td align="center">post index</td>
  <td>"Relay" number or "[0-9]*" if none, or "" if no postfix.</td>
 </tr>
 <tr>
  <td align="center">match</td>
  <td>Regexp expression to match the full topic.</td>
 </tr>
</table>
<br>
<br>
There are some details and cautions concerning the use of this helper proc.
Contact the author for further support.
<br>
<br>
<b>mqttLog</b>
<br>
<br>
Puts an entry into the current MQTT log file.
<br>
<br>
The calling plug-in should import the command via:
<pre>
    hvImport mqttLog
</pre>
The <i>mqttlog</i> command has the following format:
<pre>
    mqttLog <i>string</i> {<i>color</i>}
</pre>
<ul>
<li>
<i>string</i>: String to log.
<li><i>color</i>: If "debug" is imported into the calling proc,
will send string to the debug plug-in in <i>color</i>, Default: red.
</ul>
<br>
<b>mqttReady</b>
<br>
<br>
This proc is <i>called</i> by the MQTT plug-in to indicate whether it is connected or disconnected to the MQTT broker (and hence ready to take mqttComm calls).
The using plug-in should make the command public via:
<pre>
    hvPublic mqttReady
</pre>
When the MQTT connection changes, the MQTT plug-in calls <i>mqttReady</i> like this:
<pre>
    mqttReady <i>status</i>
</pre>
<ul>
<li>
<i>status</i> is a dict  of either {state connected} or {state disconnected reason <i>reason</i>}.
Possible values for <i>reason</i> are:
<pre>
    0 Normal disconnect
    1 Unacceptable protocol version
    2 Identifier rejected
    3 Server unavailable
    4 Bad user name or password
</pre>
</ul>
Using this proc is only necessary to make sure the plug-in's subscriptions are re-done automatically in the case the connection to the broker going down and then recovers (with a {state connected} return).
<br>
<br>
Reasons "1", "2", and "4" are fatal and need to be corrected before a connection can be made.
<br>
<br>
To use, define an mqttReady proc to respond to the connected and/or disconnected states.
Typical use:
<pre>
    hvPublic mqttReady
    proc mqttReady {status} {
    
        if {[dict get $status state] eq "connected"} {
            #code to run when MQTT is ready.
        } else {
            #code to run when MQTT is not ready.
        }
    }
</pre>
<h3>MQTT Discovery for Home Assistant</h3>
<!--
<a href="HomeVision and Home Assistant.html">Tips for Interfacing HomeVision with Home Assistant</a>
<br>
<a href="HomeVision Discovery How-to.html">How to Use Home Assistant Auto Discovery</a>
<br>
-->
[[Tips for Interfacing HomeVision with Home Assistant|https://github.com/rebel7580/MQTT-Plug-in-For-HomeVisionXL/wiki/Tips-for-Interfacing-HomeVision-with-Home-Assistant]]
<br>
[[How to Use the MQTT Plug-in's Home-Assistant Auto Discovery|How-to-Use-the-MQTT-Plug-in's-Home-Assistant-Auto-Discovery]]
</body>
</html>
