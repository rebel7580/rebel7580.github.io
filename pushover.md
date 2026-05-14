<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<!-- $Revision: 1.3 $ -->
<!-- $Date: 2026/04/15 01:19:27 $ -->
<html>
<head>
  <title>Pushover Notification Service</title>
  <link rel="prev" href="index">
  <link rel="next" href="controlvars">
</head>

<body style="" lang="EN-US" link="blue" vlink="purple">

<font color="<br>0000ff" size="12"><b>Pushover Notification Service</b></font>
<p>
<br>The pushover plug-in provides notification service by communicating with the <a href="https://pushover.net">Pushover Service (https://pushover.net)</a>.
<p>
API and User keys must be obtained from Pushover and entered in the Configuration Screen.
<p>
Notifications can be sent via a serial command or by other plugins.
<h1>Serial Command</h1>
<pre>
    <i>serial_prefix</i> {options} message<i>serial_postfix</i>
</pre>
"Serial prefix string" and "Serial terminator characters" should be set in the Configuration Screen.
<p>
Serial command options and message adhere to the same format as that used by the procedure call from other plug-ins.
See below.
<br>
<br>
<b>Examples:</b> (assumes "Serial prefix string" is "pushover:", "Serial terminator characters" is ";")
<br>
<br>Send a message with default parameters (typical use).
<pre>
    pushover: Hello World!;
</pre>
<br>Send a message to specific devices and log:
<pre>
    pushover: -l -d bobsphone,wphone Hello World;
</pre>

<h1>pushover Procedure</h1>
Other plug-ins can send notifications using the <b>pushover</b> procedure.
<br>
<pre>   <b>pushover</b> { <b>-t</b> <i>title</i> | <b>-p</b> <i>priority</i> | <b>-d</b> <i>device1,device2</i> | <b>-s</b> <i>sound</i> | <b>-ttl</b> <i>seconds</i> | <b>-l</b> } <i>message</i>
</pre>
<dt><b>-t</b> <i>title</i></dt>
<br>
<dd>Title for message. If title is more than one word, enclose in double quotes or braces.
</dd>
<br><br>
<dt><b>-p</b> <i>priority</i></dt>
<br>
<dd>Priority of message. A value of -2, -1, 0 (default), 1, or 2{,r{,e}}.
<br>For priority 2, r = retry time (default: 60, min: 30), e = expire time (default: 1800, max: 10800)
</dd>
<br><br>
<dt><b>-d</b> <i>device(s)</i></dt>
<br>
<dd>List of devices to send to, comma separated if more than one. Default: all devices.
</dd>
<br><br>
<dt><b>-s</b> <i>sound</i></dt>
<br>
<dd>Notification sound. Default: "echo".
</dd>
<br><br>
<dt><b>-ttl <i>seconds</i></b></dt>
<br>
<dd>Time to live, after which the message will be deleted. Default: Device behavior for notifications.
</dd>
<br><br>
<dt><b>-l</b></dt>
<br>
<dd>Log this command to Pushover log.
</dd>
<br><br>
<dt><i>message</i></dt>
<br>
<dd>Message to send. Must come after all options. Does not need to be enclosed in quotes.
</dd>
</dl>
<br>Invalid options, along with any other options that follow, will be interpreted as part of the message.
<br>
<br>
<b>Examples:</b>
<br>
<br>Send a message with default parameters (typical use).
<pre>
    pushover Hello World! 
</pre>
<br>Send a message to specific devices and log:
<pre>
    pushover -l -d bobsphone,wphone Hello World
</pre>
<br>Send a message with emergency priority every 30 secs for 15 mins and log:
<pre>
    pushover -l -p 2,30,900 Water Leak Detected in Utility Room!
</pre>
<br>Send a message with emergency priority every 45 secs for 30 mins and log:
<pre>
    pushover -l -p 2,45 Water Leak Detected in Utility Room!
</pre>
</body>
</html>