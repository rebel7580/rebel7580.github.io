<!-- $Revision: 1.8 $ -->
<!-- $Date: 2021/10/05 02:07:52 $ -->
<html>
<head>
  <title>Weather Plug-in - Triggering Weather Fetches</title>
  <link rel="prev" href="speechwx">
  <link rel="next" href="wxws">
</head>

<body style="" lang="EN-US" link="blue" vlink="purple">

<font color="#0000ff" size="12"><b>Triggering Weather Fetches</b></font>
<br><br>
The plug-in can triggered by the following methods:
<dl>
<dt><b>Timed</b></dt>
<dd>
Select the <a href="webwx"><i>Data Source</i></a> tab and make sure <i>Enable Fetch</i> is checked
and the <i>Read Interval</i> is set to the desire fetch interval (in minutes).
<i>Read Interval</i> determines how often the weather
plug-in will fetch the weather data.
<i>Read Interval</i> set to 0 disables timed fetches.
<br><br>
<b>This is the most common method to trigger weather fetches.</b>
<br><br>
</dd>
<dt><b>Via Serial Commands</b></dt>
<dd>
Weather related serial commands allow your HomeVision Controller schedule
to control how the Weather plug-in fetches and speaks the weather
and whether or not it loads the controller's weather variables.
<br><br>
To use a serial command to trigger a weather fetch,
send a serial command from the HomeVision Controller
with the text "weather: fetch bkgnd".
An example line in the schedule would be like this:
<pre>
Serial port 1 (Main serial port):
            Transmit string 'weather: fetch bkgnd'
</pre>
The Timed method can also be used simultaneously.
However,
a serial command triggered fetch will restart the <i>Read Interval</i> timer.
<br><br>
A serial command from the HomeVision Controller with the text "weather: speak now"
will speak the <i>previously</i> fetched weather
(it does <i>not</i> fetch new data).
An example line in the schedule would be like this:
<pre>
Serial port 1 (Main serial port):
            Transmit string 'weather: speak now'
</pre>
This serial command has no effect on the <i>Read Interval</i> timer.
</dd>
</dl>
<br>
<b> Complete List of Weather Related Serial Commands</b>
<br><br>
<dl>
<dt><pre>weather: fetch enable|disable</pre></dt>
<dd>
Enable/disable fetching of weather info.
(Sets/clears the <i>Fetch Enable</i> checkbox in the Configuration <a href="webwx"><i>Data Source</i></a> tab.)
No weather can be fetched by any means when disabled
</dd>
<br><br>
<dt><pre>weather: fetch bkgnd</pre></dt>
<dd>
Fetches new weather info in the background.
<br>Note: To speak the new weather with a serial command (see below) directly after this command,
a wait of a few seconds is necessary to bridge the time needed to fetch the weather information in the background.
Due to the amount of time it typically takes to retrieve data from the web,
an immediate fetch is not reliable as it may cause the serial command to time out.
Restarts the Read Interval timer.
<i>Fetch Enable</i> must be set.
</dd>
<br><br>
<dt><pre>weather: speak never|always|serialonly</pre></dt>
<dd>
Controls  speaking of the weather.
(Sets the <i>Never</i>, <i>Always</i>, or <i>Serial Cmd Only</i> radio button in the Configuration <a href="speechwx"><i>Speech</i></a> tab.)
</dd>
<br><br>
<dt><pre>weather: speak now</pre></dt>
<dd>
Speak the current weather.
<i>Always</i> or <i>Serial Cmd Only</i> must be set.
</dd>
<br><br>
<dt><pre>weather: hvwxvars enable|disable</pre></dt>
<dd>
Enable/disable loading of the HomeVision Controller weather variables when the weather is fetched.
(Sets/clears the <i>Enable Update</i> checkbox
in the Configuration <a href="hvwxvars"><i>HV Vars</i></a> tab.)
</dd>
<br><br>
<dt><pre>weather: station "<i>stationid</i>"</pre></dt>
<dd>
Set the Station ID for current weather. <i>stationid</i> <b>must</b> be contained within quotes.
</dd>
<br><br>
<dt><pre>
weather: current|forecast|both station "<i>stationid</i>"
weather: current|forecast|both zipcode "<i>zipcode</i>"
weather: current|forecast|both latlong "<i>lat,long</i>"
weather: current|forecast|both hvlatlong
</pre></dt>
<dd>
Set the current, forecast or both values for the given location type.
The location value <b>must</b> be contained within quotes.
Note for the latlong value, the latitude and longitude are separated with a "," but no spaces.
The location type is also selected as the current source.
</dd>
<br><br>
<dt><pre>
weather: current|forecast|both none
weather: current|forecast|both NWS
weather: current|forecast|both METAR
weather: current|forecast|both OpenWeather
</pre></dt>
<dd>
Set the current, forecast or both data sources.
The data source types are not case sensitive.
METAR is only valid for current weather.
</dd>
</dl>
<br>
<dt><b>Via Other Plug-ins</b></dt>
<dd>
A weather fetch can be triggered from other plug-ins
using the following code in the other plug-in:
<br><br>
<b>WeatherSetVar</b>
<pre>
WeatherSetVar <i>?0</i> | <i>1</i> ?<i>wx</i> | <i>fc</i> | <i>wxfc</i> | <i>fcwx</i>??
</pre>
Runs a weather fetch.
If a first argument exists and is not a zero, then the weather data is fetched immediately before returning.
if there is a second argument, it determines whether the current (wx), forecast (fc), or both (wxfc/fcwx) is fetched.
If there are no arguments, the fetch is run in the background for both current and forecast weather.
<br><br>
The plug-in must import the command via:
<pre>
hvImport WeatherSetVar
</pre>

