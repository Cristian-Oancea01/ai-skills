# Home Assistant Setup — Automations

All automations go in `/config/automations.yaml` or can be created via the HA UI at
**Settings → Automations & Scenes → Create Automation → Edit in YAML**.

Replace placeholder entity IDs with your actual entity IDs before applying.

---

## 1. VELUX Blinds — Open at Sunrise

Opens all blinds 30 minutes after local sunrise.

```yaml
alias: blinds_open_at_sunrise
description: Open VELUX blinds 30 min after sunrise
trigger:
  - platform: sun
    event: sunrise
    offset: "+00:30:00"
condition:
  - condition: state
    entity_id: input_boolean.vacation_mode    # optional: skip when away
    state: "off"
action:
  - service: cover.open_cover
    target:
      entity_id:
        - cover.velux_living_room
        - cover.velux_bedroom
        - cover.velux_kitchen
        - cover.velux_office
mode: single
```

---

## 2. VELUX Blinds — Close at Sunset

Closes all blinds at sunset.

```yaml
alias: blinds_close_at_sunset
description: Close VELUX blinds at sunset
trigger:
  - platform: sun
    event: sunset
    offset: "00:00:00"
action:
  - service: cover.close_cover
    target:
      entity_id:
        - cover.velux_living_room
        - cover.velux_bedroom
        - cover.velux_kitchen
        - cover.velux_office
mode: single
```

---

## 3. VELUX Blinds — Close on High Temperature

Closes south-facing blinds automatically when outdoor temperature exceeds 28°C
(reduces solar gain and keeps the house cooler).

```yaml
alias: blinds_close_on_heat
description: Close blinds when outdoor temp exceeds 28°C during daytime
trigger:
  - platform: numeric_state
    entity_id: sensor.outdoor_temperature
    above: 28
condition:
  - condition: sun
    after: sunrise
    before: sunset
action:
  - service: cover.set_cover_position
    target:
      entity_id:
        - cover.velux_living_room
        - cover.velux_kitchen
    data:
      position: 20    # mostly closed — still lets some light in
mode: single
```

---

## 4. Motion-Activated Lights — Living Room

Turns on living room lights when motion is detected. Turns off after 5 minutes of no motion.
Only active when lux level is low (dark enough to need lights).

```yaml
alias: motion_lights_living_room
description: Turn on living room lights on motion when dark
trigger:
  - platform: state
    entity_id: binary_sensor.hue_motion_living_room    # Hue motion sensor
    to: "on"
condition:
  - condition: numeric_state
    entity_id: sensor.hue_motion_living_room_illuminance   # Hue lux sensor
    below: 50    # lux threshold — only activate when darker than 50 lux
action:
  - service: light.turn_on
    target:
      entity_id: light.living_room
    data:
      brightness_pct: 60
      color_temp: 3500    # warm white in Kelvin
  - wait_for_trigger:
      - platform: state
        entity_id: binary_sensor.hue_motion_living_room
        to: "off"
        for:
          minutes: 5
  - service: light.turn_off
    target:
      entity_id: light.living_room
mode: restart
```

---

## 5. AC Schedule — Weekday Morning

Turns on AC to a comfortable temperature on weekday mornings before wake-up.

```yaml
alias: ac_schedule_weekday_morning
description: Pre-cool or pre-heat the bedroom before waking up on weekdays
trigger:
  - platform: time
    at: "06:30:00"
condition:
  - condition: time
    weekday:
      - mon
      - tue
      - wed
      - thu
      - fri
action:
  - service: climate.set_temperature
    target:
      entity_id: climate.lg_ac     # bedroom AC
    data:
      temperature: 22
      hvac_mode: >
        {% if states('sensor.outdoor_temperature') | float > 20 %}
          cool
        {% else %}
          heat
        {% endif %}
mode: single
```

---

## 6. AC Turn Off When Window/Door Opens

Turns off AC if a window or door is left open for more than 5 minutes (avoids wasting energy).

> Requires a window/door sensor. Replace entity IDs as needed.

