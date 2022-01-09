<!-- $Revision: 1.15 $ -->
<!-- $Date: 2021/12/10 18:45:36 $ -->
<html>
<head>
  <title>Roku Plug-in</title>
</head>

<body style="" lang="EN-US" link="blue" vlink="purple">

<font color="#0000ff" size="12"><b>Roku Plug-in</b></font>
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

<h2>Serial Commands</h2>
<pre>roku: hvwxvars enable|disable</pre>
<h2>NetIO COmmands</h2>
<h2>MQTT></h2>
<br><br>
The Roku plug-in can be controlled by MQTT and can send MQTT status messages for the active App and the App list.
<br><br>

<h3>MQTT Setup</h3>
<br><br>
Setup is done in the <i>Settings</i> tab.
Enter the desired topic in the "MQTT Topic:" field.
If you place "<>" in the topic (e.g., <topic>), the standard MQTT format will be used.
This is the preferred way to enter topics, since this topic is used for both receiving command messages and sending state messages.
<br><br>
<h3>MQTT commands</h3>
Command payloads take the same format as serial and NetIO.

<h3>MQTT State</h3>
<br><br>
State information is sent in response to reception of the App list or the Active App.
<br><br>

The App List is NOT automatically queried, so the response comes only when the plug-in receives an App List query, or the user clicks "Update" in the <i>Apps</i> tab.
<br><br>
The plug-in queries for the active app in response to other commands, so it will send Active App responses at those times.
<br><br>
State information is sent in JSON format.
<br><br>
Active App response (formatted for readability):
<pre>
{
    "active-app": {
        "app":"NetFlix",
        "id":12
    }
}
</pre>
<br><br>
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

</body>
</html>