<br><br>
<b>WeatherSet</b>
<pre>
WeatherSet ?<i>current</i> | <i>forecast</i> | <i>both</i>? \
    <i>Station</i> stationID ?stationname? | \
    <i>ZipCode</i> code | \
    <i>LatLong</i> {lat long | lat,long} | \
    <i>HVLatLong</i> | \
    <i>None</i> | <i>NWS</i> | <i>METAR</i> | <i>OpenWeather</i> | \
    <i>Go</i>
</pre>
Sets station information
and typically would be used before WeatherSetVar
to set the station information to be used for current and forecast fetches.
If <i>current</i>, <i>forecast</i> or <i>both</i> is not present as the first argument,
then the setting is applied for both current and forecast weather.
<i>WeatherSet Go</i> is identical to <i>WeatherSetVar 1 wxfc</i>.
<br><br>
The plug-in must import the command via:
<pre>
hvImport WeatherSet
</pre>
<br><br>
"WeatherSet Station" is the same as "WeatherSetStation".
"WeatherSetStation" remains available for backwards compatibility.
<br><br>
The Timed method can also be used simultaneously.
However, a plug-in triggered fetch will restart the <i>Read Interval</i> timer.
<br><br>
<dt><b>Via The Web</b></dt>
<dd>
Two additional commands are provided mainly to access weather information using <HV> tags in HTML pages.
However, they can also be used by other plug-ins, similar to the method in the previous section.
<br><br>
<b>WeatherGet</b>
<pre>
&lt;HV:Run WeatherGet ?Table? <i>weather_item</i> <i>?weather_item ...?</i>&gt;
</pre>
Returns weather information.
<br><br>
If "Table" is present, a table will be returned, with a table header and one row per weather item.
The table uses the same style as the built-in tables provided by the Web plug-in.
However, the style can be overridden by use of additional style definitions.
<br><br>
If "Table" is not present, weather data will be returned as a simple text string of concatenated weather items.
An exception is the weather item <i>weathericon</i>,
which will return an HTML &lt;img&gt; tag inserted in the string if other items are present.
<br><br><i>Weather_items</i> can be any combination of the items found in the <i>Control Variable</i>
column of <a href="controlvars"><b>Table 2</b></a> and <a href="controlvars"><b>Table 3</b></a>. The names are not case-sensitive when used in tags.
<br><br>
Weather items <i>WeatherIcon</i> and <i>FcImgn</i>,
will return HTML &lt;img&gt; tags.
If used in another plug-in (as opposed to within a web page), <i>WeatherGetIcon</i> should be used instead.
<br><br>
<b>WeatherGetIcon</b>
<pre>
&lt;HV:Image png WeatherGetIcon <i>?icon?</i>&gt;
</pre>
Returns weather icon image via an HTML &lt;img&gt; tag.
When used in a plug-in, <i>WeatherGetIcon</i> returns icon image data.
<br><br>
If <i>icon</i> is specified, that icon will be provided.
<i>Icon</i> may contain the  extension, or the extension may be omitted.
If omitted, it is assumed to be ".png".
If <i>icon</i> is not specified,
the weather icon for the current weather is provided.
<br><br>
<i>Icon</i> should be one of the icon names in the Icon column of <a href="hvwxvars"><b>Table 5</b></a> or one of the forecast icons.
<br><br>
This method is slightly different from
<pre>
&lt;HV:Run WeatherGet WeatherIcon&gt;
</pre>
In fact, the Run method simply determines the weather icon for the current weather and calls an Image WeatherGetIcon command with the explicit icon name.
The difference is only important if you use something similar to the example below.
<br><br>
<b>Using WeatherSetVar and WeatherSet with HV:Run Tags</b>
<br><br>
Tags using WeatherGet or WeatherGetIcon typically display data for the <u>current</u> weather
(i.e., the last weather info fetched).
They do not fetch new weather data on their own.
However, WeatherSetVar and WeatherSet can be used in a Run tag to get new data for specific stations.
<br><br>
Examples:
<pre>
&lt;HV:Run WeatherSet Station KNYC CentralPark&gt;
&lt;HV:Run WeatherSetVar 1&gt;
&lt;HV:Run WeatherGet Table location weather wind Temp weathericon&gt;
&lt;br&gt;
&lt;HV:Run WeatherSet Station PAWS Wasilla&gt;
&lt;HV:Run WeatherSetVar 1&gt;
&lt;HV:Run WeatherGet Table location weather wind Temp weathericon&gt;
</pre>

