<!-- $Revision: 1.5 $ -->
<!-- $Date: 2015/02/06 02:47:47 $ -->
<html>
<head>
<title>
Weather Plug-in- Custom Objects
</title>
<link rel="prev" href="wxws">
<link rel="next" href="netio">
</head>
<body>
<font size=12 color="#0000FF"><b>Custom Objects</b></font>
<br>
<br>
In addition to the "wx" and "fc" weather objects, the weather plug-in (via <i>wxjsondata.hap</i>) also provides "hooks" to get status for other "custom" object types.

You can also provide your own custom objects via another "custom" plug-in.

Here's what is needed to do so:
<br>
<br>

The "custom" plug-in should provide the following:

<br>
<br>
<dl>
<dt>
<b>get_update Procedure</b>
</dt>
<br>
<dd>
The providing plug-in should have a public (via hvPublic) <b>get_update</b> procedure
that creates the object's state information.
If the plug-in sees an <i>events</i> command with the object name in it,
if will call <i>get_update</i>.
Additionally, when the  plug-in starts initially, it calls <b>get_update</b>
to quickly get the object's initial status.
<pre>
    <b>get_update</b> <i>{object_type}</i>
</pre>

Where:
<dl>
<dt>
<i>object_type</i>
</dt><dd>
If called <u>with</u> an <i>object_type</i>, <b>get_update</b> should create the object's state information
only if <i>object_type</i> matches one of the objects that it supports.
If called <u>without</u> an <i>object_type</i>, <b>get_update</b> should create the state information for all objects it supports.
</dd></dl>
Note 1: <i>object_type</i> must be a unique name.
<br>
Note 2: <b>get_update</b> doesn't return anything directly.
However, it should trigger a call to <b>wsupdate</b> for each object it supports.
See below for details of <b>wsupdate</b>.
<br>
Note 3: There can be more than one plug-in with a public <b>get_update</b> procedure.
Calling the procedure will actually run the procedure in each plug-in that published it (via hvPublic).
<br>
<br>
Here is a simple example, as used in the Weather plug-in:
{% raw %}
 <pre>
    hvPublic get_update
    proc get_update {{type wx}} {
        if {$type in "wx fc"} {WeatherSetVar}
    }
</pre>
{% endraw %}

In this case, <b>get_update</b> calls <b>WeatherSetVar</b>, another procedure within the Weather plug-in,
if the object type is "wx", "fc", or null.
<b>WeatherSetVar</b> will fetch weather info according to its configuration
and eventually call <b>wsupdate</b> for "wx" and/or "fc", accordingly.
Note that unless the object type is present and does NOT match "wx" or "fc", state data for both "wx" and "fc" will be generated if they are both enabled in the Weather plug-in, regardless of
which is requested. This is OK since the only harm is that the plug-in may track unneeded object information.
</dd></dl>
The "custom" plug-in should call the following:
<br>
<br>
<dl>
<dt>
<b>wsupdate Procedure</b>
</dt>
<dd>
This public procedure is made available in the <i>wxjsondata.hap</i> plug-in.
(It may also be available in other plug-ins that want object state data.)
The "custom" plug-in generating object state information should import (via hvImport) the <b>wsupdate</b> procedure.
The "custom" generating plug-in may call this procedure either because it was triggered by a previous call to <i>get_update</i>,
or because a object's state has changed and needs to be reported.
The <b>wsupdate</b> procedure takes the generated object state information
and stores it for later use.
<br>
<br>
Its format should be:
<pre>
    <b>wsupdate</b> <i>object_type state_value_list</i>
</pre>

Where:
<dl>
<dt>
<i>object_type</i>
</dt>
<dd>
The object type being reported.
</dd>
<dt>
<i>state_value_list</i>
</dt>
<dd>
This is a list of object state information. one entry for each valid ID.
Entries may be as simple as a numerical state value, e.g., "0" or "1" to indicate an object's state is clear or set,
or may be more complex structures like a list of key-value pairs, e.g., {"tempf": 29, "dewpointf": 27}
</dd>
</dl>
</dd>
<dt>
<b>Full Example of a Custom Object Plug-in</b>
</dt>
<dd>
Here's an example of a plug-in that uses all of the above procedures. It creates an object "dm"
which has only one ID ("0").
State information consists of two key-value pairs.
For demonstration purposes,
the values of these two keys are incremented each time the <i>get_update</i> procedure is called.
Once started, it will simulate periodic (5 minute) state changes.
{% raw %}
<pre>
# Sample custom object generator plug-in

hvImport debug
hvImport wsupdate

set Wx(L) 1
set Wx(T) 100

hvPublic get_update dm
proc dm {{type dm}} {
    global  Wx

    if {$type ne "dm"} {return}

    lappend list [format {{"location": "%s", "temperature": "%s"}} \
        $Wx(L) $Wx(T)]
    wsupdate dm $list

    incr Wx(L)
    incr Wx(T)
    after cancel dm
    after 300000 dm
}

</pre>
{% endraw %}

The dm object info can be added to the example from <a href="wxws">Weather Websockets</a>,
showing how weather and "custom" objects can be integrated into one web page (as can standard HomeVision objects):
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
    &lt;br&gt;
    dm T is &lt;span id="dmt"&gt;&lt;/span&gt;
    &lt;br&gt;
    dm L is &lt;span id="dml"&gt;&lt;/span&gt;
    &lt;/center&gt;
    &lt;script src="wxwebsocket.js" type="text/javascript"&gt;&lt;/script&gt;
    &lt;script type="text/javascript"&gt;
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
     hvobjs["dm"] = [
         {
            "element": document.getElementById("dmt"),
            "index": 0, "function": wxf, "type": "temf"
         }, {
            "element": document.getElementById("dml"),
            "index": 0, "function": wxf, "type": "location"
         }
     ]
     &lt;/script&gt;

</pre>
Note that for simplicity, this example uses the weather helper functions from <i>wxwebsocket.js</i>.
If the "custom" objects need unique helper functions,
then they should be included in a separate javascript file.
</dd>
</dl>
<br>
<br>
<font color="#0000FF"><b>Next:</b></font><br>
<a href="netio">Weather Info for NetIO</a><br>
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
<a href="disclaimer">Disclaimer</a><br>

