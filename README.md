# HA-UpdateClimate

[![hacs](https://img.shields.io/badge/HACS-Default-orange.svg?style=flat)](https://github.com/custom-components/hacs)
[![license](https://img.shields.io/github/license/Santobert/HA-UpdateClimate.svg?style=flat)](https://github.com/Santobert/HA-UpdateClimate/blob/master/LICENSE)
[![release](https://img.shields.io/github/v/release/Santobert/HA-UpdateClimate.svg?style=flat)](https://github.com/Santobert/HA-UpdateClimate/releases)
[![last-commit](https://img.shields.io/github/last-commit/Santobert/HA-UpdateClimate.svg?style=flat)](https://github.com/Santobert/HA-UpdateClimate/commits/master)
[![language](https://img.shields.io/github/languages/top/Santobert/HA-UpdateClimate.svg?style=flat)](https://github.com/Santobert/HA-UpdateClimate)
[![code-size](https://img.shields.io/github/languages/code-size/Santobert/HA-UpdateClimate.svg?style=flat)](https://github.com/Santobert/HA-UpdateClimate)

Python script to update climate devices.

## Installation

Install via HACS (recommended) or download the `update_climate.py` file from inside the python_scripts directory here to your local python_scripts directory, then reload python_scripts in Home Assistant.

## Usage

This python script sets `hvac_mode` and `preset_mode` according to the specified sensors and time.

![BPMN](bpmn.png)

The `hvac_mode` will be _off_ when:

- At least one window is open
- At least one of the given `sensors_off` is _on_
- At least one of the given `sensors_on` is _off_

In all other cases the `hvac_mode` will be the one specified in `hvac_active`.
The `preset_mode` is None if the `sensor_presence` is _on_ or not given and the current time is between `heating_from_hour` and `heating_to_hour`.
Otherwise the `preset_mode` is the one specified under `away_mode`.
If one of the specifications `heating_from_hour` or `heating_to_hour` is not given, the `preset_mode` depends only on the `sensor_presence`.

| Name              | Required | Description                                                 |
| ----------------- | -------- | ----------------------------------------------------------- |
| entity_id         | True     | The climates enitity_id                                     |
| sensors_on        | False    | The climate will be off when one of these sensors is off    |
| sensors_off       | False    | The climate will be off when one of these sensors is on     |
| sensor_presence   | False    | The climate will switch to active mode if this sensor is on |
| heating_from_hour | False    | Start time from which heating is to start                   |
| heating_to_hour   | False    | End time to which the heating is to last                    |
| hvac_active       | False    | The hvac_mode when active *(defaults to heat)*              |
| preset_away       | False    | The preset_mode when away/eco *(defaults to Heat Eco)*      |


me was not getting the window logic
so
I have change it 

sensors_on and sensors_off

meaning:

all sensors_on must be ON
 
all sensors_off must be OFF
 
if one not in the right state then logic is wrong (true)

22 may:  got the seson logic right (leaning Python here)

```python
for season in SENSOR_SEASON:
    if season not in SENSOR_SEASON:
        bool_off = True
```


## Service Example


The following is the content of a [service call](https://www.home-assistant.io/docs/scripts/service-calls/).
This example includes all possible parameters.
You may not need them all.


```yaml
service: python_script.update_climate
data:
  enitity_id: climate.livingroom
  sensors_on:
    - binary_sensor.livingroom_window
    - binary_sensor.livingroom_door
  sensors_off:
    - binary_sensor.climate_on
    - binary_sensor.livingroom_climate_on
  sensor_presence: binary_sensor.someone_at_home
  season:
    - winter
    - autumn
  heating_from_hour: 8
  heating_to_hour: 17
  hvac_active: heat
  preset_away: away
```

season: lets you add a season you only want it to work in

## Automation example

The following is the content of an [automation](https://www.home-assistant.io/docs/automation/).
This works with Eurotronic Spirit Z-Wave thermostats.
To use different devices, you may want to change `hvac_active` and `preset_away`.

```yaml
- id: 0123456789
  alias: Climate Livingroom
  trigger:
    - hours: "*"
      minutes: "1"
      platform: time_pattern
    - entity_id: 
        - binary_sensor.presence
        - binary_sensor.livingroom_window
        - input_boolean.livingroom_climate
        - binary_sensor.all_climates_on
      platform: state
  condition: []
  action:
    - data:
        entity_id: climate.livingroom
        sensors_on:
          - binary_sensor.livingroom_window
        sensors_off:
          - input_boolean.livingroom_climate
          - binary_sensor.all_climates_on
        sensor_presence: binary_sensor.presence
        season:
         - winter
         - autumn
        heating_from_hour: 8
        heating_to_hour: 23
      service: python_script.update_climate
```
## MUST HAVE

you must that the season sensor

```yaml
sensor:
  - platform: season
```
