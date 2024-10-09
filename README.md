---
# victron_ble

A Python library to parse Instant Readout advertisement data from Victron devices.

Disclaimer: This software is not an officially supported interface by Victron and is provided entirely "as-is"

**Supported Devices:**

* SmartShunt 500A/500mv and BMV-712/702 provide the following data:
    * Voltage
    * Alarm status
    * Current
    * Remaining time
    * State of charge (%)
    * Consumed amp hours
    * Auxilary input (temperature, midpoint voltage, or starter battery voltage)
* Smart Battery Sense
    * Voltage
    * Temperature (°C)
* Solar Charger (Tested with BlueSolar 75/15):
    * Charger State (Off, Bulk, Absorption, Float)
    * Battery Voltage (V)
    * Battery Charging Current (A)
    * Solar Power (W)
    * Yield Today (Wh)
    * External Device Load (A)

If you'd like to support development for additional devices, consider [sponsoring this project](https://github.com/sponsors/keshavdv/)


## Install it from PyPI

```bash
pip install victron_ble
```

## Usage

#### Fetching Keys

To be able to decrypt the contents of the advertisement, you'll need to first fetch the per-device encryption key from the official Victron application:
 
1. Install the VictronConnect app ([Android](https://play.google.com/store/apps/details?id=com.victronenergy.victronconnect), [IOS](https://apps.apple.com/us/app/victron-connect/id943840744), [Linux](https://www.victronenergy.com/support-and-downloads/software#victronconnect-app), [OSX](https://apps.apple.com/us/app/victronconnect/id1084677271?ls=1&mt=12), [Windows](https://www.victronenergy.com/support-and-downloads/software#victronconnect-app))
2. Open the app and pair with your device
3. Navigate to Settings, Menu, Product Info
4. Enable Instant readout via Bluetooth to be able to receive advertisements from your device
5. Copy MAC Address & Encryption Key by clicking on the Show button
6. Turn the MAC Address to the right format: fd2afb297f8f becomes FD:2A:FB:29:7F:8F 

**Headless system**
You can follow the above instruction to get the keys but you will need to pair with your headless system (using `bluetoothctl` for ex) to continue the proccess.

**OSX**

[MacOS's bleak backend](https://bleak.readthedocs.io/en/latest/backends/macos.html) uses a bluetooth UUID address instead of the more traditional MAC address to identify bluetooth devices. This UUID address is often unique to the device scanned *and* the device being scanned such that it cannot be used to connect to the same device from another computer. 

If you are going to use `victron-ble` on the same Mac computer as you have the Victron app on, follow the instructions below to retrieve the address UUID and advertisement key:

1. Install the VictronConnect app from the [Mac App Store](https://apps.apple.com/us/app/victronconnect/id1084677271?ls=1&mt=12)
2. Open the app and pair with your device
3. Enable Instand readout via Bluetooth to be able to receive advertisements from your device
4. Run the following from Terminal to dump the known keys (install `sqlite3` via Homebrew)
```bash
❯ sqlite3 ~/Library/Containers/com.victronenergy.victronconnect.mac/Data/Library/Application\ Support/Victron\ Energy/Victron\ Connect/d25b6546b47ebb21a04ff86a2c4fbb76.sqlite 'select address,advertisementKey from advertisementKeys inner join macAddresses on advertisementKeys.macAddress == macAddresses.macAddress'

{763aeff5-1334-e64a-ab30-a0f478s20fe1}|0df4d0395b7d1a876c0c33ecb9e70dcd
❯
```

#### Reading data

Here we'll take OSX system as example. If you're using an other system, replace UUID by Mac address representation.
The project ships with a standalone CLI that can be used to print device data to the console. 

```bash
# Will show all discovered Victron devices with Instant Readout enabled, their names, and IDs
$ > victron-ble discover 
763aeff5-1334-e64a-ab30-a0f478s20fe1: SmartShunt HT4531A246S
...


# Dump data for a particular device (replace the ID and key with your own)
$ > victron-ble read "763aeff5-1334-e64a-ab30-a0f478s20fe1@0df4d0395b7d1a876c0c33ecb9e70dcd"
INFO:victron_ble.scanner:Reading data for ['763aeff5-1334-e64a-ab30-a0f478s20fe1']
{
  "name": "SmartShunt HT4531A246S",
  "address": "763AEFF5-1334-E64A-AB30-A0F478S20FE1",
  "rssi": -79,
  "payload": {
    "aux_mode": "temperature",
    "consumed_ah": 0.0,
    "current": 0.0,
    "high_starter_battery_voltage_alarm": false,
    "high_temperature_alarm": false,
    "high_voltage_alarm": false,
    "low_soc_alarm": false,
    "low_starter_battery_voltage_alarm": false,
    "low_temperature_alarm": false,
    "low_voltage_alarm": false,
    "midpoint_deviation_alarm": false,
    "remaining_mins": 65535,
    "soc": 100.0,
    "temperature": 382.2,
    "voltage": 12.87
  }
}
...

# Dump data for debugging and supporting new devices (replace the ID)
$ > victron-ble dump "763aeff5-1334-e64a-ab30-a0f478s20fe1"
Dumping advertisements from 763aeff5-1334-e64a-ab30-a0f478s20fe1
1671843194.0534039      : 100289a302413bafd03bb245e131ae926267f6fd0b59e0
1671843194.682535       : 100289a302423baf58a1546e5262dcdf0ef642f353ed65
1671843197.676384       : 100289a302453baf804707549cffb2ab970c981ae897b6
...
```

To consume this project as a library, you can import the particular parser for your device:
```py
from victron_ble.devices import detect_device_type

data = <ble advertisement data>
parser = detect_device_type(data)
parsed_data = parser(<key>).parse(<ble advertisement data>)
```

## Development

If you'd like to help support a new device, collect the following and create a new Github issue:

1. Run `victron-ble discover` to find the ID of the device you'd like to support
2. Run `victron-ble dump <ID>` for a few minutes while collecting corresponding screenshots from the official apps instant readout to identify the current values

For pull requests:

Read the [CONTRIBUTING.md](CONTRIBUTING.md) file.

## Contributors

Special thanks to https://github.com/rochacbruno/python-project-template for the project template
