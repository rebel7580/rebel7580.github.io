<!-- $Revision: 1.4 $ -->
<!-- $Date: 2015/02/06 02:47:47 $ -->
<html>
<head>
  <title>Weather Plug-in - Weather Websockets</title>
  <link rel="prev" href="wxtriggers">
  <link rel="next" href="custom">
</head>

<body style="" lang="EN-US" link="blue" vlink="purple">

<font color="#0000ff" size="12"><b>Weather Info on Web Pages using Websockets</b></font>
<br><br>
While weather data can be displayed in web pages using
&lt;HV:Run </i>&gt;
and
&lt;HV:Image </i>&gt; tags
(See <a href="wxtriggers.html">Triggering Weather Fetches</a>),
the weather data is determined at the time the web page is sent to the browser and is static thereafter.
The Weather plug-in can send "real-time" weather data to web pages via the Web plug-in's websockets.
<br><br>
Web plug-in version 7.2 or later is required to support websockets.
See the "<b>Real-time Updates of Web Pages</b>" help page in the Web plug-in for more details on websockets.
<br><br>
An additional supplementary plug-in, <i>wxjsondata.hap</i>, provides support for websockets.
If websocket support is desired, this plug-in should be enabled.
Also, if other custom objects are supported, then <i>wxjsondata.hap</i>  should be enabled.
Otherwise, it can remain disabled.
<br><br>
An additional supplementary JavaScript file containing websocket setup and weather helper functions, <i>wxwebsocket.js</i>,
is also required.
This file is placed in  the plugin/html directory when the weather plug-in is installed.
If your web server uses a directory other than plugin/html to store its web files,
you must manually move <i>wxwebsocket.js</i> to that directory.
<br><br>
Like the Web plug-in's "real-time" support for HomeVision objects,
to get weather data in "real-time", the following is needed:
<ul>
<li>
One or more HTML tags that define the desired data.
</li><li>
A Script tag that sources the weather websocket functions (<i>wxwebsocket.js</i>).
</li><li>
A JavaScript array that describes how to handle each tag.
</li></ul>
Example:
<pre>
    &lt;center&gt;
    &lt;h3&gt;Weather&lt;/h3&gt;
    &lt;HV:Image png WeatherGetIcon id="icon"&gt;
    &lt;br&gt;
    &lt;span id="wind"&gt;&lt;HV:Run WeatherGet Wind&gt;&lt;/span&gt;
    &lt;br&gt;
    &lt;span id="fcloc"&gt;&lt;HV:Run WeatherGet FcLoc&gt;&lt;/span&gt;
    &lt;br&gt;
    &lt;span id="fcday1"&gt;&lt;HV:Run WeatherGet FcDay1&gt;&lt;/span&gt;
    &lt;br&gt;
    &lt;HV:Image png WeatherGetIcon FcImg1 id="iconfc1"&gt;
    &lt;br&gt;
    &lt;span id="FcSc1"&gt;&lt;HV:Run WeatherGet FcSc1&gt;&lt;/span&gt;
    &lt;/center&gt;
    &lt;script src="wxwebsocket.js" type="text/javascript"&gt;&lt;/script&gt;
    &lt;script type="text/javascript"&gt;
    if (typeof(hvobjs) === "undefined") {hvobjs = {}}
    hvobjs["wx"] = [
        {
            "element": document.getElementById("icon"),
            "index": 0, "function": wxf, "type": "icon"
        }, {
            "element": document.getElementById("wind"),
            "index": 0, "function": wxf, "type": "wind"
        }
     ]
     hvobjs["fc"] = [
         {
            "element": document.getElementById("fcloc"),
            "index": 0, "function": fcf, "type": "loc"
         }, {
            "element": document.getElementById("fcday1"),
            "index": 1, "function": fcf, "type": "day"
         }, {
            "element": document.getElementById("iconfc1"),
            "index": 1, "function": fcf, "type": "img"
         }, {
            "element": document.getElementById("FcSc1"),
            "index": 1, "function": fcf, "type": "sc"
         }
     ]
     &lt;/script&gt;
</pre>
Let's go through this rather simple example.
<br><br>
The first part has a series of HTML tags that will show the current weather icon and the current wind,
followed by the forecast location, "day 1" day name, weather icon, and the short weather description.
Note each HTML tag of interest has a unique ID.
The &lt;HV:Image&gt; tag will generate an HTML &lt;img&gt; tag with the included "id".
The other &lt;HV&gt; tags need a surrounding HTML tag.
In this case, a simple &lt;span&gt; tag will do.
<br><br>
Technically, when using websockets to provide weather data, the  &lt;HV&gt; tags
are not necessary.
An "empty" HTML tag like this:
<pre>
    &lt;span id="wind"&gt;&lt;/span&gt;
</pre>
would display the right weather text via websockets.
However, it is probably a good idea to include the corresponding &lt;HV&gt; tag
so that some information will be shown even if a browser that doesn't support websockets is used.
<br><br>
Next, the <i>wxwebsocket.js</i> script is sourced.
This JavaScript file sets up the websocket and provides weather helper functions.
<br>
Note: weather objects and standard HomeVision objects (e.g., "var") can be mixed in a web page, as long as
<pre>
    &lt;script src="wxwebsocket.js" type="text/javascript"&gt;&lt;/script&gt;
</pre>
comes AFTER
<pre>
    &lt;script src="hvwebsocket.tcl" type="text/javascript"&gt;&lt;/script&gt;
