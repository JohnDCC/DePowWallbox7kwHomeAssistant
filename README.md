# DePow Wallbox 7kW Home Assistant Package

Home Assistant templates, helpers, and automations for controlling a **DePow 7kW EV charger** through **LocalTuya**, with support for:

- cleaner charger status readouts,
- solar-aware dynamic charging,
- manual timed boost charging,
- a safer UI amperage slider,
- and optional calendar-based boost scheduling.

This setup is intended for users who already have their charger exposed in Home Assistant and want more flexible, smarter charging control than simple on/off automations.

---

## What this package does

Instead of controlling the charger purely with direct on/off commands, this setup works by adjusting the charger's scheduling/current behavior in a more reliable way.

The automations are designed to help you:

- charge harder during cheap or free power periods,
- reduce charging when household demand is high,
- increase charging when solar surplus is available,
- temporarily override solar logic with a manual boost,
- and optionally trigger charging from a calendar schedule.

---

## Features

- **Human-friendly charger state sensors**
- **Dynamic charging current adjustment**
- **Manual timed boost mode**
- **Two-way EV amperage slider sync**
- **Optional scheduled boost from a calendar**
- **Helper entities for dashboard control and diagnostics**

---

## Prerequisites

Before using these files, you should already have:

- **Home Assistant** installed and working
- **LocalTuya** installed  
  Recommended source: `https://github.com/make-all/tuya-local`
- Your **DePow charger added to LocalTuya**
- A **Tuya Developer account** if needed to obtain device details or inspect device status  
  `https://developer.tuya.com/en/`

### Recommended
- Use Tuya API Explorer to inspect device status once the charger is connected.
- Be comfortable editing YAML and replacing entity IDs with your own.

---

## Important naming assumption

The example configuration assumes your charger is named:

`EVCharger2`

If your charger entities use a different name, you will need to replace all references such as:

- `number.evcharger2_operating_state`
- `number.evcharger2_charge_current`

with your own entity IDs throughout the files.

---

## Repository contents

| File / Section | Purpose | Required |
| --- | --- | --- |
| **Templates YAML** | Makes charger state and power readouts easier to understand in Home Assistant. | Yes |
| **Solar EV Adjust** | Main automation logic for dynamic charging based on solar, battery, grid allowance, and time-of-day rules. | Yes |
| **EV Manual Boost Controller** | Temporarily overrides solar automation and enables timed charging. | Yes |
| **EV UI Slider True Two-Way Sync Bridge** | Keeps a Home Assistant slider synced with charger current while enforcing a safe minimum of 6A. | Recommended |
| **EV Scheduled Boost Sync** | Optional calendar-based boost charging automation. | Optional |

---

## Installation overview

A recommended setup order is:

1. Install and configure **LocalTuya**
2. Add the charger to Home Assistant
3. Confirm all required charger entities exist
4. Create the required helper entities
5. Add the template sensors
6. Add the main charging automations/scripts
7. Replace all example entity IDs with your own
8. Reload YAML / restart Home Assistant
9. Test manual boost first
10. Test solar-based adjustment
11. Optionally enable calendar-based boost

---

## Required Home Assistant entities

This setup uses a mixture of:

- **Helpers created in the Home Assistant UI**
- **Template sensors created in YAML**

---

## Helpers to create in Home Assistant

Create these in **Settings → Devices & Services → Helpers** where applicable.

### Helper summary

| Friendly Name | Entity ID | Type | Purpose |
| --- | --- | --- | --- |
| **Boost Time** | `input_number.ev_boost_duration` | Input Number | Sets the duration of a manual boost session |
| **EV Override Boost** | `input_boolean.ev_override_boost` | Input Boolean | Enables/disables manual override of solar charging logic |
| **EV UI Amperage Slider** | `input_number.ev_ui_amperage_slider` | Input Number | Dashboard slider for charger current, constrained to safe values |

### Suggested helper configuration

| Friendly Name | Entity ID | Helper Type | Min | Max | Step | Unit |
| --- | --- | --- | ---: | ---: | ---: | --- |
| **Boost Time** | `input_number.ev_boost_duration` | Number (Slider) | `0` | `5` | `0.5` | `h` |
| **EV Override Boost** | `input_boolean.ev_override_boost` | Toggle | — | — | — | — |
| **EV UI Amperage Slider** | `input_number.ev_ui_amperage_slider` | Number (Slider) | `6` | `32` | `1` | `A` |

> Adjust the max boost time if you want longer manual charge periods.

---

## Template sensors to define in YAML

These are not UI helpers. They should be created as template entities in YAML.

