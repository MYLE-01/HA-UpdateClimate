# HA-UpdateClimate

Python script to update climate devices.

## Installation

Download the hacs.py file from inside the python_scripts directory here to your local python_scripts directory, then reload python_scripts in Home Assistant.

## Usage

This python script sets `hvac_mode` and `preset_mode` according to the specified sensors and time.

![BPMN](/img/bpmn.gif)

The `hvac_mode` will be _off_ when:

- At least one window is open
- At least one of the given `sensors_off` is _off_

In all other cases the `hvac_mode` will be the one specified in `active_mode`.
The `preset_mode` is None if the `sensor_presence` is _on_ or not given and the current time is between `heating_from_hour` and `heating_to_hour`.
Otherwise the `preset_mode` is the one specified under `away_mode`.
If one of the specifications `heating_from_hour` or `heating_to_hour` is not given, the `preset_mode` depends only on the `sensor_presence`.

## Service description

Paste the following into `python_scripts/services.yaml` to enable the service description within HomeAssistant.

```yaml
update_climate:
  description: Change the hvac mode of the given climate device.
  fields:
    enitity_id:
      description: The climates enitity_id
      example: climate.livingroom
    windows:
      description: The climate will be off when one of these sensors is on.
      example: [binary_sensor.livingroom_window, binary_sensor.livingroom_door]
    sensors_off:
      description: The climate will be off when one of these sensors is off.
      example: [binary_sensor.climate_on, binary_sensor.livingroom_climate_on]
    sensor_presence:
      description: The climate will switch to active mode if this sensor is on.
      example: binary_sensor.someone_at_home
    heating_from_hour:
      description: Start time from which heating is to start
      example: 8
    heating_to_hour:
      description: End time to which the heating is to last.
      example: 17
    active_mode:
      description: The hvac_mode when active (defaults to heat)
      example: heat
    away_preset:
      description: The preset_mode when away/eco (defaults to Heat Eco)
      example: away
```

## Automation example
This works with Eurotronic Spirit Z-Wave thermostats.
To use different devices, you may want to change `active_mode` and `away_preset`.

```yaml
- id: 0123456789
  alias: Climate Livingroom
  trigger:
    - hours: "*"
      minutes: "1"
      platform: time_pattern
    - entity_id: binary_sensor.presence
      platform: state
    - entity_id: binary_sensor.livingroom_window
      platform: state
    - entity_id: input_boolean.livingroom_climate
      platform: state
    - entity_id: binary_sensor.all_climates_on
      platform: state
  condition: []
  action:
    - data:
        entity_id: climate.livingroom
        heating_from_hour: 8
        heating_to_hour: 23
        sensor_presence: binary_sensor.presence
        sensors_off:
          - input_boolean.livingroom_climate
          - binary_sensor.all_climates_on
        windows:
          - binary_sensor.livingroom_window
      service: python_script.update_climate
```

As one can see, the livingroom will be heated between 8am and 11pm if `binary_sensor.presence` is on.
As soon as `binary_sensor.livingroom_window` turns on or one of `input_boolean.livingroom_climate` or `binary_sensor.all_climates_on` turns off, the thermostat will be off as well.
