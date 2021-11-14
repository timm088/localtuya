Holman WX1 appears similar to Tuya Zigbee/Wifi gateway, except it is not Zigbee between tap and gateway just some UHF proprietary radio carrier. 
As such it can't be used with any Zigbee coordinator

To have tap integration working a few steps were required:
- I merged proposed gateway shortcut adding cid in "status" command
- I expanded it from "status" only to work on "set" command too.
- I also merged PR adding "number" platform as tap has timer which requires writable number

I tested it as far, as I could, and so far:
- It does work on tap
- cid shortcut works with multiple devices on the same IP - tested with socket and tap
- So far it did not appear as breaking change for devices without gateway. While cid is added in commands payload, my Arlec fan hapilly ignores it and executes command.

DPS map:
{ dps:
   { '101': 0,    // temperature
     '102': 0,    // moisture
     '103': 4,    // amount of water used last time
     '105': '2',
     '106': '1',   // 1 when runs water, 0 when stopped, 3 when stoped and time/rain delay active
     '107': 4,     // minutes set for manual run, up to 60 (starts with 108)
     '108': false, // starts manual countdown when false
     '113': '48',  // delay to skip
     '114': '24',  // don't know
     '115': false, // presence of soil sensor
     '116': false, // presence of rain sensor? (guess)
     '117': true,  // presence of TAP? (guess)
     '119': '1',   // 0 = litres/C, 0 < Gal/F
     '120': 0,     // amount of rain? (guess)
     '125': false, // pause due to high soil moisture
     '127': 'HOL9-018-023-000-000' },
  cid: '131' }

CID can be found by quering Tuya API (I use tuya-cli), mine is 131 for tap, 13B for socket. I believe all may have the same, please let me know what you found.
Timer countdown counts down as it goes but gains back initial position after stop.

Sample yaml config (I don't like clicking):

localtuya:
  - host: 192.0.2.1
    device_id: '13B'
    local_key: key from API
    friendly_name: Holman Socket
    protocol_version: "3.3"
    entities:
      - platform: switch
        friendly_name: "Holman Hub Socket"
        id: 1
  - host: 192.0.2.1
    device_id: 131
    local_key: key from API
    friendly_name: Holman Irrigation Tap
    protocol_version: "3.3"
    entities:
      - platform: sensor
        friendly_name: "Soil Temperature"
        id: 101
        unit_of_measurement: "°C"
        device_class: temperature
      - platform: sensor
        friendly_name: "Soil Moisture"
        id: 102
        unit_of_measurement: "%"
        device_class: humidity
      - platform: sensor
        friendly_name: Water spent
        unit_of_measurement: "litres"
        id: 103
      - platform: sensor
        friendly_name: "Irrigation status"  # 0 - stopped, 1 - runs, 3 - delayed
        id: 106
      - platform: number
        friendly_name: Countdown minutes
        min_value: 0
        max_value: 60   # Probably can go further
        id: 107
      - platform: switch
        friendly_name: "Irrigation Tap (negated)"
        id: 108
      - platform: binary_sensor
        friendly_name: "Postponed due to moisture"
        id: 125

