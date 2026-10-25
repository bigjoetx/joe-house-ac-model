# Whole-House Thermostat Simulator

An interactive, single-file HTML simulator for a two-story Houston house with two 5-ton AC systems. Set a thermostat schedule and see what each unit does on a typical sunny August day: run time, cycling, dehumidifying time, hourly energy, peak demand, and monthly cost.

The model is calibrated to real data: one-minute attic and supply-vent sensors upstairs, and the smart meter's 15-minute readings for the whole house.

## Running it

Open `whole-house-thermostat-simulator.html` in any modern browser. Everything is in that one file: the data, the model and the charts. There's no server, no build step and no dependencies beyond a Google Fonts stylesheet, which falls back to system fonts when you're offline.

## Controls

| Control | What it does |
|---|---|
| Normal setpoint | The temperature both floors hold outside the setback window |
| Setback setpoint | The temperature during the setback window |
| Setback starts / back to normal at | The setback window, in 30-minute steps. It can wrap past midnight. |
| Thermostat swing | The gap between the AC turning on and off (0.5–3°F). A wider swing means longer, less frequent runs. |
| Upstairs equipment | Current 5-ton single-stage, or variable speed (estimate) |
| Downstairs equipment | New 5-ton two-stage, or the old 30-year single-stage unit |
| Electricity rate | ¢/kWh, used for monthly cost |

Preset buttons load common schedules. The comparison table simulates all of them live with your current swing, equipment and rate. Click a row to load it.

## Outputs

- **Whole house:** kWh/day, cost per August month, AC-only kWh, and 7–8 pm demand. Each is compared with holding the normal setpoint all day on both floors.
- **Per floor:** kWh, number of starts, median run length, hours running, and dehumidifying hours.
- **24-hour trace:** both floors' temperatures against the setpoint, with run bars for each unit. Downstairs bars are shaded by stage.
- **Hourly energy:** stacked non-AC, upstairs and downstairs energy, with the no-setback schedule overlaid.
- **Calibration chart:** model against meter for the actual schedule.

## Model

Each floor is a **two-mass thermal model** stepped once per minute. One mass is the indoor air, which the thermostat reads; the other is the house structure and furnishings. Heat enters through:

- walls and windows, from outdoor temperature
- the ceiling, from attic temperature
- west-facing sun, using a solar shape that peaks in late afternoon
- internal gains from people and electronics

The AC removes heat at a capacity that falls with outdoor temperature (−0.6%/°F) and draws power that rises with it (+0.9%/°F). The thermostat turns on at setpoint + swing/2 and off at setpoint − swing/2, with a 5-minute minimum off time.

- **Single-stage:** full capacity whenever it runs.
- **Two-stage:** starts on low stage (68% capacity, 58% power) and moves to high stage after 20 minutes or when it falls 0.6°F behind.
- **Variable speed (estimate):** modulates between 30% and 100% to hold the setpoint, cycling only below 30%. Part-load power follows a generic inverter curve.

Each scenario runs for three identical days so the thermal masses settle, and the third day is reported.

### Weather

The average of 35 sunny August 2026 days with an attic peak of 115°F or more: outdoor temperature from Ellington Field (KEFD) via the Iowa Environmental Mesonet, and attic temperature from the house's sensor.

### Calibration

- **Upstairs:** fit to vent-sensor run times on those 35 days. It reproduces the hourly run-time pattern, about 11–12 hours of running a day, and roughly 8-minute median runs.
- **Downstairs:** fit so that non-AC load plus upstairs plus downstairs (the old unit, which was installed in August) matches the meter's average hourly profile on 34 hot August 2026 days. Model: 129.8 kWh/day. Meter: 128.5. The fit reproduces the 7 am drop and 7 pm spike from both thermostats setting back and recovering together. The downstairs air mass is fixed at a realistic value, because the meter alone can't pin down cycling behavior.
- **Non-AC load:** the winter 2026 hourly profile (gas heat), scaled during the fit to about 34 kWh/day.

## Example results (sunny August day, current equipment, 15¢/kWh)

| Schedule | House kWh/day | vs. no setback | 7–8 pm demand |
|---|---|---|---|
| No setback (72°F) | 116.8 | — | 6.4 kW |
| 74°F, 7 am–7 pm | 112.7 | −3.5% | 8.3 kW |
| 74°F, noon–7 pm | 113.3 | −3.0% | 8.7 kW |
| 74°F, 7 am–9 pm | 111.5 | −4.6% | 5.3 kW |
| 76°F, 7 am–7 pm | 108.1 | −7.5% | 10.4 kW |

Takeaways: the setback saves roughly $19 per August month and causes an evening demand spike when both floors recover at once. Thermostat swing and equipment matter far more than the schedule.

## Limitations

- **Equipment figures are typical values, not measurements:** about 5.2 kW for the upstairs unit at 95°F, 4.3 kW on high stage for the new downstairs unit, and 6.0 kW for the old one. A CT clamp on each condenser would replace them with measured values.
- **Variable-speed results are rough estimates,** not a quote for any specific unit.
- **"Dehumidifying hours" is a proxy:** run time after the first 5 minutes of each run. It's for comparing schedules; the model doesn't track humidity.
- **The two floors are modeled separately.** Heat moving between them is absorbed into the calibration, which works because both thermostats follow the same schedule.
- **It models one sunny August day.** Shoulder months will show smaller absolute differences.

## Data privacy

The calibration used Smart Meter Texas interval exports, which include an ESIID tied to the service address. The HTML file contains only derived parameters and averaged hourly profiles, with no raw meter data and no identifiers.
