# HomeAssistant HACS custom RPI GPIOD integration

This HACS integration is used to address GPIO (especially and only tested for RaspberryPi) using libgpiod and python gpiod >=v2.02 since RPI.gpio is no longer supported. I created this for my own use, and to learn how to build a custom integration, but since it is working on Raspberry Pi feel free to use.

:bulb: The code of this custom component upto v1.4.5 is copied over to the original HACS [RPI_GPIO](https://github.com/thecode/ha-rpi_gpio) gpio integration. You should most likely check and use that integration instead of this one. I'm keeping this repository for my own fun and use.

:warning: This is working in my homeassistant environment. Shared for testing. Use at your own risk.

`ha_gpiod` is based on [ha-rpi_gpio](https://github.com/thecode/ha-rpi_gpio), which was already adapted for `gpiod` in [ha-gpio](https://codeberg.org/raboof/ha-gpio) and rewritten from scratch for `gpiod` by me. Just recently this code is copied back to [ha-rpi_gpio](https://github.com/thecode/ha-rpi_gpio)

# Installation
Via HACS: add as custom repository from github using this github https://github.com/jdeneef/ha_gpiod/ (HACS => integrations => 3-dots menu => custom repositories, enter repository URL, select "Integrations" in Category drop-down, go back to hacs, install gpiod component, reboot, add config, reboot).  
If you run into errors, ensure HACS is up-to-date.

# Usage
The `gpiod` platform will be initialized using the path to the gpio chip. When path is not in the config `/dev/gpiochip[0-5]` are tested for a gpiodevice having `pinctrl`, in sequence `[0,4,1,2,3,5]`. So with a raspberry pi you should be OK to leave the path empty.

Raspberry Pi | GPIO Device
---          | ---
RPi3, RPi4   | `/dev/gpiochip0`
RPi5         | `/dev/gpiochip4`


## Sample config for the raspberry pi berryclip hat

See also configuraton-example.yaml; note that below config shows most options, and can't be used as is, since `cover` reuses `switch` and `sensor` ports.
```yaml
# setup gpiod chip; mostly not required
gpiod:
  path: '/dev/gpiochip0'

# Example of switches (eg switched leds or buzzer) with some sample parameters
switch:
  - platform: gpiod
    switches:
      - name: "Led 1 red"
        port: 4
      - name: "Led 2 red"
        port: 17
        unique_id: "gpio_led_red_2"
      - name: "Led 3 yellow"
        port: 22
        active_low: true
      - name: "Led 6 green"
        port: 11
        bias: "AS_IS"
        drive: "PUSH_PULL"
      - name: "Fan"
        port: 8
        persistent: true

# Example of binary_sensor (eg push button) setup
binary_sensor:
  - platform: gpiod
    sensors:
    - name: "Button"
      port: 7

# Example of cover setup
cover:
  - platform: gpiod
    covers:
      - relay_port: 4
        name: Cover blue
        state_port: 7
        relay_time: 1200
 ```

## Platform

Key | Required | Default | Type | Description
--- | --- | --- | --- | ---
`gpiod` | only for path|- |- | `gpiod` platform config and initialization, only required when you need to specify a specific gpiodevice path (see path)
`path` | no | discovered | string | path to gpio device, if not set auto discovered

## Binary Sensor

The `gpiod` binary sensor platform allows you to read sensor values of the GPIOs of your device. To ensure consistence over reboots the sensor status is read at startup and set accordingly.

### Configuration options

Key | Required | Default | Type | Description
--- | --- | --- | --- | ---
`sensors` | yes | | list | List of sensor IO ports ([Raspberry Pi BCM mode pin numbers](https://pinout.xyz/resources/raspberry-pi-pinout.png))
`name` | yes | | string  | The name for the binary sensor entity
`port` | yes | | integer | the GPIO port to be used
`unique_id` | no | generated | string  | An ID that uniquely identifies the sensor. Set this to a unique value to allow customization through the UI, auto generated when not set manually in config
`debounce` | no | `50` | integer | The time in milliseconds for port debouncing
`active_low` or `invert_logic`[^1] | no | `false` | boolean | If `true`, input of `gpio` is inverted, `active_low` results in `on`; `invert_logic` kept for backwards compatibility
`bias` or `pull_mode` | no | `PULL_UP` | string  | control bias setting of GPIO, used to define the electrical state of a GPIO line when not actively driven; `PULL_UP` set weak pull-up resistor on the line, ensuring that the line is pulled to a high level (3.3V or 5V) when not actively driven; `PULL_DOWN` sets weak pull-down resistor to pull to low level (0V), `DISABLED` remains floating, `AS_IS` not changed; `pull_mode` kept for backwards compatibility

[^1]: In `ha_gpiod` the `gpiod` terminology is used as default, but the original `ha-rpi_gpio` config parameters are maintained for backwards compatibility. Using the `gpiod` terminology helps in understanding what the `libgpiod` is really doing.

## Switch

The `gpiod` switch platform allows you to control the GPIOs of your device. To ensure consistence over restarts the `persistent` flag should be set.


### Options

Key | Required | Default | Type | Description
--- | --- | --- | --- | ---
`switches` | yes | | list | List of switch IO ports ([Raspberry Pi BCM mode pin numbers](https://pinout.xyz/resources/raspberry-pi-pinout.png))
`name` | yes | | string | The name for the switch entity
`port` | yes | | integer | the GPIO port to be used
`unique_id` | no | generated | string | An ID that uniquely identifies the switch. Set this to a unique value to allow customization through the UI, auto generated when not set manually in config
`active_low` or `invert_logic`| no | `false` | boolean | If `true`, output of `gpio` is inverted, `active_low` switches `on`; `invert_logic` kept for backwards compatibility
`bias` or `pull_mode` | no | `AS_IS` | string  | Type of internal pull resistor to use: `PULL_UP` - pull-up resistor, `PULL_DOWN` - pull-down resistor, `AS-IS` no change; `pull_mode` kept for backwards compatibility
`drive`|no| `PUSH_PULL`|string | control drive configuration of the GPIO, determines how the line behaves when it is set to output mode; `PUSH_PULL`, GPIO line can both source and sink current, can actively drive the line to both high and low states. `OPEN-DRAIN`, GPPIO can only sink current (drive the line to low) and is otherwise left floating, and `OPEN-SOURCE` the reverse.
`persistent` | no | `false` | boolean | If true, the switch state will be persistent in HA and will be restored if HA restart / crash.

## Cover

The `gpiod` cover platform allows you to control GPIOs to open/close covers; note that I have only verified cover functionality simulating with switches and buttons, so logic could be off on some points .. 
Covers consist of a switch for triggering cover motor, and a state sensor for determining the state of the cover. This state is as for all sensors read at startup time, so should be consistent over reboots.

### Options

Key | Required | Default | Type | Description
--- | --- | --- | --- | ---
`name` | yes | | string | The name for the cover entity
`relay_port` or `relay_pin`|yes| |integer|Relay switch gpio switching cover motor; `relay_pin` kept for backwards compatibility
`relay_time`|no|`200` |integer|Time in milliseconds relay switch will be switched to open/close cover
`relay_active_low` or `invert_relay`|no | `false`| boolean| invert input for `relay_port`; `invert_relay` kept for backwards compatibility
`relay_bias` | no | `AS_IS` | string  | Type of internal pull resistor to use: `PULL_UP` - pull-up resistor, `PULL_DOWN` - pull-down resistor
`relay_drive`|no|`PUSH_PULL`|string|set `relay_pin` `drive_mode`, options: `OPEN_DRAIN`, `OPEN_SOURCE`, `PUSH_PULL`
`state_port` or `state_pin`|yes| | integer|State port for opened/closed status of cover; `state_pin` kept for backwards compatibility
`state_bias` or `state_pull_mode`| no | `PULL_UP` | string  | Type of internal pull resistor to use: `PULL_UP` - pull-up resistor, `PULL_DOWN` - pull-down resistor; `state_pull_mode` kept for backwards compatibility
`state_active_low` or `invert_state`|no | `false`| boolean| invert output for state pin; `invert_state` kept for backwards compatibility
`state_debounce`|no | 50 | integer | debounce parameter for the cover state sensor
`unique_id` | no | generated | string | An ID that uniquely identifies the switch. Set this to a unique value to allow customization through the UI, auto generated when not set manually in config


# Add Debug info and issue reporting
*Before* reporting issues please add this to your `configuration.yaml` `logger` section, check logs and report issue adding logging.
```yaml
logger:
  default: info
  logs:
    custom_components.gpiod: debug
```