| Friendly Name | Entity ID | Type | Purpose |
| --- | --- | --- | --- |
| **Charging Automation State** | `sensor.charging_automation_state` | Template Sensor | Diagnostic/visibility sensor for automation state |
| **EV Charger Power** | `sensor.ev_charger_power` | Template Sensor | Tracks EV charger power draw in kW |
| **EV Charger State** | `sensor.ev_charger_state` | Template Sensor | Converts charger state codes into readable statuses |

---

## Required entity replacements

Before enabling the automations, replace all example entity IDs with your own.

### Common examples to update

| Example entity | Replace with |
| --- | --- |
| `sensor.smoothed_solar_kw` | Your solar production sensor |
| `sensor.foxess_bat_soc` | Your battery state-of-charge sensor |
| `number.evcharger2_operating_state` | Your charger operating state entity |
| `number.evcharger2_charge_current` | Your charger charge current entity |
| `input_boolean.ev_override_boost` | Your chosen override helper if renamed |
| `input_number.ev_boost_duration` | Your chosen boost-time helper if renamed |
| `input_number.ev_ui_amperage_slider` | Your chosen dashboard amperage slider helper if renamed |

### Example mapping

```yaml name=entity-mapping-example.yaml
# Replace these with your real entities

sensor.smoothed_solar_kw: sensor.your_solar_sensor
sensor.foxess_bat_soc: sensor.your_battery_soc
number.evcharger2_operating_state: number.your_charger_operating_state
number.evcharger2_charge_current: number.your_charger_charge_current
```

---

## File details

## 1) Templates YAML

This file improves the readability of the charger in Home Assistant by exposing more user-friendly sensor outputs.

Typical uses:

- turn raw charger codes into readable states,
- show charger power in a clearer format,
- provide simple dashboard entities.

### What you will usually need to edit
- Charger entity IDs
- Any source sensors referenced by the templates

---

## 2) Solar EV Adjust

This is the main logic for **dynamic charging current control**.

It adjusts charging behavior based on:

- solar generation,
- battery state of charge,
- time of day,
- daylight status,
- whether manual boost is active,
- charger operating state,
- and live charging amperage.

### Key values to review

| Setting | Example | Meaning |
| --- | ---: | --- |
| `max_grid_allowance` | `14.1` | Maximum grid import you are willing to allow |
| `amps_per_kw_ratio` | `0.5` | Additional charging amps per kW of solar during non-free periods |
| `solar_allowance` | `0` | Solar headroom before extra charging amps are added |
| `phase_2_compensation_amps` | `4` | Base extra amps allowed during free/cheap power periods |
| `battery_charge_kw` | `9.8` | Power to reserve for charging the house battery |

### Notes on tuning
- If charging is too aggressive, reduce `amps_per_kw_ratio`
- If you want more solar headroom before EV charging ramps up, increase `solar_allowance`
- If your free-power period can support a higher base charge, increase `phase_2_compensation_amps`
- If your battery should take priority, ensure `battery_charge_kw` matches its real charging demand

### Example variables used in the logic

```yaml name=solar-ev-adjust-example.yaml
solar_kw: "{{ states('sensor.smoothed_solar_kw') | float(0) - solar_allowance }}"
battery_soc: "{{ states('sensor.foxess_bat_soc') | int(0) }}"
current_time: "{{ now().strftime('%H:%M') }}"
is_daylight: "{{ states('sun.sun') == 'above_horizon' }}"
manual_boost_active: "{{ is_state('input_boolean.ev_override_boost', 'on') }}"
operating_state: "{{ states('number.evcharger2_operating_state') | int(0) }}"
live_amps: "{{ states('number.evcharger2_charge_current') | float(0) }}"
```

### Important
You will need to adjust:
- all sensor entity IDs,
- any battery-related entities,
- all charger-related entities,
- and the time windows in the file to match your tariff or preferred schedule.

---

## 3) EV Manual Boost Controller

This automation allows you to temporarily override solar-based charging logic and run a timed charging session.

### How it works
- Turn on `input_boolean.ev_override_boost`
- Set a duration using `input_number.ev_boost_duration`
- The boost runs until the timer expires
- The boost can be extended by changing the duration while active

### Suggested boost helper settings
- Minimum: `0.5`
- Step: `0.25` or `0.5`
- Maximum: `12` or more if desired
- Unit: `h`

You can also pair this with a calendar if you want scheduled manual boosts.

---

## 4) EV UI Slider True Two-Way Sync Bridge

LocalTuya may allow the charger current to be set to `0`, while the charger itself may only support a minimum of **6A**.

