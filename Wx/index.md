<!-- $Revision: 1.13 $ -->
<!-- $Date: 2021/10/05 02:07:51 $ -->
<html>
<head>
  <title>Weather Plug-in - Introduction</title>
  <link rel="next" href="webwx">
</head>

<body style="" lang="EN-US" link="blue" vlink="purple">

<font color="#0000ff" size="12"><b>Introduction to the Weather Plug-in</b></font>
<br>
<br>
<b>
NOTE: In Verson 5.3 and later, OpenWeather.org has been substituted for Weatherbug.
Any references to Weatherbug remaining in this help should be ignored.
</b>
<p>
The Weather plug-in retrieves weather data from either
the U.S. National Weather Service (XML or METAR formats)  
or OpenWeather.org (JSON format).
Current and forecast weather information can be retrieved and displayed.
Depending on the source selected, the plug-in can retrieve weather information
from any one of thousands of weather stations worldwide.
Retrieved weather data can be used in any of the following ways:
<ul>
<li>The Weather plug-in can send weather data to
the Control plug-in to display weather information on custom control screens.
<li>
The Weather plug-in can send a subset of the
 weather data to certain weather variables in the HomeVision controller.
<li>
The Weather plug-in can speak selected weather data using the wintts plug-in.
<li>
The Web Server plug-in can use weather data in custom HomeVisionXL Web pages.
<li>
Other custom plug-ins can access weather data.
</ul>
<p>
Weather data can be retrieved from a local file instead of the web,
as long as that file uses a similar format as the on-line data.
</p><p>
Control plug-in version
4.4 or later and HomeVision 2.1 or later are
required.
Web plug-in version 4.0 or later is required for web access of weather data.
NetIO Server plug-in verion 3.2 is required for NetIO access to weather data.
</p>

<br><font color="#0000FF"><b>Next:</b></font><br>
<a href="webwx">Web-Based Weather Data</a><br>
<a href="controlvars">Using Weather Control Variables</a><br>
<a href="forecast">Forecast Examples</a><br>
<a href="hvwxvars">HomeVision Controller Weather Variables</a><br>
<a href="localwx">Local Weather Data Files</a><br>
<a href="speechwx">Speaking Weather Data</a><br>
<a href="wxtriggers">Triggering Weather Fetches</a><br>
<a href="wxws">Weather Websockets</a><br>
<a href="custom">Custom Objects and Websockets</a><br>
<a href="netio">Weather Info for NetIO</a><br>
<a href="wxmqtt">Weather with MQTT</a><br>
<br><font color="#0000FF"><b>See Also:</b></font><br>
<a href="disclaimer">Disclaimer</a><br>

