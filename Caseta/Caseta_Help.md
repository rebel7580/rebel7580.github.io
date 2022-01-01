# Caseta Plug-in

[Back to Projects Index](/index)

[Back to Caseta Index](/Caseta/Caseta_index)

<!-- $Revision: 1.3 $ -->
<!-- $Date: 2019/11/08 20:22:51 $ -->
<html>
<body>
<h1>Lutron Caseta Plugin for HomeVisionXL</h1>
<h2>Overview</h2>
The Caseta Client plug-in provides a way to interface Lutron Caseta switches to HomeVision.
<h3>Getting Started</h3>
<b>This plug-in only works with the Lutron Smart Bridge Pro, NOT the non-Pro version!</b>
<br><br>
Caseta switches are automatically configured in the plug-in using a file obtained from the Lutron application.
<ol>
<li>Before you can set up the plug-in, you must configure the lights and scenes within the Lutron application.
The names you assign to your switches are the names you will use in the plug-in.
</li>
<li>
If you are using the iOS version of the Lutron application, you will need to access the OS settings and find the Lutron application. After you select the Lutron application, enable Advanced settings. Once you have enabled the Advanced settings go back to the Lutron application.
<br>
<br>
If you are using the Android version of the Lutron application, go to settings within the Lutron application and click the check box to show Advanced settings.
</li>
<li>
Choose Integration and then
"enable telnet support"
and then
"Send integration report".
</li>
<li>
The integration report will be sent to your email address that you have registered via the application. 
Copy the configuration and save it to a file on the computer running HomeVisionXL.
The default is to have it in your plugin directory with the name CasetaConfigFile.json.
The plug-in will read this file and pick up the switch names.
<br><br>
Note: If you subsequently add or remove any Caseta devices, you must generate a new integration report and create an up-to-date json file.
You must restart the Caseta plug-in to read any changes to the json file for them to take effect.
</li>
<li>
Find the IP of the Lutron Smart Bridge Pro. It might make sense to lock this IP in (make it static) on your router so it doesn't change.
</li>
<li>
Proceed to Caseta plug-in configuration by enabling the Caseta plug-in
and going to the Caseta entry in the HomeVisionXL <i>Plugins</i> menu.
</li>
</ol>
<br><br>
<h3>Settings Tab</h3>
<ul>
<li>
"Lutron Smart Bridge Pro IP Address" should be set to the Smart Bridge Pro's IP address from Step 5 above.
</li>
<li>
"Lutron Smart Bridge Pro Port" defaults to "23" (for a telnet connection) and probably won't need to change.
</li>
<li>
Fill in the "Username" and "Password".
Default values are "lutron" and "integration", respectively and probably won't need to change.
</li>
<li>
Make sure "Caseta Config File" points to the file in Step 4 above.
</li>
<li>
If logging is desired, check "Create log file".
Doing so will display an entry box for the folder for the logs.
Log files will be created in this folder.
You can enter the folder name directly, or click the "..." button to browse to or create it.
To keep file sizes to a reasonable length, a new file is created each day there is logging information.
The file name is "CasetaLogYYMMDD", with a ".txt" extension if HomeVisionXL is running in Windows.
</li>
<li>
If you want the Caseta switches to use MQTT, check "Enable MQTT".
You must have the MQTT plug-in enabled.
Switches will be subscribed using the MQTT plug-in's standard command/power topic using the switch's name as the sub-topic. E.g., 
<pre>
    cmnd/Dimmer 1/POWER