produces a table of info for Central Park, followed by one with info for Wasilla.
<br><br>
Notes:
<ul>
<li>Running WeatherSet and WeatherSetVar in a web page changes the weather info stored by the Weather plug-in,
as is the case for any other way to trigger weather fetches. If you are also using weather variables with Control Plug-in,
or sending weather info to the HomeVision controller, that data will be affected by these two commands.
</li><li>Usually, WeatherSetVar should be run with a "1" argument,
signaling WeatherSetVar to complete fetching weather info from the Internet <u>before</u> returning.
Normally, data is fetched from the Internet in the background. However, in the web page scenario,
new data may not arrive in time to be captured by a following Run tag.
</li></ul>

<br><br>
See the Web plug-in Help pages for more info on the Run and Image tags.
Web plug-on 4.0 or later is required for this functionality.
<br><br>
<dt><b>Via The NetIO Server Plug-in</b></dt>
<dd>
A weather fetch is triggered by the NetIO Server plug-in
by the <i>events</i> or <i>netioaction</i> commands:
<pre>
netioaction wx|fc|wxfc|fcwx update
</pre>
or
<pre>
events wx|fc|wxfc|fcwx
</pre>
For more details, see <a href="netio">Weather Info for NetIO</a>.
<br><br>
<dt><b>Via The Control Plug-in</b></dt>
<dd>
A weather fetch is triggered by the Control plug-in
automatically when the Control plug-in starts up.
This ensures that weather data variables in control screens are updated
as soon as the Control plug-in starts.

No user configuration is needed in either plug-in.
</dd>
<br><br>
<dt><b>Via MQTT</b></dt>
<dd>
Most setting can be set and weather fetches triggered via MQTT.
See the <b>MQTT Setup</b> section of <a href="hvwxvars">HomeVision Controller Weather Variables</a>
for details.
</dd>
</dl>
<br>
<br><font color="#0000FF"><b>Next:</b></font><br>
<a href="wxws">Weather Websockets</a><br>
<a href="custom">Custom Objects and Websockets</a><br>
<a href="netio">Weather Info for NetIO</a><br>
<font color="#0000FF"><b>See Also:</b></font><br>
<a href="index">Introduction to the Weather Plug-in</a><br>
<a href="webwx">Web-Based Weather Data</a><br>
<a href="controlvars">Using Weather Control Variables</a><br>
<a href="forecast">Forecast Examples</a><br>
<a href="hvwxvars">HomeVision Controller Weather Variables</a><br>
<a href="localwx">Local Weather Data Files</a><br>
<a href="speechwx">Speaking Weather Data</a><br>
<a href="disclaimer">Disclaimer</a><br>


