<!-- $Revision: 1.1 $ -->
<!-- $Date: 2015/02/05 20:24:38 $ -->
<html>
<head>
  <title>Weather Plug-in - NetIO Interface</title>
  <link rel="prev" href="custom">

</head>

<body style="" lang="EN-US" link="blue" vlink="purple">

<font color="#0000ff" size="12"><b>Weather Info for NetIO</b></font>
<br><br>
The Weather plug-in supplies built-in "custom" procedures (netio, netioaction and get_update) to support NetIO. 
<h3>Triggering a Weather Fetch via NetIO</h3>
<p>
A weather fetch can be triggered in the NetIO application by the <i>netioaction</i> command:
</p>
<pre>
    <b>netioaction wx</b>|<b>fc</b>|<b>wxfc</b>|<b>fcwx</b> <b>update</b>
</pre>
<b>wx</b> fetches new current weather only, <b>fc</b> fetches new forecast weather only, <b>wxfc</b> or <b>fcwx</b>
fetches both (the order is not significant).
When this command is executed, the NetIO Server plug-in calls the <i>netioaction</i> procedure which the Weather plug-in recognizes.
The requested weather data is fetched immediately.
<br><br>Note 1: When <b>wx</b> or <b>fc</b> is used, older data for the opposite type may exist.
<br>Note 2: The command does not reset the current Read Interval.
<br><br>
This is the preferred way to explicitly update weather data.
<p>
Example:
<p>
Fetch the latest current and forecast data.
</p>
<pre>
sends: netioaction wxfc update
</pre>
<p>
The Weather plug-in can also fetch weather data 
by providing a weather specifier as one of the "objects" in the <i>events</i> command:
</p>
<pre>
    <b>events wx</b>|<b>fc</b>|<b>wxfc</b>|<b>fcwx</b>
</pre>
Other object types (standard or custom) can be combined with weather objects.
When this command is executed,
the NetIO Server plug-in calls the <i>get_update</i> procedure which the Weather plug-in recognizes.
Otherwise, this command works identically to the <i>netioaction</i> command above.

<h3>Getting Weather Items via NetIO</h3>
The Weather plug-in provides a <i>netio</i> procedure to return weather information in response to NetIO <i>get</i> commands.
<br><br>
Possible weather items for NetIO use are the same as those for the Control and Web plug-ins. 
However, there is no need to configure which items are available - all are.

<dl>
<dt>The <i>get</i> command for weather has the following format:
</dt>
<dd>
<pre>
<b>get</b> <b>wx</b>|<b>fc</b> <i>weather_item</i> ?<b>best</b>? ?;?<b>get</b>? <b>wx</b>|<b>fc</b> <i>weather_item</i> ?<b>best</b>?<i> ...</i>?
</pre>
</dd>
Where:
<dl>
<dt>
<b>wx</b>|<b>fc</b>
</dt>
<dd>
Choose <b>wx</b> or <b>fc</b>.
Either will work for both current or forecast weather items,
although for clarity you may want to use
<b>wx</b> for current weather items and
<b>fc</b> for forecast items.
</dd>
<dt>
<i>weather_item</i>
</dt>
<dd>Can be any item in <a href="controlvars"><b>Table 2</b></a>,
or <a href="controlvars"><b>Table 3</b></a>, but is case-insensitive.
</dd>
<dt>
<b>best</b>
</dt>
<dd>Optional. If present, must be <b>best</b>.
Only valid for FcDay, FcSc, FcLc, FcPop, FcImg items.
Ignored otherwise.

Both "day" and "night" forecasts are available for any specific day.
However, sometimes only one of the "day" and "night" sets of information
for the first or last day are provided.
For example, a weather fetch in the evening will usually have only the "night" forecast for that day.
For these situations, <b>best</b> chooses the best information
(either the day or the night data) for a specified day.
</dd>
</dl>
The <i>get</i> command may consist of several separate commands.
Multiple commands are separated with semi-colons.
The keyword <i>get</i> is only needed once but could be present for each command.
Each command can be a different object type.
<br><br>
Examples: 
<dl>
<dt>Get current temperature in Celsius (<i>TempC</i>).
</dt>
<dd>
<pre>
    reads:          get wx tempc 
    parseResponse:  \d+
    formatResponse: {0}&deg;

    Result:         21&deg; 
</pre>
</dd>
<dt>Get forecast location (<i>Fcloc</i>).
</dt>
<dd>
<pre>
    reads:          get fc fcloc

    Result:         New York, NY, US
</pre>
</dd>
<dt>Get forecast night day name for day 2 (<i>Fcdayn2</i>).
</dt>
<dd>
<pre>
    reads:          get fc fcdayn2

    Result:         Thu Night
</pre>
</dd>
<dt>Get forecast High and Low temps for day 3 (<i>FcHi3, FcLo3</i>).
</dt>
<dd>
<pre>
    reads:          get fc FcHi3; fc fclo3
    parseResponse:  [^|]+
    formatResponse: High/Low: {0}/{1}

    Result:         High/Low: 43/21
</pre>
</dd>
<dt>Get either the day (preferred) or night short forecast for day one, depending on which is available:
</dt>
<dd>
<pre>
    reads:          get fc fcday1 best; fc fcsc1 best
    parseResponse:  [^|]+
    formatResponse: {0}: {1}

    Result:         Wed Night: Mostly Cloudy
</pre>
</dd>
</dl>
</dl>
<i>A Note about weather icons</i>: Getting a weather icon (img) returns an <i>image name</i>, complete with its extension (i.e., cond001.png), not the image itself.
For the image to be displayed,
you must manually download image files from the appropriate weather web site
and pre-load them into the NetIO application's data space.
Since there are dozens of possible weather icons, getting icons may not be worth the effort.
<br>
<br>
<font color="#0000FF"><b>Next:</b></font><br>
<a href="wxmqtt">Weather with MQTT</a><br>
<font color="#0000FF"><b>See Also:</b></font><br>
<a href="index">Introduction to the Weather Plug-in</a><br>
<a href="webwx">Web-Based Weather Data</a><br>
<a href="controlvars">Using Weather Control Variables</a><br>
<a href="forecast">Forecast Examples</a><br>
<a href="hvwxvars">HomeVision Controller Weather Variables</a><br>
<a href="localwx">Local Weather Data Files</a><br>
<a href="speechwx">Speaking Weather Data</a><br>
<a href="wxtriggers">Triggering Weather Fetches</a><br>
<a href="wxws">Weather Websockets</a><br>
<a href="custom">Custom Objects and Websockets</a><br>
<a href="disclaimer">Disclaimer</a><br>


