<!-- $Revision: 1.15 $ -->
<!-- $Date: 2021/12/10 18:45:36 $ -->
<html>
<head>
  <title>Weather Plug-in - Weather with MQTT</title>
  <link rel="prev" href="webwx">
  <link rel="next" href="forecast">
</head>

<body style="" lang="EN-US" link="blue" vlink="purple">

<font color="#0000ff" size="12"><b>Weather with MQTT</b></font>

<br><br>
The Weather plug-in can send via MQTT selected <i>current</i> and <i>forecast</i> weather items, if available in the retrieved weather data.
<h2>MQTT Setup</h2>
Setup is done in the <i>Ctrl/Web/MQTT Vars</i> tab.
See the <i>MQTT Setup</i> section of <a href="hvwxvars">HV Controller Weather Variables</a>
for details.
<br><br>
<b>Sending Weather Information</b>
<br><br>
Weather information can be sent via MQTT whenever a weather fetch is done.
To send weather information via MQTT, the Weather plug-in uses the defined <i>State Topic</i>.
<p>
Current and/or forecast weather is sent in separate MQTT messages with the same topic.
The payload is in JSON format.
<br><br>
Current weather example (formatted for readability):</p>
<pre>
{
    "wx": {
        "location":"Lincroft, New Jersey, US",
        "lat":40.36,
        "long":-74.13,
        "temp":"66 F (19 C)",
        "humidity":72,
        "pressure":"29.88\"",
        "weather":"Clear sky",
        "wind":"From the southwest at 3 MPH",
        "uvi":0,
        "sunrise":"06:50:04 AM",
        "sunset":"06:43:59 PM"
    }
}
</pre>
<p>
Forecast weather example (formatted for readability):
</p>
<pre>
{
    "fc": {
        "loc": "Lincroft, New Jersey, US",
        "time":"28 Sep 2021 03:33:16 PM",
        "day1": {
            "clouds": 76,
            "day": "Tue",
            "flhi": 73,
            "lc": "Moderate rain",
            "pop": 100
        },
        "day2": {
            "clouds": 1,
            "day": "Wed",
            "flhi": 65,
            "lc": "Clear sky",
            "pop": 5
        },
        "day3": {
            "clouds": 17,
            "day": "Thu",
            "flhi": 61,
            "lc": "Few clouds",
            "pop": 0
        }
    }
}
</pre>
<p>Note that in the forecast payload, there may be some general items, like "loc", and then forecast-day specific items. These are grouped under "day1", "day2", etc.. Only days that have Active variables will show up. For example, if you only made Active items for "Day2" and "Day4", there will be no "Day1", "Day3", "Day5", "Day6" or "Day7" in the JSON payload.
<br><br>
<b>Controlling Weather Settings</b>
<br><br>
To control weather settings via MQTT,
the Weather plug-in uses the defined <i>Command Topic</i>.
<br><br>
The payload to use is identical to the syntax of the "WeatherSetVar" and "WeatherSet" commands.
<br><br>
Examples, assuming the <i>Command Topic</i> is set to "cmnd/weather/set":
<br><br>
Set <i>Location Selection</i> to "HVLatLong":
</p>
<pre>
    Topic: cmnd/weather/set
    Payload: "WeatherSet HVLatLong"
</pre>
<p>Set <i>Location Selection</i> to "Lat/Long", with new values:</p>
<pre>
    Topic: cmnd/weather/set
    Payload: "WeatherSet LatLong 40 -74"
</pre>
<p>Set <i>Data Source</i> to "NWS" for both current and forecasts:</p>
<pre>
    Topic: cmnd/weather/set
    Payload: "WeatherSet NWS"
</pre>
<p>Set <i>Data Source</i> to "OpenWeather" for forecasts:</p>
<pre>
    Topic: cmnd/weather/set
    Payload: "WeatherSet forecast OpenWeather"
</pre>
<p>Trigger a Weather fetch in the background:</p>
<pre>
    Topic: cmnd/weather/set
    Payload: "WeatherSetVar"
</pre>
<p>See <a href="wxtriggers">Triggering Weather Fetches</a> for complete details of the WeatherSet and WeatherSetVar syntax.
</p>
<br>
<br>
<font color="#0000FF"><b>See Also:</b></font><br>
<a href="index">Introduction to the Weather Plug-in</a><br>
<a href="webwx">Web-Based Weather Data</a><br>
<a href="controlvars">Using Weather Control Variables</a><br>
<a href="forecast">Forecast Examples</a><br>
<a href="hvwxvars">HV Controller Weather Variables</a><br>
<a href="localwx">Local Weather Data Files</a><br>
<a href="speechwx">Speaking Weather Data</a><br>
<a href="wxtriggers">Triggering Weather Fetches</a><br>
<a href="wxws">Weather Websockets</a><br>
<a href="custom">Custom Objects and Websockets</a><br>
<a href="netio">Weather Info for NetIO</a><br>
<a href="disclaimer">Disclaimer</a><br>
</body>
</html>