```yaml
alias: ac_off_when_window_open
description: Turn off AC if window open for 5+ minutes
trigger:
  - platform: state
    entity_id: binary_sensor.living_room_window
    to: "on"
    for:
      minutes: 5
condition:
  - condition: not
    conditions:
      - condition: state
        entity_id: climate.samsung_ac
        state: "off"
action:
  - service: climate.turn_off
    target:
      entity_id: climate.samsung_ac
  - service: notify.mobile_app    # optional: notify on phone
    data:
      message: "AC turned off — living room window has been open for 5 minutes."
mode: single
```

---

## 7. Konfortvent — Auto Boost on High Humidity

Boosts ventilation automatically when humidity rises above 65% (e.g. after shower or cooking).

```yaml
alias: konfortvent_boost_on_humidity
description: Boost ventilation when humidity is high
trigger:
  - platform: numeric_state
    entity_id: sensor.konfortvent_humidity
    above: 65
action:
  - service: select.select_option
    target:
      entity_id: select.konfortvent_mode
    data:
      option: boost
  - wait_for_trigger:
      - platform: numeric_state
        entity_id: sensor.konfortvent_humidity
        below: 55
    timeout:
      minutes: 30    # return to normal after 30 min max even if humidity stays high
  - service: select.select_option
    target:
      entity_id: select.konfortvent_mode
    data:
      option: normal
mode: single
```

---

## 8. Konfortvent — Away Mode When Everyone Leaves

Switches ventilation to away/eco mode when all residents leave home.
Requires a presence detection method (phone GPS tracker, person entity, etc.).

```yaml
alias: konfortvent_away_mode
description: Set ventilation to away mode when nobody is home
trigger:
  - platform: state
    entity_id: group.all_people    # a group of person.* entities
    to: "not_home"
action:
  - service: select.select_option
    target:
      entity_id: select.konfortvent_mode
    data:
      option: away
mode: single

---

alias: konfortvent_return_home
description: Restore normal ventilation when someone arrives home
trigger:
  - platform: state
    entity_id: group.all_people
    to: "home"
action:
  - service: select.select_option
    target:
      entity_id: select.konfortvent_mode
    data:
      option: normal
mode: single
```

---

## 9. Presence-Based Lighting — Welcome Home

Turns on entrance/living room lights to a welcoming scene when someone arrives home after dark.

```yaml
alias: presence_welcome_home
description: Turn on lights when arriving home after dark
trigger:
  - platform: state
    entity_id: person.your_name    # replace with your person entity
    to: "home"
condition:
  - condition: sun
    after: sunset
action:
  - service: scene.turn_on
    target:
      entity_id: scene.evening
mode: single
```

---

## 10. Good Night Routine

Triggered manually (via dashboard button or voice) or at a set time.
Turns off all lights, closes all blinds, sets AC to night mode.

```yaml
alias: good_night_routine
description: Turn off lights, close blinds, set AC to night mode
trigger:
  - platform: time
    at: "23:00:00"    # optional: also auto-trigger at 11pm
action:
  - service: scene.turn_on
    target:
      entity_id: scene.night
  - service: cover.close_cover
    target:
      entity_id:
        - cover.velux_living_room
        - cover.velux_bedroom
        - cover.velux_kitchen
        - cover.velux_office
  - service: climate.set_temperature
    target:
      entity_id:
        - climate.samsung_ac
        - climate.lg_ac
        - climate.vivax_ac
    data:
      temperature: 20
      hvac_mode: >
        {% if states('sensor.outdoor_temperature') | float > 18 %}
          cool
        {% else %}
          off
        {% endif %}
mode: single
```

---

## Helper entities needed for automations

Create these at **Settings → Devices & Services → Helpers**:

| Helper | Type | Purpose |
|--------|------|---------|
| `input_boolean.vacation_mode` | Toggle | Suppress automations while on vacation |
| `group.all_people` | Person group | Track household presence |

To create a person group, add to `configuration.yaml`:

```yaml
group:
  all_people:
    name: All People
    entities:
      - person.your_name
      - person.partner_name    # add all residents
```
