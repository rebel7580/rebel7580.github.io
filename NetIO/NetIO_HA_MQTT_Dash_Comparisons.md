# NetIO, Home Assistant and MQTT Dash Comparisons

[Back to Projects Index](/index)

## Quick comparison of selected screen shots showing similar functionallity for NetIO, Home Assistant and MQTT Dash

See the this document for screen shots: [NetIO HA Dash Comparison](https://github.com/rebel7580/img/blob/master/NetIO%20HA%20Dash%20Comparison.pdf)

Notes:
* The NetIO screens were the model for creating the other screens.
* The Home Assistant screens could be made better looking with some more effort, but they are functionally equivalant (or better) than the NetIO ones.
* The MQTT Dash screens have less capability for customizing the 'look and feel' but, even so, these examples may not take advantage of possible customizations.

## General Comparisons

<table border="1" cellspacing="0">

 <tr>
  <th width="10%">Feature/Function</th>
  <th width="30%">NetIO</th>
  <th width="30%">Home Assistant</th>
  <th width="30%">MQTT Dash</th>
 </tr>
 <tr>
  <td align="center">Connection to HomeVisionXL</td>
  <td align="center">TCP (Via NetIO plug-in)</td>
  <td align="center">MQTT (Via MQTT plug-in)</td>
  <td align="center">MQTT (Via MQTT plug-in)</td>
 </tr>
 <tr>
  <td align="center">Remote Connectivity</td>
  <td align="center">Port Obfuscation & uuencoded Username/Password</td>
  <td align="center">Various methods, varying from low security to high security</td>
  <td align="center">Must expose MQTT broker. However, supports SSL connections to broker. (MQTT broker configuration should enable SSL AND non-encrypted listners. The MQTT plug-in doesn't support SSL connections. Neither do most local IOT devices (e.g., Tasmota, unless using a custom-built binary). However, the Mosquitto broker can be set up to be SSL to the outside world, and not to local network (I think!).</td>
 </tr>
 <tr>
  <td align="center">Ease of screen design</td>
  <td align="center">Easy to Moderate</td>
  <td align="center">Large Learning Curve</td>
  <td align="center">Easy</td>
 </tr>
 <tr>

 <tr>
  <td align="center">Flexibility/Features</td>
  <td align="center">Medium</td>
  <td align="center">Very flexible and feature rich</td>
  <td align="center">Low</td>
 </tr>
 <tr>
  <td align="center">Configuration Method</td>
  <td align="center">Browser-based GUI Designer<br>Manual JSON editing</td>
  <td align="center">Browser or in-App</td>
  <td align="center">in-App only</td>
 </tr>
 <tr>
  <td align="center">Browser Access</td>
  <td align="center">No</td>
  <td align="center">Yes</td>
  <td align="center">No</td>
 </tr>
 <tr>
  <td align="center">Android App</td>
  <td align="center">Yes (but going away!)</td>
  <td align="center">Yes</td>
  <td align="center">Yes</td>
 </tr>
 <tr>
  <td align="center">iPhone App</td>
  <td align="center">Yes (but going away!)</td>
  <td align="center">Yes</td>
  <td align="center">No</td>
 </tr>
 <tr>
  <td align="center">Smartphone widgets</td>
  <td align="center">No</td>
  <td align="center">No</td>
  <td align="center">Yes</td>
 </tr>
 <tr>
  <td align="center">Hardware Requirements</td>
  <td align="center">Basic HomeVisionXL HW</td>
  <td align="center">Raspberry Pi or Docker on Linux computer. Several possibilities with varying complexity.</td>
  <td align="center">HW to support MQTT broker, which could be run on same HW as HomeVisionXL.</td>
 </tr>

</table>