</pre>
Lastly, the hvobjs array is set up with an entry for each weather item.
There are two separate "sub-arrays", one for current weather info - "wx", and one for forecast info - "fc".
<ul>
<li>
The "element" part identifies a specific HTML tag, according to the tag's "id".
</li><li>
The "index" is always "0" for current weather items. For forecast information,
index "0" contains general information items for the forecast (e.g., location).
Index "1" through "7" contain the day-specific forecast information for day "n".
</li><li>
The "function" and "type" parts specify the weather helper function and its "sub-function".
The helper function retrieves specific weather information from data received over the websocket.
</li></ul>
<b>Weather Helper Functions</b>
<br><br>
The weather helper functions are divided into two main functions: "wxf" for current weather, and "fcf" for forecast weather.
Each has its own "sub functions", which are specified in the "type" field.
Note that when specifying the helper function in the "function" field of the array, it is NOT quoted.
However, the "sub function" specified in the "type" filed IS quoted.
<br><br>
<b>Selecting Weather Items and Corresponding Helper Functions</b>
<br><br>
Possible weather items for web use are the same as those for the Control plug-in. In the same way as for the Control plug-in,
a subset of items can be selected for web use,
but independently of those selected for the Control plug-in.
Since there are a lot of possible weather items,
some of which can be very wordy,
it is a good idea to minimize the amount of data transmitted over the websocket by selecting only those items actually needed by the web pages.
See <a href="controlvars.html">Using Weather Control Variables</a> for how to set up which items to send over the websocket.
<br><br>
To use a weather item on a web page,
you must configure its corresponding array entry to use the right helper function and sub function.
<br><br>
For current weather, "function" is "wxf" and the sub function ("type") is as follows:
<ul>
<li>
For items in <a href="controlvars.html"><b>Table 2</b></a>
the sub function ("type") is exactly the name of the variable,
but <i>in lower case</i>.
One exception is "WeatherIcon", which uses the sub function "img".
The "index" is always "0".

Example: <i>TempC</i> would be
<pre>
    "index": 0, "function": wxf, "type": "tempc"
</pre>
</li></ul>

For forecast weather, "function" is "fcf" and the sub function ("type") is as follows:
<ul>
<li>
For items in <a href="controlvars.html"><b>Table 3</b></a> that have no day number,
the sub function ("type") is the name of the variable <i>without the leading "fc"</i>, and <i>in lower case</i>.
The "index" is "0".

Example: <i>Fcloc</i> would be
<pre>
    "index": 0, "function": fcf, "type": "loc"
</pre>
</li><li>
For items in <a href="controlvars.html"><b>Table 3</b></a> that have a day number,
the sub function ("type") is  the name of the variable <i>without the leading "fc"</i> and <i>without the trailing day number</i>,
and <i>in lower case</i>.
The "index" is "<i>n</i>". where <i>n</i> is the day number.
<br><br>
Example: <i>Fcdayn2</i> would be
<pre>
    "index": 2, "function": fcf, "type": "dayn"
</pre>
</li><li>
When using the NWS as a source, both "day" and "night" forecasts are available for any specific day.
However, sometimes only one of the "day" and "night" sets of infomation
for the first or last day are provided.
For example, a weather fetch in the evening will usually have only the "night" forecast for that day.
For these situations, a set of forecast sub functions are available that choose the best information
(either the day or the night data) for a specific day:
"bestday", "bestsc", "bestlc", "bestpop", "bestimg".
<br><br>
Example: To get either the day or night icon for day one, depending on which was available:
<pre>
    "index": 1, "function": fcf, "type": "bestimg"
</pre>
</li></ul>
<br><br>
<b>Changing Weather Settings Via a Web Page</b>
<br><br>
The <i>wxinvoke</i> function is used primarily with an HTML button or input object's "onclick" attribute.
<i>wxinvoke</i> sends an <i>WeatherSet</i> command through the websocket to the web server,
which in turn sends it to the Weather plug-in for execution.
Using <i>wxinvoke</i> allows web control of any action that <i>WeatherSet</i> can do.
Arguments for <i>wxinvoke</i> are the same as those used for the <i>WeatherSet</i> command.
See <a href="wxtriggers.html">Triggering Weather Fetches</a> for details.
<br><br>
Here's an example that has an entry box to put in the zipcode, a button to send the entry box's value,
and another button to trigger a new weather fetch with the new zipcode information.
<pre>
    &lt;form&gt;
    &lt;input id="szip" class="entry" type="text" name="setzip"/&gt;
    &lt;button class="button" onclick="return wxinvoke('ZipCode ' + this.form.setzip.value)"&gt;Set ZipCode&lt;/button&gt;
    &lt;/form&gt;
    &lt;button class="button" onclick="return wxinvoke('Go')"&gt;Update&lt;/button&gt;
    .
    .
    .
    &lt;script src="wxwebsocket.js" type="text/javascript"&gt;&lt;/script&gt;
    &lt;script type="text/javascript"&gt;
    hvobjs["wx"] = [
       {
           "element": document.getElementById("szip"),
           "index": 0, "function": wxf, "type": "zipcode"
       }
    ]
    &lt;/script&gt;
</pre>
<br>
<br>
<font color="#0000FF"><b>Next:</b></font><br>
<a href="custom.html">Custom Objects and Websockets</a><br>
<a href="netio.html">Weather Info for NetIO</a><br>
<font color="#0000FF"><b>See Also:</b></font><br>
<a href="index.html">Introduction to the Weather Plug-in</a><br>
<a href="webwx.html">Web-Based Weather Data</a><br>
<a href="controlvars.html">Using Weather Control Variables</a><br>
<a href="forecast.html">Forecast Examples</a><br>
<a href="hvwxvars.html">HomeVision Controller Weather Variables</a><br>
<a href="localwx.html">Local Weather Data Files</a><br>
<a href="speechwx.html">Speaking Weather Data</a><br>
<a href="wxtriggers.html">Triggering Weather Fetches</a><br>
<a href="disclaimer.html">Disclaimer</a><br>


