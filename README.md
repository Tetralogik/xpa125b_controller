# XPA125B Amplifier Network & Serial Controller
Xiegu XPA125B (https://xiegu.eu/product/xpa125-100w-solid-state-linear-amplifier/) network control interface designed for a D1 Mini (https://www.wemos.cc/en/latest/d1/d1_mini.html). This allows you to use virtually any rig, including SDRs, with this amplifier. Both PTT and automatic band selection are supported. Although WiFi is required for the APIs you can also operate without network access in `yaesu`, `icom`, `sunsdr`, `elecraft` and `serial` mode. This would enable you to use a Yaesu, Icom, SunSDR or Elecraft rig and have the benefit of automatic band selection. The primary intended use of the controller is to hook directly into `rigctld` (https://hamlib.github.io/) meaning any rig which it supports will work (albeit requiring the use of WiFi). The latency is ~100ms in rigctl mode which is perfectly adequate for almost all digital work as well as phone. With modes which interface direct, such as Yaesu, SunSDR etc the latency is an order of magnitude less.

Written using the Arduino IDE. Required 3rd party libraries included for convience.

![assembled](controller_assembled.jpg)

![internals](controller-internals.jpg)

# Supported radios:

+ Yaesu (including 817/818)
+ Icom (via Bluetooth, such as the IC-705)
+ SunSDR (EXT CTRL port)
+ Elecraft (KX2 / KX3)
+ SparkSDR
+ Rigctld (any Hamlib compatible rig)

Rigctld support is the major feature which allows this to work with almost any radio, including SDRs such as the Flex 1500/3000, Hermes-Lite and ANAN based radios.

# Supported Amplifiers:

+ Xiegu XPA125B
+ Hardrock-50 (serial and Yaesu 817 mode)
+ MiniPA50 (and other Yaesu 817 compatible amplifiers)

Adding additional amplifiers is fairly trivial providing it uses either voltage or serial for band control and PTT is triggered by grounding an input.

# Supported APIs:

+ Serial
+ Web Interface
+ REST
+ MQTT

Even if a particular API isn't the current active mode for control purposes it can still be consumed for state. For example, providing MQTT is configured and enabled it will still publish events such as band/frequency and PTT changes. This allows the current state to be easily consumed by other MQTT compatible software. Similarly for REST, you can always `GET /state` to find out the current PTT state.

# Use Cases

This project has grown arms and legs and is now a powerful tool way beyond its initial intended use. The most useful feature is providing multiple APIs on top of rigctl to allow integration into almost any custom software.

+ Use any Yaesu radio with the amplifier (no WiFi required)
+ Use the Icom-705 via Bluetooth with the amplifier (no WiFi required)
+ Use a SunSDR via EXT CTRL port with a level converter (no WiFi required)
+ Use an Elecraft KX2/KX3 (no WiFi required)
+ Use any Hamlib compatible radio with the amplifier (no WiFi required)
+ Provide a Web/REST/MQTT/Serial API interface to rigctl
+ Allow automation via Node-RED to any hamlib compatible radio
+ Expand to support other radios and amplifiers if desired

# Bill of Materials

To Do

# Schematic

To Do

# Bluetooth module (HC-05)

To Do

# Configuration

The top of the file contains config options which need set first:

+ `wifi_enabled` - change to true if you want WiFi
+ `ssid` - your WiFi SSID
+ `password` - your WiFi password

If you want to enable MQTT set the following:

+ `mqtt_enabled` = true
+ `mqttserver` = <host>  (MQTT server hostname or IP address)
+ `mqttuser` = <username>  (MQTT username)
+ `mqttpass` = <password>  (MQTT password)
  
# Rigctl

To make rigctl default on boot first ensure WiFi credentials are filled in and also set:

+  `mode` = rigctl
+  `rigctl_default_enable` = true
+  `rigctl_default_address` = <IP address> (rigctl IP address)
+  `rigctl_default_port` = <port> (rigctl port)
  
As an example, if you want to use SDR Console as the rig, first enable CAT control and attach it to one end of a pair of virtual com cables, then run rigctld:

`rigctld.exe -r COM18 -m 2014 -s 57600 -t 51111`
  
 You need to ensure the port is allowed through the firewall.
  
 In the above we are using `COM18` as the other end of the virtual com cable. `2014` is the rig ID for a Kenwood TS-2000 which SDR Console emulates. The port `51111` is what you would then set `rigctl_default_port` to. This method also works with PowerSDR  and Thetis for Flex 1500/3000, Hermes-Lite and ANAN radios.
  
  Other software such as SparkSDR (https://www.sparksdr.com/) has rigctld built in so no need to run the daemon - simply point the controller to the IP/port of SparkSDR directly (you must enable CAT control). This is the ideal way to run the controller and perfect for radios such as the Hermes-Lite (http://www.hermeslite.com/).
  
 You can control basic functions of the rig ('rig' could also be SDR Console/SparkSDR etc) via the Serial, REST and MQTT APIs. Namely frequency, mode and PTT. For example, to set the current frequency of the rig via REST:
  
  `curl -s -d 'freq=7074000' http://xpa125b.local/setrigctlfreq`
  
 In essense this operates as an API translation layer to rigctl.
  
 If you want to use rigctl for band selection but prefer PTT to be via the control cable you can set `hybrid` to `true`.
 
 If you are using `rigctld` on Windows you need a recent version of Hamlib due to a bug I discovered while developing this controller. More details here: https://github.com/Hamlib/Hamlib/issues/873.
  
# Yaesu Mode

Most Yaesu radios (except the 817/818 - see below) use a similar method to SunSDR. There are four pins named Band A - D. We read the logic levels of the pins from the ACC port to determinte the band. PTT is handled via another pin called 'TX GND'.

Because the voltage on the pins are 5VDC we need to level shift them down to 3V3 so as not to damage the D1 Mini. An Arduino is 5VDC logic level so doesn't require such shifting. You only need to shift the band pins so a 4 way logic level shifter is perfect.
  
1. Wire up the ACC port using the Band pins (via a logic level shifter) and TX GND pin.
2. Configure the controller with `mode = "yaesu"` and ensure `hl_05_enabled = false` (no other config needed)

You can still enable WiFi and/or MQTT with this mode if desired. This allows you to have access to the other APIs and web interface aswel as push MQTT events but have control entirely handled locally.
  
# Yaesu 817 Mode

The Yaesu 817 & 818 radios use a different method for band selection. They have a stepped voltage output so we use the ADC in the D1 Mini to read this voltage and determine the band.

1. Wire up the ACC port using the band voltage pin and TX GND pin.
2. Configure the controller with `mode = "yaesu817"` (no other config needed)

# Icom IC-705

A popular QRP radio is the Icom-705 and the XPA125B makes a good companion in situations where you need extra power. The problem is only PTT works using a cable from the radio to the amplifier - no automatic band selection. 

## Solution 1

The preferred method of interfacing the controller with an IC-705 is to use the Blueooth feature built into the radio. With this method the radio presents the CI-V interface over bluetooth which connects to the HC-05 module of the controller. We use this to request the frequency (and thus band) from the radio. This method requires no network connectivity of the controller if you don't want or need access to the other APIs.
  
  1. Change the Bluetooth Data setting on the IC-705 to CIV Data (Echo Back).
  2. Connect the controller (name 'XPA128B') in the Bluetooth menu. Security code: 1234.
  3. Connect the control cable to the 3.5mm port on the IC-705 (this is used for PTT detection).
  4. Configure the controller with `mode = "icom"` and `hl_05_enabled = true` (no other config needed)
  
## Solution 2
  
This solution requires no cabling between the radio and the amplifier - you just need the controller and a computer. This works with any Hamlib supported radio including other Icoms and Kenwood, Yaesu etc.

To solve the lack of band selection you can utilise `rigctld` running on a computer. Most stations have a computer of some kind for logging etc so this feels an adequate solution. You can use either the built in `rigctl` mode or run the controller over serial using the method in the `Serial Example` secion below.

  1. Plug the 705 into the computer via USB.
  2. Run rigctld in a terminal (assumes Windows): `rigctld.exe -r COM3 -m 3085 -t 51111`.
  3. Configure the controller in `rigctl` mode with the IP address of the computer and port `51111`.
  
  Ensure the COM port is correct for your radio and you are using the latest version of Hamlib. The hamlib rig ID for the Icom 705 is `3085`.
  
  If you want to use the controller without network access you can use the example serial scripts under Linux. Although Windows has the Linux subsystem you can't access the serial ports (annoying). Here is how you could do this under any version of Linux (a Raspbery Pi would be perfect).
  
  1. Plug the 705 into the computer via USB.
  2. Plug the controller into the computer via USB
  3. Configure the controller with `mode = "serial"` (no other config needed)
  4. Find the serial device for the radio (`ls /dev/serial/by-id`)
  5. Find the serial device for the controller (`ls /dev/ttyUSB*`)
  6. In a terminal: `rigctld -r <path to radio serial device> -m 3085 -t 51111`
  7. In a terminal: `./amp_serial_control.sh localhost 51111 <path to controller serial device>`

That's it - you now have PTT and automatic band selection. For other software, such as WSJX-T, which needs control of the radio you can use `Hamlib` as the radio type and point it at rigctld using `localhost` port `51111`.

# SunSDR
  
The SunSDR radios provide an EXT CTRL port which can be used to signal band and PTT. X1 – X7 are programmable and X8 is always PTT. We will use X3 - X6 for band selection.

Because the voltage on the pins are 5VDC we need to level shift them down to 3V3 so as not to damage the D1 Mini. An Arduino is 5VDC logic level so doesn't require such shifting. You only need to shift the band pins so a 4 way logic level shifter is perfect.
  
1. Wire the EXT CTRL port to the D1 Mini via a logic level shifter for X3 - X6 and X8 directly onto the ptt_pin
2. Configure the controller with `mode = "sunsdr"` and ensure `hl_05_enabled = false` (no other config needed)

# Elecraft

Elecraft radios such as the KX2 and KX3 provide a serial interface for control. We can use this to request current frequency and band from the rig. You need to use a MAX3232 serial interface with the D1 then wire it up to the 3.5mm ACC1 port on the radio. The jack tip connection is RX data from the MAX3232 and ring is TX data to the MAX3232.
  
1. Connect the radio's ACC port to the controller (3.5mm to RS232)
2. Configure the controller with `mode = "elecraft"`, ensure `max3232_enabled = true` and `hl_05_enabled = false` (no other config needed)

# MQTT

As well as allowing control of the amplifier by publishing MQTT events the system is always updating current state to MQTT. You can use this feature to act as a bridge between Rigctl and MQTT even when no amplifier is involved. To do this:
  
  1. Configure MQTT server/username/password in the config (ensure `mqtt_enabled` is `true`)
  2. Configure Rigctl settings in the config and optionally make it the default mode
  
Now every state change will be published as an event to MQTT to be consumed by other software, such as Node-RED (https://nodered.org/) to trigger other actions. You could go wild and use this to have a big LED matrix display showing the current frequency and PTT state.
  
# Web Interface
  
A simple web interface is available on port 80 which allows access to basic functionality as well as documentation. This uses REST so anything which can be done here can also be achieved via the API.
  
![web](https://raw.githubusercontent.com/madpsy/xpa125b_controller/main/webinterface.png "wen")
  
# Valid serial commands (115200 baud):

+ serialonly [true|false] (disables every other mode and wifi entirely)
+ setmode [yaesu|yaesu817|icom|elecraft|serial|http|mqtt|rigctl|none]
+ setstate [rx|tx]
+ setband [160|80|60|40|30|20|17|15|12|11|10]
+ setfreq [frequency in Hz]
+ setrigmode mode=[rigmode] (USB/FM etc)
+ setmqtt [enable|disable]
+ setrigctl [address] [port]
+ setrigctlfreq freq=[frequency in Hz] (only available in rigctl mode)
+ setrigctlmode mode=[mode] ('mode' depends on radio - only available in rigctl mode)
+ setrigctlptt ptt=[0|1] (only available in rigctl mode)
  
Note: There are no commands to get current states over serial. The reason being all state changes are printed when they occur so you need to parse and track them as they appear.

# Valid HTTP POST paths:

+ /setmode mode=[yaesu|yaesu817|icom|elecraft|serial|http|mqtt|rigctl|none]
+ /setstate state=[rx|tx]
+ /setband band=[160|80|60|40|30|20|17|15|12|11|10]
+ /setfreq freq=[frequency in Hz]
+ /setrigmode mode=[rigmode] (USB/FM etc)
+ /setmqtt mqtt=[enable|disable] (only available via http)
+ /setrigctl address=[rigctl IP address] port=[rigctl port] (http only)
+ /setrigctlfreq freq=[frequency in Hz] (rigctl only)
+ /setrigctlmode mode=[mode] ('mode' depends on radio - rigctl only)
+ /setrigctlptt ptt=[0|1] (rigctl only)
  
Note: When you use `setfreq` the system will translate this into a band automatically. The idea is it makes it a lot easier than having to do the translation yourself.

# Valid HTTP GET paths:

+ /mode (show current mode)
+ /state (show current PTT state)
+ /band (show current band)
+ /frequency (show current frequency - must have been set)
+ /txtime (show tx time in seconds)
+ /txblocktimer (show tx countdown block timer in seconds)
+ /network (show network details)
+ /mqtt (show if mqtt is enabled - only available via http)
+ /rigctl (show rigctl server and performs connection test - only available via http)
+ /rigmode (show mode the rigctl radio is set to (FM, USB etc)
+ /status (show status summary in HTML - only available via http)

MQTT topic prefix is 'xpa125b' followed by the same paths as above (where the message is the values in [])

# Example API calls

Note: mDNS should be xpa125b.local

+ `curl -s -d 'mode=http' http://xpa125b.local/setmode`
+ `curl -s http://xpa125b.local/txtime`
+ `mosquitto_pub -h hostname -u username -P password -t xpa125b/setmode -m http`
+ `mosquitto_sub -h hostname -u username -P password -t xpa125b/txtime`

When `serialonly` is set to `true` neither http/mqtt (wifi is disabled) nor yaesu/icom/sunsdr/elecraft etc can be used. You can always use 'setmode' with serial/http/mqtt reguardless of current mode except when serialonly is enabled, in which case it only works via serial
  
Note: When using `setfreq` it automatically sets the correct band. Therefore, use either `setfreq` or `setband` but not both.

+ In yaesu mode only the Yaesu standard voltage input is used for band selection and rx/tx is only via the control cable
+ In icom mode only a Bluetooth attached Icom radio is used for band selection and rx/tx is only via the control cable
+ In sunsdr mode only the band data is used for band selection and rx/tx is only via the control cable
+ In elecraft mode we only accept band/freq selection and rx/tx via the serial port on the radio
+ In serial mode we only accept band/freq selection and rx/tx via serial
+ In mqtt mode we only accept band/freq selection and rx/tx via mqtt messages
+ In http mode we only accept band/freq selection and rx/tx via http messages
+ In rigctl mode we only accept band/freq selection and rx/tx via rigctl. You can control the rig itself in this mode via http/mqtt/serial. Server connection must succeed for this mode to activate.
+ In none mode then no control is possible
  
You can set `hybrid` to `true` which will use the control cable for PTT in every mode.

If MQTT is disabled and the mode is changed to MQTT then it will be automatically enabled.

# Serial Example

As mentioned previously you can run the controller without any network connectivity at all. In fact you can still interface with rigctl without network access by using the serial interface. Simply plug the controller into a USB port on the system you wish to run the scripts on.
  
Example scripts written in Bash showing how this may be achieved can be found in the `examples/serial` directory. In the configuration you may leave everything default (blank) and only change the default mode to serial (`mode = "serial"`). This isn't strictly neccessary as the script sets this when it starts just to be sure. Because we haven't configured the WiFi credentials there won't be any network access.

Assuming rigctld is available on `localhost` port `51111` and the serial interface of the controller is available at `/dev/ttyUSB0` you would run:

`./amp_serial_control.sh localhost 51111 /dev/ttyUSB0`

In a different terminal you can watch the serial output by running:
  
`./amp_serial_messages.sh /dev/ttyUSB0`
  
Now all frequency, mode and PTT state changes will be passed to the controller and it will work just as it does when connecting directly to rigctl over the network. 
  
To reduce PTT latency you can set `hybrid` to `true` in the config. This will use the analog control cable instead of rigctl for switching rx/tx.

## Bluetooth Serial Mode

You can also use the HC-05 for sending serial commands and reading the output. We achieve this by redirecting all serial operations from the built in serial port to the HC-05. This is particularly useful if the controller is not plugged into the computer you want to use serial with (but in range of Bluetooth).

Note: You can't use this with an Icom 705 (unless via rigctl) as the radio uses the HC-05 for communication.
  
The way this works is the HC-05 appears as a serial port once paired over Bluetooth. In Linux this is achieved using `rfcomm` which creates a pseudo serial device. This is transparent to the software accessing the device so you can do anything you usually can with serial devices. To use this feature with Linux:

1. Set `use_bluetooth_serial` to `true` and `hc_05_enabled` to `true` in the config
2. Run `hcitool scan` - this should find the controller as something like `98:D3:65:F1:3A:C0	XPA125B`
3. Run `sudo bluetoothctl` - the CLI will start
4. Run `trust 98:D3:65:F1:3A:C0` - use the MAC found with the previous scan
5. Run `pair 98:D3:65:F1:3A:C0` - enter the PIN you chose when configuring the HC-05
6. Run `set-alias XPA125B`
7. Run `connect XPA125B`
8. Run `quit` to close bluetoothctl
9. Run `sudo rfcomm bind rfcomm0 98:D3:65:F1:3A:C0` - use the MAC of your own
  
Now you will have a new serial device at `/dev/rfcomm0` ready for use. Following on from the example above you could use it with the script, like this:
  
`./amp_serial_control.sh localhost 51111 /dev/rfcomm0`

`./amp_serial_messages.sh /dev/rfcomm0`

Windows supports Bluetooth serial devices too (not sure about Mac) and there are even mobile phone apps available, such as `Serial Bluetooth Terminal` by Kai Morich for Android. If using that app I've found setting 'newline' to 'CR+LF' for both Send and Recieve works well.
  
Hopefully this demonstrates how flexible the controller can be for scenarios where WiFi connectivity is unavailable or undesirable. You can still enable WiFi with serial mode if you want the control to remain via serial but still have access to the other APIs and optionally also publish events to MQTT.
  
Note: The example scripts are very bare bones and intended to demonstrate the feature. One improvement would be to not open a new TCP connection to rigctld every time but instead keep it open and send commands over the same session.
  
# TX Block
  
If TX time exceeds 300 seconds (default) then TX will be blocked for 60 seconds (default). After the block releases you must send another TX event to start again - this includes Yaesu & Icom modes (i.e. release PTT).

In every mode this tells the amp to switch to RX. In rigctl mode this also tells the radio itself to stop TX'ing.
  
To configure the timings, just set the following in the config (in seconds):
 
+ `tx_limit` (maximum allowed TX time)
+ `tx_block_time` (how long to block TX for)

# Home Assistant / Node-RED
  
If you use Home Assistant (https://www.home-assistant.io/) you can add sensors via MQTT as follows:
  
```
#########################################################################
# XPA125B
#########################################################################
- platform: mqtt
  name: "XPA125B State"
  state_topic: "xpa125b/state"
- platform: mqtt
  name: "XPA125B Band"
  state_topic: "xpa125b/band"
- platform: mqtt
  name: "XPA125B Frequency"
  state_topic: "xpa125b/frequency"
  unit_of_measurement: "Hz"
- platform: template
  sensors:
      xpa125b_frequency_mhz:
        friendly_name: "XPA125B Frequency MHz"
        unit_of_measurement: 'MHz'
        icon_template: mdi:sine-wave
        value_template: >-
                {{ (states('sensor.xpa125b_frequency') | float  / 1000000 | round(6)) }}
- platform: mqtt
  name: "XPA125B Mode"
  state_topic: "xpa125b/mode"
- platform: mqtt
  name: "XPA125B Rig Mode"
  state_topic: "xpa125b/rigmode"
- platform: mqtt
  name: "XPA125B TX Time"
  state_topic: "xpa125b/txtime"
  unit_of_measurement: "s"
- platform: mqtt
  name: "XPA125B TX Block Timer"
  state_topic: "xpa125b/txblocktimer"
  unit_of_measurement: "s"
```

  Then create some cards on a dashboard and it will look something like this:
  
  ![ha](https://raw.githubusercontent.com/madpsy/xpa125b_controller/main/ha.png "ha")
  
Node-RED has built in support for MQTT. You can subscribe to the relevant topics and easily create automations based on these. Of course you can also send commands assuming you are in MQTT mode. 

This is particularly useful if you want to trigger other events based on the current state. An example might be to illuminate a light when you are transmitting or even send a notification if the TX Block Timer is activated.
  
In my case if TX Block is activated (i.e. `txblocktimer` is > 0) then the power to the amplifier is removed via a 'smart' plug. The basis being if TX exceeds the threshold then something must be amiss with the system so best to cut the power and find out what went wrong.
  
# To Do
  
+ Reduce the excessive use of `Strings` in the code
+ There is virtually no input validation. Therefore all input values are trusted. This can be a pro or a con.
+ Provide schematics for the interface board to the XPA125B amplifier
+ Add instructions how to adapt for other amplifiers

# Thanks

Thank you to Mike from Hamlib for promptly fixing a connection delay issue with Windows (https://github.com/Hamlib/Hamlib/issues/873)

Quick shout out to https://www.hobbypcb.com/ for some of the CI-V logic.