This bridge helps clean up the Home Assistant UI by:

- reading the real charger current,
- writing valid values back to the charger,
- keeping the dashboard slider in sync,
- and preventing out-of-range current values in normal use.

### Recommended slider range
- Minimum: `6A`
- Maximum: `32A`
- Step: `1A`

Adjust the max to suit your charger and installation.

---

## 5) EV Scheduled Boost Sync (Optional)

This optional automation lets you run a boost from a Home Assistant calendar, for example using a calendar named:

`EV Boost Schedule`

### Notes
- Includes an example battery cutoff of **40% SOC**
- You should adjust this threshold to match your own preference
- Make sure the calendar entity name matches your system

Good use cases:
- overnight scheduled boosts,
- pre-departure charging windows,
- manual planning without editing automations.

---

## How the system works

The general control strategy is:

1. Avoid clunky charger on/off toggling where possible
2. Use charger schedule/current control instead
3. Adapt charging current to available solar and household conditions
4. Allow manual override when needed
5. Optionally allow scheduled/calendar-based boost behavior

This gives smoother control and is usually easier to live with than hard start/stop automations.

---

## Configuration checklist

Before enabling everything, confirm you have updated:

- [ ] Charger entity IDs
- [ ] Solar sensor entity
- [ ] Battery SOC entity
- [ ] Any battery power/reserve values
- [ ] Time windows for cheap/free power periods
- [ ] Grid import allowance
- [ ] Current limits and slider ranges
- [ ] Calendar name and SOC cutoff if using scheduled boost

---

## Recommended testing order

Test in this order to make troubleshooting easier:

1. **Templates YAML**
   - Confirm the sensors appear
   - Confirm they show sensible values

2. **EV UI Slider Sync**
   - Confirm the slider reflects the real charger current
   - Confirm it does not allow invalid values below 6A

3. **Manual Boost**
   - Turn on boost manually
   - Confirm it starts and ends correctly
   - Confirm changing the duration updates the stop time

4. **Solar EV Adjust**
   - Confirm solar values are being read correctly
   - Confirm charging current changes as expected

5. **Scheduled Boost**
   - Confirm the calendar entity is correct
   - Confirm scheduled boosts start and stop correctly

---

## Troubleshooting

### Charger does not respond
- Confirm the charger is available in LocalTuya
- Confirm the correct entity IDs are being used
- Confirm the relevant DP mappings are correct in LocalTuya

### Slider allows invalid values
- Check that the UI slider helper is set to a minimum of `6`
- Confirm the sync automation is enabled and using the correct charger current entity

### Solar automation is not changing charge current
- Verify the solar sensor and battery sensors return valid numeric values
- Check template rendering in **Developer Tools → Templates**
- Confirm the automation conditions and time windows match your tariff plan

### Manual boost does not stop
- Check the `input_number.ev_boost_duration` helper value
- Confirm the automation has the correct triggers and timing logic
- Confirm no other automation is immediately re-enabling boost

### Scheduled boost does not run
- Confirm the calendar exists and is named correctly
- Confirm the automation references the correct calendar entity
- Check whether your battery SOC cutoff is preventing the boost from running

### Charger state looks wrong
- Review the template sensor mappings
- Confirm the raw charger state values match the logic in your templates

---

## Customisation notes

This package is not completely plug-and-play. You should expect to tailor it to:

- your charger entity names,
- your inverter/battery integration,
- your tariff schedule,
- your acceptable grid import level,
- your solar surplus strategy,
- and your preferred charging behavior.

If you use a battery system other than Fox ESS, replace the battery-specific entities and assumptions with equivalents from your own integration.

---

## Safety notice

This package can affect EV charging current and power usage. Review all settings carefully before enabling unattended automation.

You are responsible for ensuring:

- your configured current limits are safe,
- your circuit and supply limits are respected,
- your charger supports the values being sent,
- and your household load constraints are understood.

**Test with conservative settings first** before relying on the automations day to day.

---

## Suggested next improvements for this repo

If you want to make this repository easier for other users to adopt, consider adding:

- example YAML files with filenames matching the sections above,
- screenshots of the required LocalTuya entities,
- a sample dashboard card,
- a “known working entity mapping” example,
- and a dedicated troubleshooting section for common charger state codes.

---

## Credits / dependencies

- **LocalTuya**: `https://github.com/make-all/tuya-local`
- **Tuya Developer Portal**: `https://developer.tuya.com/en/`

---

## Final note

The examples in this repository are based on one working setup and should be treated as a starting point rather than a universal configuration. Review every entity ID, limit, and time window before use.
