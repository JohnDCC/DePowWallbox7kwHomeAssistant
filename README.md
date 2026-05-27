You'll need:
- Local Tuya here: https://github.com/make-all/tuya-local
- Probably a Tuya Development account to get your device ID etc (https://developer.tuya.com/en/)
  - Use API explorer to query device status once connected
  - For ease of using the below, **call your charger EVCharger2 as I have**, otherwise be aware you'll need to update this in all scripts
- Add your device to local tuya once integration installed
- 

Files attached here are:

**Templates YAML**
  Makes the sensor readout a bit nicer for charger status.

**Solar EV adjust**
  The brains behind dynamic charging rate. You'll need to adjust the times in the file to match your power plan or desires. My suggestion is to put the file into an LLM and ask it to help you set it up to your needs if you aren't comfortable making the changes.
  
  In this file you'll need to adjust a lot of settings:
  _max_grid_allowance: 14.1_ 
  Your Maximum grid import limit. Varies from One Phase to 3 Phase
  
  _amps_per_kw_ratio: 0.5_ 
  How many additional amps you want to supply per kw of solar generated during the non free period. If you find the charging is too greedy then ratio is too high. Use this to balance your non free period usage.
  
  _solar_allowance: 0_
  If you want to create some additional headroom, no extra amps added to charging unless solar is above the allowance, kw.
  
  _phase_2_compensation_amps: 4_
  The baseline amps you use during the free power period. Depends on your household load and whether you can allow extra. If default_amps is set to 8, it would be 12 in the free period.
  
  _battery_charge_kw: 9.8_
  How many kw you need to reserve to charge your house battery at it's maximum rate.
  
  In the below you will need to a**djust the sensors in bold according to your inputs / integrations**. Not my ev charger added from localtuya is called EVCharger2 so yours will need to match what you have named it during local tuya setup.
  
  solar_kw: "{{ states('**sensor.smoothed_solar_kw**') | float(0) - solar_allowance }}"
  battery_soc: "{{ states('**sensor.foxess_bat_soc**') | int(0) }}"
  current_time: "{{ now().strftime('%H:%M') }}"
  is_daylight: "{{ states('sun.sun') == 'above_horizon' }}"
  manual_boost_active: "{{ is_state('input_boolean.ev_override_boost', 'on') }}"
  operating_state: "{{ states('number.**evcharger2_operating_state**') | int(0) }}"
  live_amps: "{{ states('number.**evcharger2_charge_current**') | float(0) }}"

**EV Manual Boost Controller**
  This allows you to temporarily override the solar automation with a timed charging period.
  It requires the creation of a helper.
  
  ev_boost_duration. Number, min; 0.5, step size; 0.25, max; 12 (or more as needed), unit of measurement; h
  
  The boost will turn off automatically at time end. Time can be adjusted mid boost to extend the end time. You can also setup a calendar to turn boost on and off at specific times.

**EV UI Slider True Two-Way Sync Bridge**
  Improves the slider function as the local tuya integration let's it go to 0 and the minimum the charger supports is 6, so this just cleans up the interface. It reads from the charger and writes to the charger when adjusted.

**EV Scheduled Boost Sync** (Optional)
  Let's you run boost off a calendar called EV Boost Schedule. Has an in built battery 40% SOC cutoff which you will need to adjust to your preferences.

The basic premise of how this operates:
Controlling the charger using On/Off Commands is clunky. Instead it's better to use the scheduling feature to turn schedule on and off or adjust schedule time in order that the charger will then turn on and off automatically if it is inside or outside the schedule.

Thus, I have built some functions to control amp output based on household limitations, houshold power use and solar generation. I use Fox ESS battery so you will need to replace with your relevant battery data.
