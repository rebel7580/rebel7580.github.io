<!-- $Revision: 1.15 $ -->
<!-- $Date: 2021/12/10 18:45:36 $ -->

[Back to Projects Index](/index)

[Back to Roku Index](/Roku/Roku_index)

<font color="#0000ff" size="12"><b>Roku Plug-in</b></font>
<h2>Overview</h2>
The Roku plug-in provides an interface to Roku streaming devices.
This plug-in was tested with a Roku 3 streaming device.

Some commands for Roku Tvs have been included but not tested.
<h2>Configuration</h2>
<h3>Settings Tab</h3>
<ul>
<li>
"Roku IP address" "Roku port number" should be set to the Roku's IP address and port.
If "Use discovered Roku IP" is checked, the plug-in will search for a Roku device and automatically populate the IP address and port.
</li>
<li>
"Netio string", "Serial string prefix string", and "Serial string terminator character(s)" are set to reasonable defaults and probably don't need to be changed, except in the rare case that they conflict with other plug-ins.
</li>
<li>
If you want to use MQTT, add a topic.
You must have the MQTT plug-in enabled.
See <i>MQTT Setup</i> for details.
</li>
</ul>
<h3>Apps Tab</h3>
This tab shows the current Applications installed on the Roku and the currently active Application.

The list is populated when "Update" is clicked
or an Apps query is received via serial, NetIO, or MQTT.

You can "tune" to an Application by right-clicking on an App and clicking "Tune".
A command will be sent to the Roku to launch that App.
<h2>Serial Commands</h2>
You can send serial commands to the Roku plug-in from your schedule or from other plug-ins.
<pre>
    roku: *command*;
</pre>

*command* can be any of the following:
<dl>
<dt>An App ID:</dt>
<dd>The numeric Application id.</dd>
<dt>A Keypress:</dt>
<dd>One of: Home,
    Rev,
    Fwd,
    Play,
    Select,
    Left,
    Right,
    Down,
    Up,
    Back,
    InstantReplay,
    Info,
    Backspace,
    Search,
    Enter,
    FindRemote,
    VolumeDown,
    VolumeMute,
    VolumeUp,
    PowerOff,
    ChannelUp,
    ChannelDown,
    InputTuner,
    InputHDMI1,
    InputHDMI2,
    InputHDMI3,
    InputHDMI4,
    InputAV1
</dd>
<dt>A Query:</dt>
<dd>One of: apps,
    active-app,
    media-player,
    device-info,
    icon/{id},
    tv-channels,
    tv-active-channel
</dd>
</dl>
Notes: 
<ul>
<li>Keypress and Query payloads are case sensitive.</li>
<li>Of the queries, only "apps" and "active-app" currently return a response. See <i>MQTT State</i>.</li>
</ul>

Examples:

Launch the NetFlix channel (12):
<pre>
    roku: 12;
</pre>

Query for the active app:

<pre>
    roku: active-app;
</pre>


<h2>NetIO Commands</h2>
<pre>
    netioaction roku *command*
</pre>

*command* is the same as for serial.

<h2>MQTT</h2>
The Roku plug-in can be controlled by MQTT and can send MQTT status messages for the active App and the App list.

<h3>MQTT Setup</h3>
Setup is done in the <i>Settings</i> tab.
Enter the desired topic in the "MQTT Topic:" field.
If you place "<>" in the topic (e.g., &lt;topic&gt;), the standard MQTT format will be used.
This is the preferred way to enter topics, since this topic is used for both receiving command messages and sending state messages.
<pre>
    cmnd/roku/POWER *command*
    stat/roku/POWER *state info*
</pre>

In fact, if "POWER" seems an odd postfix for a streaming device, then you can define a postfix explicitly while preserving the standard prefixes.
A topic defined like this:
<pre>
    &lt;roku/State
</pre>
would receive commands like this:
<pre>
    cmnd/roku/State *command*
</pre>
and state responses like this:
<pre>
    stat/roku/State *state info*
</pre>

<h3>MQTT commands</h3>
<pre>
    cmnd/roku/POWER *command*
</pre>

Command payloads take the same format as serial and NetIO.

<h3>MQTT State</h3>
State information is sent in response to reception of the App list or the Active App.
<br><br>
The App List is NOT automatically queried, so the response comes only when the plug-in receives an App List query, or the user clicks "Update" in the <i>Apps</i> tab.
<br><br>
The plug-in queries for the active app in response to other commands, so it will send Active App responses at those times.

State information is sent in JSON format.

Active App response (formatted for readability):

<pre>
{
    "active-app": {
        "app":"NetFlix",
        "id":12
    }
}
</pre>

 App list response (formatted for readability):

<pre>
{
    "apps": {
        "Vudu Movie & TV Store": 
            "id": 31012,
            "type": "menu"
        },
        "Prime Video": {
            "id": 13,
            "type": "appl"
        },
        "Netflix": {
            "id": 12,
            "type": "appl"
        },
        "HBO Max": {
            "id": 61322,
            "type": "appl"
        }
    }
}
</pre>

