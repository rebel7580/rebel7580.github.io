# MQTT Event Log Helper Plug-ins

[Back to Projects Index](/index)

## Overview

MQTT helper plug-in, *MQTT_evlog.hap*,
provides an MQTT interface to the *Netio_evlog.hap* helper plug-in,
which in turn provides an interface to the HomeVision Event Log.

The MQTT helper plug-in was designed with Home Assistant in mind but is useful in other use cases as well.
Control of the Event Log is accomplished via several MQTT messages and results are reported via four MQTT messages. MQTT payloads for HA are max 256 characters, so the 20-line log view is broken into 4 separate MQTT messages.

## Command Messages

```
cmnd/HVLog/update      # refresh to the latest log

cmnd/HVLog/get last
cmnd/HVLog/get first
cmnd/HVLog/get next
cmnd/HVLog/get prev
```
* *update*: refreshes to the latest log entries. This takes some time. While retrieving the log information, progress stat messages are sent aput every 10 seconds. The "stat/HVLog/part1" response will
contain a % complete message, while the other three will be five "blank" lines. 
Once the log retrieval is complete, the last 20 lines of the log will be sent (as if a ```cmnd/HVLog/get last``` command had been sent.
* *last*: return last 20 lines of log. 
* *first*: return first 20 lines of log. 
* *next*: return next 20 lines of log (relative to what was sent previously). 
* *prev*: return previous 20 lines of log (relative to what was sent previously). 

## Response Messages

```
stat/HVLog/part1 {log lines 1-5}
stat/HVLog/part2 {log lines 6-10}
stat/HVLog/part3 {log lines 11-15}
stat/HVLog/part4 {log lines 16-20}
```




Notes:
* The NetIO screens were the model for creating the other screens.
* The Home Assistant screens could be made better looking with some more effort, but they are functionally equivalant (or better) than the NetIO ones.
* The MQTT Dash screens have less capability for customizing the 'look and feel' but, even so, these examples may not take advantage of possible customizations.