</pre>
</li>
<li>
"Netio string", "Serial string prefix string", and "Serial string terminator character(s)" are set to reasonable defaults and probably don't need to be changed, except in the rare case that they conflict with other plug-ins.
</li>
</ul>
<h3>Devices Tab</h3>
This tab displays a list of devices from the Caseta Config file.
Caseta switches are listed in the "Zone" dropdown section.
The "Devices" dropdown section includes other Caseta devices,
most notably the PICO remote.
(The Caseta Config File contains a "device" representing the Smart Bridge itself,
and has 100 buttons defined for it.
This "device" is supressed, at least until there is a definite use for it.)
<ul>
<li>
The name of the device for use by serial and NetIO commands is shown.
This name can only be changed via the Lutron Caseta application.
</li>
<li>
<i>ID</i> is the ID assigned by the Lutron Smart Bridge Pro.
</li>
<li>
The <i>Flag/Var</i> column shows the Flag (FL-#) or Variable (VA-#) assigned to the device.
If none is assigned, a "-" will show.
</li>
<li>
The <i>Macro</i> column shows the Macro(s) assigned to the device in the form {on macro#}/{off macro#}.
If no macro is assigned, a "-/-" will show. 
</li>
<li>
The <i>State</i> column will usually show "ON {level}" or "OFF", depending on the reported state of the device.
When the state has not (yet) been reported, a "--" will show. 
</li>
</ul>
<h4>Editing Zone/Device Settings</h4>
<ul>
<li>
Flag/Var, Macro and logging settings can be edited by clicking the "Edit" button.
<ul>
<li>
For each device, an HV Flag or Variable can be assigned. 
These can be used to reflect the state of the Caseta device in your schedule.
Assigning a Flag or Variable is optional.
There is no checking to make sure a Flag or Variable is not used more than once.
</li>
<li>
If a variable is selected, an additional option is available.
<ul>
<li>
"Use Variable as Flag" treats the variable as a flag with its value being either a "0" (off) or "1" (on). If the received value is 0, it represents "off"; all other values represent "on". The variable will be set  to "0" or "1" accordingly.
</li>
<li>
If "Use Variable as Flag" is NOT selected, then the received value is written as a 1-byte integer number to the specified Variable.
</li>
</ul>
<li>
Macros can be assigned for "On" and "Off" states.
The same macro can be assigned to both states, or different macros can be assigned.
You can also assign a macro to just one of the states, leaving the other set to "None".
When a device changes state, any Flag/Var assigned is always set before a macro is run.
</li>
<li>
Choose whether to log messages sent to or received from this device.
</li>
<li>
Click "OK" to accept edits or "Cancel" to discard edits.
</li>
</ul>
</li>
<li>
When "Done" is clicked, if there were changes, the client connection to the Lutron Smart Bridge Pro is restarted.
</li>
</ul>
Note 1: For PICO remotes, each remote has a further dropdown that shows the available buttons.
PICO remotes send out a "pressed" signal when the button is depressed, and a "released" signal when the botton is released.
In normal use, the "pressed" and "released" signals occur in pairs, so only the "pressed" signal will trigger an action in the plug-in.
(I.e., the "released" signal is ignored.)
Because of this, only an "ON" macro makes sense.
All the rest are disabled.
<br>
Note 2: Names for Buttons on PICO remotes are in the form "{n} {b}", where {n} is the PICO remote name and {b} is the button name.
<br>
Note 3: If MQTT is enabled, an MQTT "stat/{name}/POWER ON" message is sent when the button is pressed.
<br>
Note 4: For the same reason as in Note 1,
the right-click menu for Devices only has "On" enabled.
If clicked, this will simulate a button press,
which may result in the action of Note 3.

<h2>Controlling Devices</h2>
<h3>Device Display Area</h3>
Right-clicking on a device line in the Device tab
will bring up a menu from which you can select "Off", "On", or "Set to".
When clicked, the appropriate command is sent to the selected switch.
<br><br>
Note: The Lutron Smart Bridge Pro returns levels to two decimal places.
The plug-in will accept levels with decimals; however, they will be truncated to a whole (integer) number when sent to the switch.
<br><br>
<h3>MQTT Control</h3>
When MQTT is enabled, switches can be controlled by issuing the appropriate MQTT topic. E.g.,
<pre>
    cmnd/Dimmer 1/POWER on       Turns Dimmer 1 on to 100%
    cmnd/Dimmer 1/POWER 50       Turns Dimmer 1 on to 50%
    cmnd/Dimmer 1/POWER off      Turns Dimmer 1 off
    cmnd/Dimmer 1/POWER 0        Turns Dimmer 1 off
</pre>
The payload is case-insensitive.
<br><br>
Switch state can be retrieved with a payload of "?" or empty. E.g.,
<pre>
    cmnd/Dimmer 1/POWER          Returns Dimmer 1 state
    cmnd/Dimmer 1/POWER ?        Returns Dimmer 1 state
</pre>
When a switch changes state, it will automatically report its state. E.g.,
<pre>
    stat/Dimmer 1/POWER ON 50    Dimmer 1 is on at 50%
    stat/Dimmer 1/POWER OFF      Dimmer 1 is off
</pre>
<br><br>
<h3>Serial Control</h3>
Caseta switches can be controlled within a schedule via serial commands which take the form:
<pre>
    caseta: {device name} {command};
</pre>
"caseta:" is whatever the "Serial string prefix" is defined as.
":' is whatever the "Serial string terminator character(s)" is defined as.
<i>device name</i> can contain spaces and is <i>case-sensitive</i> when matching a <i>name</i> in the Caseta Configuration file.
<i>command</i> can be "off", "on", "on 0-100" or 0-100 and is <i>case-insensitive</i>.
A value of 0 is equivalent to "off"; 100 is equivalent to "on".
<br>
<br>
For example, to turn on a switch "Dimmer 1",
the serial command would be:
<pre>
    caseta: Dimmer 1 on;
</pre>
or equivalently,
<pre>
    caseta: Dimmer 1 100;
</pre>
<h3>NetIO</h3>
Caseta switches can be controlled via NetIO using a "netioaction" command in the NetIO application and take the form:
<pre>
    netioaction caseta [device name] [command]
</pre>

"caseta" is whatever the "Netio string" is defined as.
<i>device name</i> can contain spaces and is <i>case-sensitive</i> when matching a <i>name</i> in the Caseta Configuration file.
<i>command</i> can be "off", "on", or 0-100 and is <i>case-insensitive.</i> 
A value of 0 is equivalent to "off"; 100 is equivalent to "on".
<br>
<br>
For example, to turn on a switch named "Dimmer 1",
the button's <i>sends</i> attribute would be set to:
<pre>
    sends:  netioaction caseta Dimmer 1 on
</pre>
The state of the device can be retrieved by:
<pre>
    reads:  get caseta {status_type} [device name]
</pre>
The <i>get</i> command will return data depending on <i>status_type</i>.
<i>Status_type</i> can be one of the following: "state", "status", "slider", "name", "namestate", "namestatus", or "image".
If <i>status_type</i> is not present, it defaults to "status".
<i>Status_type</i> is case insensitive.
<br>
<br>
<i>Status_type</i> determines the output format as follows (Similar to the X10 and Custom Light formats provided by the NetIO plug-in):
<ul>
<li>state
- [Off | On]
</li>
<li>status
- [Off | On [level]%]
</li>
<li>slider
- [level]
</li>
<li>name
- [device name]
</li>
<li>namestate
- [device name]: [Off | On]
</li>
<li>namestatus
- [device name]: [Off | On [level]%]
</li>
<li>image
- [gray | green]
</li>
</ul>

An empty string is returned if the current state is unknown.
<br>
<h2>Responding to Switch State Changes</h2>
The plug-in will update switch states in the Device Tab when switches change state.
If a Flag/Variable or Macro is defined, that object will be set accordingly.
<br>
<br>
If MQTT is enabled, a "stat" message will be sent with the new state info.
If the MQTT plug-in has such a "stat" message defined in its "Ext Devices" section,
it will respond to the "stat" message and execute any actions that may be defined in that plug-in. 
<h2>MQTT Discovery for Home Assistant</h2>
The Caseta plug-in supports switch and dimmer discovery for Home Assistant. For details, see the MQTT help at:

[How to Use the MQTT Plug-in's Home-Assistant Auto Discovery](/MQTT/HomeVision_Discovery_How-to)