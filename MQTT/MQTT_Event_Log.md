# MQTT Event Log Helper Plug-ins

[Back to Projects](/index)

[Back to MQTT Index](/MQTT/MQTT_index)

## Overview

MQTT helper plug-in, *mqtt_evlog.hap*,
provides an MQTT interface to the *netio_evlog.hap* helper plug-in,
which in turn provides an interface to the HomeVision Event Log.
You must have both *mqtt_evlog.hap* and *netio_evlog.hap* enabled.
You *do not* need the *NetIO plug-in* enabled for the MQTT part to work;
*mqtt_evlog.hap* and *netio_evlog.hap* run independently of the *NetIO plug-in*.

The MQTT helper plug-in was designed with Home Assistant in mind but could useful in other MQTT environments as well.
Control of the Event Log is accomplished via several MQTT messages and results are reported via four MQTT messages. MQTT payloads for HA are max 256 characters, so the 20-line log view (of a total of 256 lines for the complete log) is broken into 4 separate MQTT messages.
Commands allow for "scrolling" through the 256 lines, 20 lines at a time.

## Command Messages

```
cmnd/HVLog/update

cmnd/HVLog/get last
cmnd/HVLog/get first
cmnd/HVLog/get next
cmnd/HVLog/get prev
```
* *update*: refreshes to the latest log entries. This takes some time, as the entire log is read from the HomeVision controller. While retrieving the log information, progress stat messages are sent about every 10 seconds. The ```stat/HVLog/part1``` response will
contain a percent complete message, while the other three will contain five "blank" lines. 
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

Note:
* The payload for each stat message contains five lines of log information, each ending in a newline.
* All four messages are always sent.

## Home Assistant Configuration Setup for Log Messages

Create four sensors in configuration.yaml like this:

{% raw %}
``` yaml
sensor:
  - platform: mqtt
    unique_id: hv_log1
    name: "HV Log1"
    state_topic: "stat/HVLog/part1"
  - platform: mqtt
    unique_id: hv_log2
    name: "HV Log2"
    state_topic: "stat/HVLog/part2"
  - platform: mqtt
    unique_id: hv_log3
    name: "HV Log3"
    state_topic: "stat/HVLog/part3"
  - platform: mqtt
    unique_id: hv_log4
    name: "HV Log4"
    state_topic: "stat/HVLog/part4"
```
{% endraw %}

These sensor entities can be used in automations, dashboard GUIs, etc.

## Home Assistant Setup for Log GUI

Create a GUI. Example like this,
with corresponding yaml. Do this easily with the GUI editor, 
with a markdown card for the log itself, and three horizontal-stacks of two buttons each. (yaml is shown here just for detail.)

![Log GUI](HV_Log_GUI.gif)

{% raw %}
``` yaml
 - title: Log
    path: log
    visible:
      - user: xxxxxxxxxxxxxxxxxxxxxxx
    badges: []
    cards:
      - type: markdown
        content: >-
          ### HV Event Log

          {{ states('sensor.hv_log1') }}{{ states('sensor.hv_log2') }}{{
          states('sensor.hv_log3') }} {{ states('sensor.hv_log4') }} 
      - type: horizontal-stack
        cards:
          - type: 'custom:button-card'
            color_type: blank-card
          - type: button
            tap_action:
              action: call-service
              service: mqtt.publish
              service_data:
                retain: 0
                payload: ''
                topic: cmnd/HVLog/update
                qos: 0
            entity: sensor.hv_log1
            name: Update
            show_icon: false
            hold_action:
              action: none
      - type: horizontal-stack
        cards:
          - type: button
            tap_action:
              action: call-service
              service: mqtt.publish
              service_data:
                retain: 0
                payload: next
                topic: cmnd/HVLog/get
                qos: 0
            entity: sensor.hv_log1
            name: Next
            show_icon: false
            hold_action:
              action: none
          - type: button
            tap_action:
              action: call-service
              service: mqtt.publish
              service_data:
                retain: 0
                payload: prev
                topic: cmnd/HVLog/get
                qos: 0
            entity: sensor.hv_log1
            name: Previous
            show_icon: false
            hold_action:
              action: none
      - type: horizontal-stack
        cards:
          - type: button
            tap_action:
              action: call-service
              service: mqtt.publish
              service_data:
                retain: 0
                payload: first
                topic: cmnd/HVLog/get
                qos: 0
            entity: sensor.hv_log1
            name: First
            show_icon: false
            hold_action:
              action: none
          - type: button
            tap_action:
              action: call-service
              service: mqtt.publish
              service_data:
                retain: 0
                payload: last
                topic: cmnd/HVLog/get
                qos: 0
            entity: sensor.hv_log1
            name: Last
            show_icon: false
            hold_action:
              action: none
```
{% endraw %}
