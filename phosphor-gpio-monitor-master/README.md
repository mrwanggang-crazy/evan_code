# GPIO Monitoring

## Implemented Daemons

### `phosphor-gpio-monitor`

This daemon accepts a command line parameter for monitoring single gpio line and
take action if requested. This implementation uses GPIO keys and only supports
monitoring single GPIO line, for multiple lines, user has to run this daemon
seperately for each gpio line.

### `phosphor-multi-gpio-monitor`

This daemon accepts command line parameter as a well-defined GPIO configuration
file in json format to monitor list of gpios from config file and take action
defined in config based on gpio state change. It uses libgpiod library.

### Difference

New implementation (phosphor-multi-gpio-monitor) provides multiple gpio line
monitoring in single instance of phosphor-multi-gpio-monitor running. It is very
easy to add list of gpios into JSON config file and it also supports of GPIO
line by name defined in kernel.

## Configuration

There is a phosphor-multi-gpio-monitor.json file that defines details of GPIOs
which is required to be monitored. This file can be replaced with a platform
specific configuration file via bbappend.

Following are fields in json file

1. Name: Name of gpio for reference.
2. LineName: this is the line name defined in device tree for specific gpio
3. GpioNum: GPIO offset, this field is optional if LineName is defined.
4. ChipId: This is device name either offset ("0") or complete gpio device
   ("gpiochip0"). This field is not required if LineName is defined.
5. EventMon: Event of gpio to be monitored. This can be "FALLING", "RISING" OR
   "BOTH". Default value for this is "BOTH".
6. Target: This is an optional systemd service which will get started after
   triggering event. A journal entry will be added for every event occurs
   irrespective of this definition.
7. Targets: This is an optional systemd service which will get started after
   triggering corresponding event(RASING or FALLING). A journal entry will be
   added for every event occurs irrespective of this definition.
8. Continue: This is a optional flag and if it is defined as true then this gpio
   will be monitored continously. If not defined then monitoring of this gpio
   will stop after first event.

## Sample config file

```json
[
  {
    "Name": "PowerButton",
    "LineName": "POWER_BUTTON",
    "GpioNum": 34,
    "ChipId": "gpiochip0",
    "EventMon": "FALLING",
    "Target": "PowerButtonDown.service",
    "Continue": true
  },
  {
    "Name": "PowerGood",
    "LineName": "PS_PWROK",
    "EventMon": "BOTH",
    "Targets": {
      "FALLING": ["PowerGoodFalling.service", "PowerOff.service"],
      "RISING": ["PowerGoodRising.service", "PowerOn.service"]
    },
    "Continue": false
  },
  { "Name": "SystemReset", "GpioNum": 46, "ChipId": "0" }
]
```

### `phosphor-multi-gpio-presence`

This daemon accepts command line parameter as a well-defined GPIO configuration
file in json format to monitor list of gpios from config file and sets inventory
presence as defined in config based on gpio state change. It uses libgpiod
library.

### Difference

New implementation (phosphor-multi-gpio-presence) provides multiple gpio line
monitoring in single instance of phosphor-multi-gpio-presence running. It is
very easy to add list of gpios into JSON config file and it also supports of
GPIO line by name defined in kernel.

## Configuration

There is a phosphor-multi-gpio-presence.json file that defines details of GPIOs
which is required to be monitored. This file can be replaced with a platform
specific configuration file via bbappend.

Following are fields in json file

1. Name: PrettyName of inventory item
2. LineName: this is the line name defined in device tree for specific gpio
3. GpioNum: GPIO offset, this field is optional if LineName is defined.
4. ChipId: This is device name either offset ("0") or complete gpio device
   ("gpiochip0"). This field is not required if LineName is defined.
5. Inventory: Object path under inventory that will be created
6. ExtraInterfaces: [Optional] List of interfaces to associate to inventory item
7. ActiveLow: [Optional] Object is present on LOW level
8. Bias: [Optional] Configure a BIAS on the GPIO line, for example PULL_UP

## Sample config file

```json
[
  {
    "Name": "DIMM A0",
    "LineName": "POWER_BUTTON",
    "Inventory": "/system/chassis/motherboard/dimm_a0"
  },
  {
    "Name": "Powersupply 0",
    "ChipId": "0",
    "GpioNum": 14,
    "Inventory": "/system/chassis/motherboard/powersupply0",
    "ActiveLow": true,
    "Bias": "PULL_UP",
    "ExtraInterfaces": ["xyz.openbmc_project.Inventory.Item.PowerSupply"]
  }
]
```
