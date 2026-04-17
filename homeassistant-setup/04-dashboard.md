# Home Assistant Setup — Lovelace Dashboard

## Overview

The dashboard is split into 6 views (tabs):

| View | Icon | Contents |
|------|------|----------|
| Home | `mdi:home` | Quick status chips, active devices summary, scene buttons |
| Lighting | `mdi:lightbulb` | All lights grouped by room |
| Climate | `mdi:thermostat` | All AC units (Samsung, LG, Vivax) |
| Ventilation | `mdi:air-filter` | Konfortvent full control card |
| Blinds | `mdi:blinds` | VELUX blinds per room |
| Settings | `mdi:cog` | Automation toggles, presence, debug |

---

## How to apply this dashboard

1. In HA: **Settings → Dashboards → Add Dashboard** → name it `Home`
2. Open the dashboard → top-right **⋮** menu → **Edit Dashboard**
3. Click **⋮** again → **Raw configuration editor**
4. Paste the YAML below, replacing placeholder entity IDs with your actual ones

---

## Dashboard YAML

> **Entity ID placeholders** — replace these with your real entity IDs:
> - `light.living_room` → your actual Hue/Govee/IKEA light group or bulb entity
> - `climate.samsung_ac` → your Samsung AC climate entity
> - `climate.lg_ac` → your LG AC climate entity
> - `climate.vivax_ac` → your Vivax AC climate entity
> - `cover.velux_living_room` → your VELUX cover entity
> - `fan.konfortvent` → your Konfortvent fan entity
> - `sensor.konfortvent_*` → your Konfortvent sensor entities
> - `scene.*` → your HA scenes

```yaml
title: Home
views:

  # ─────────────────────────────────────────────
  # VIEW 1 — HOME OVERVIEW
  # ─────────────────────────────────────────────
  - title: Home
    path: home
    icon: mdi:home
    cards:

      # Status chips row — quick glance at active devices
      - type: custom:mushroom-chips-card
        chips:
          - type: entity
            entity: sensor.outdoor_temperature     # replace with your outdoor temp sensor
            icon: mdi:thermometer
          - type: template
            icon: mdi:lightbulb
            content: >
              {{ states.light | selectattr('state','eq','on') | list | count }} on
            tap_action:
              action: navigate
              navigation_path: /lovelace/lighting
          - type: template
            icon: mdi:air-conditioner
            content: >
              {{ states.climate | selectattr('state','ne','off') | list | count }} active
            tap_action:
              action: navigate
              navigation_path: /lovelace/climate
          - type: entity
            entity: fan.konfortvent
            icon: mdi:air-filter
          - type: template
            icon: mdi:blinds
            content: >
              {{ states.cover | selectattr('state','eq','open') | list | count }} open

      # Scene quick-launch tiles
      - type: custom:mushroom-title-card
        title: Scenes

      - type: grid
        columns: 4
        square: true
        cards:
          - type: custom:button-card
            name: Morning
            icon: mdi:weather-sunrise
            tap_action:
              action: call-service
              service: scene.turn_on
              service_data:
                entity_id: scene.morning
            styles:
              card:
                - background: linear-gradient(135deg, #f6d365, #fda085)
                - border-radius: 16px
              icon:
                - color: white
              name:
                - color: white

          - type: custom:button-card
            name: Movie
            icon: mdi:movie-open
            tap_action:
              action: call-service
              service: scene.turn_on
              service_data:
                entity_id: scene.movie
            styles:
              card:
                - background: linear-gradient(135deg, #2c3e50, #4ca1af)
                - border-radius: 16px
              icon:
                - color: white
              name:
                - color: white

          - type: custom:button-card
            name: Evening
            icon: mdi:weather-sunset
            tap_action:
              action: call-service
              service: scene.turn_on
              service_data:
                entity_id: scene.evening
            styles:
              card:
                - background: linear-gradient(135deg, #f093fb, #f5576c)
                - border-radius: 16px
              icon:
                - color: white
              name:
                - color: white

          - type: custom:button-card
            name: Night
            icon: mdi:weather-night
            tap_action:
              action: call-service
              service: scene.turn_on
              service_data:
                entity_id: scene.night
            styles:
              card:
                - background: linear-gradient(135deg, #1a1a2e, #16213e)
                - border-radius: 16px
              icon:
                - color: white
              name:
                - color: white

      # Active alerts / reminders (template card)
      - type: custom:mushroom-title-card
        title: Active

      - type: custom:mushroom-template-card
        primary: "{{ states('sensor.konfortvent_temperature_supply') }}°C supply air"
        secondary: "Konfortvent — {{ states('select.konfortvent_mode') }} mode"
        icon: mdi:air-filter
        icon_color: >
          {% if is_state('fan.konfortvent', 'on') %}blue{% else %}grey{% endif %}
        tap_action:
          action: navigate
          navigation_path: /lovelace/ventilation


  # ─────────────────────────────────────────────
  # VIEW 2 — LIGHTING
  # ─────────────────────────────────────────────
  - title: Lighting
    path: lighting
    icon: mdi:lightbulb
    cards:

      - type: custom:mushroom-title-card
        title: Living Room
        subtitle: Philips Hue

      - type: horizontal-stack
        cards:
          - type: custom:mushroom-light-card
            entity: light.living_room
            name: Living Room
            show_brightness_control: true
            show_color_temp_control: true
            show_color_control: true
            collapsible_controls: true

          - type: custom:mushroom-light-card
            entity: light.living_room_strip     # Govee LED strip example
            name: LED Strip
            show_brightness_control: true
            show_color_control: true
            collapsible_controls: true

      - type: custom:mushroom-title-card
        title: Bedroom

      - type: horizontal-stack
        cards:
          - type: custom:mushroom-light-card
            entity: light.bedroom
            name: Bedroom
            show_brightness_control: true
            show_color_temp_control: true
            collapsible_controls: true

          - type: custom:mushroom-light-card
            entity: light.bedroom_nightstand
            name: Nightstand
            show_brightness_control: true
            show_color_temp_control: true
            collapsible_controls: true

      - type: custom:mushroom-title-card
        title: Kitchen

      - type: horizontal-stack
        cards:
          - type: custom:mushroom-light-card
            entity: light.kitchen
            name: Kitchen
            show_brightness_control: true
            show_color_temp_control: true
            collapsible_controls: true

          - type: custom:mushroom-light-card
            entity: light.kitchen_counter      # IKEA under-cabinet
            name: Counter
            show_brightness_control: true
            collapsible_controls: true

      - type: custom:mushroom-title-card
        title: All Lights

      - type: custom:mushroom-entity-card
        entity: light.all_lights               # create a light group in HA for all lights
        name: All Lights
        icon: mdi:lightbulb-group
        tap_action:
          action: toggle


  # ─────────────────────────────────────────────
  # VIEW 3 — CLIMATE (AC)
  # ─────────────────────────────────────────────
  - title: Climate
    path: climate
    icon: mdi:thermostat
    cards:

      - type: custom:mushroom-title-card
        title: Air Conditioning

      # Samsung AC
      - type: custom:simple-thermostat
        entity: climate.samsung_ac
        name: Samsung AC
        header:
          toggle:
            entity: climate.samsung_ac
        sensors:
          - entity: sensor.living_room_temperature
            name: Room temp
        control:
          hvac:
            heat:
              name: Heat
            cool:
              name: Cool
            heat_cool:
              name: Auto
            fan_only:
              name: Fan
            dry:
              name: Dry
          fan:
            auto:
              name: Auto
            low:
              name: Low
            medium:
              name: Med
            high:
              name: High
        step_size: 0.5

      # LG AC
      - type: custom:simple-thermostat
        entity: climate.lg_ac
        name: LG AC
        header:
          toggle:
            entity: climate.lg_ac
        sensors:
          - entity: sensor.bedroom_temperature
            name: Room temp
        control:
          hvac:
            heat:
              name: Heat
            cool:
              name: Cool
            heat_cool:
              name: Auto
            fan_only:
              name: Fan
            dry:
              name: Dry
          fan:
            auto:
              name: Auto
            low:
              name: Low
            medium:
              name: Med
            high:
              name: High
        step_size: 0.5

      # Vivax AC
      - type: custom:simple-thermostat
        entity: climate.vivax_ac
        name: Vivax AC
        header:
          toggle:
            entity: climate.vivax_ac
        control:
          hvac:
            heat:
              name: Heat
            cool:
              name: Cool
            heat_cool:
              name: Auto
            fan_only:
              name: Fan
            dry:
              name: Dry
          fan:
            auto:
              name: Auto
            low:
              name: Low
            medium:
              name: Med
            high:
              name: High
        step_size: 0.5

      # Temperature history graph
      - type: custom:mushroom-title-card
        title: Temperature History

      - type: custom:mini-graph-card
        entities:
          - entity: sensor.living_room_temperature
            name: Living Room
          - entity: sensor.bedroom_temperature
            name: Bedroom
          - entity: sensor.outdoor_temperature
            name: Outdoor
        hours_to_show: 24
        points_per_hour: 2
        line_width: 2
        smoothing: true
        show:
          labels: true
          points: false
          legend: true


  # ─────────────────────────────────────────────
  # VIEW 4 — VENTILATION (Konfortvent)
  # ─────────────────────────────────────────────
  - title: Ventilation
    path: ventilation
    icon: mdi:air-filter
    cards:

      - type: custom:mushroom-title-card
        title: Konfortvent HRV
        subtitle: Heat Recovery Ventilation

      # Main control card
      - type: custom:mushroom-fan-card
        entity: fan.konfortvent
        name: Konfortvent
        show_percentage_control: true
        show_oscillate_control: false
        icon_animation: true

      # Mode selector
      - type: custom:mushroom-title-card
        title: Mode

      - type: custom:button-card
        entity: select.konfortvent_mode
        name: Ventilation Mode
        show_state: true
        tap_action:
          action: call-service
          service: select.select_option
          service_data:
            entity_id: select.konfortvent_mode
            option: normal
        hold_action:
          action: more-info

      - type: grid
        columns: 4
        square: false
        cards:
          - type: custom:button-card
            name: Eco
            icon: mdi:leaf
            tap_action:
              action: call-service
              service: select.select_option
              service_data:
                entity_id: select.konfortvent_mode
                option: eco
            state_display: >
              [[[
                return states['select.konfortvent_mode'].state === 'eco' ? 'Active' : ''
              ]]]
            styles:
              card:
                - border-radius: 12px
                - background: >
                    [[[
                      return states['select.konfortvent_mode'].state === 'eco'
                        ? 'var(--primary-color)'
                        : 'var(--card-background-color)';
                    ]]]

          - type: custom:button-card
            name: Normal
            icon: mdi:fan
            tap_action:
              action: call-service
              service: select.select_option
              service_data:
                entity_id: select.konfortvent_mode
                option: normal
            styles:
              card:
                - border-radius: 12px
                - background: >
                    [[[
                      return states['select.konfortvent_mode'].state === 'normal'
                        ? 'var(--primary-color)'
                        : 'var(--card-background-color)';
                    ]]]

          - type: custom:button-card
            name: Boost
            icon: mdi:fan-plus
            tap_action:
              action: call-service
              service: select.select_option
              service_data:
                entity_id: select.konfortvent_mode
                option: boost
            styles:
              card:
                - border-radius: 12px
                - background: >
                    [[[
                      return states['select.konfortvent_mode'].state === 'boost'
                        ? '#f5576c'
                        : 'var(--card-background-color)';
                    ]]]

          - type: custom:button-card
            name: Away
            icon: mdi:home-export-outline
            tap_action:
              action: call-service
              service: select.select_option
              service_data:
                entity_id: select.konfortvent_mode
                option: away
            styles:
              card:
                - border-radius: 12px
                - background: >
                    [[[
                      return states['select.konfortvent_mode'].state === 'away'
                        ? '#f6d365'
                        : 'var(--card-background-color)';
                    ]]]

      # Sensor readings
      - type: custom:mushroom-title-card
        title: Sensors

      - type: horizontal-stack
        cards:
          - type: custom:mushroom-entity-card
            entity: sensor.konfortvent_temperature_supply
            name: Supply
            icon: mdi:thermometer-chevron-up
            icon_color: orange

          - type: custom:mushroom-entity-card
            entity: sensor.konfortvent_temperature_extract
            name: Extract
            icon: mdi:thermometer-chevron-down
            icon_color: blue

          - type: custom:mushroom-entity-card
            entity: sensor.konfortvent_humidity
            name: Humidity
            icon: mdi:water-percent
            icon_color: teal

      # Temperature history
      - type: custom:mini-graph-card
        entities:
          - entity: sensor.konfortvent_temperature_supply
            name: Supply air
            color: "#f6a623"
          - entity: sensor.konfortvent_temperature_extract
            name: Extract air
            color: "#4a90d9"
        hours_to_show: 12
        points_per_hour: 4
        line_width: 2
        smoothing: true
        show:
          labels: true
          legend: true
        name: Temperature History


  # ─────────────────────────────────────────────
  # VIEW 5 — BLINDS (VELUX)
  # ─────────────────────────────────────────────
  - title: Blinds
    path: blinds
    icon: mdi:blinds
    cards:

      - type: custom:mushroom-title-card
        title: VELUX Blinds

      # All blinds quick control
      - type: horizontal-stack
        cards:
          - type: custom:button-card
            name: All Open
            icon: mdi:blinds-open
            tap_action:
              action: call-service
              service: cover.open_cover
              service_data:
                entity_id: group.all_velux_covers    # create a cover group in HA
            styles:
              card:
                - border-radius: 12px

          - type: custom:button-card
            name: All Close
            icon: mdi:blinds
            tap_action:
              action: call-service
              service: cover.close_cover
              service_data:
                entity_id: group.all_velux_covers
            styles:
              card:
                - border-radius: 12px

          - type: custom:button-card
            name: All 50%
            icon: mdi:blinds-horizontal
            tap_action:
              action: call-service
              service: cover.set_cover_position
              service_data:
                entity_id: group.all_velux_covers
                position: 50
            styles:
              card:
                - border-radius: 12px

      - type: custom:mushroom-title-card
        title: Individual Blinds

      - type: custom:mushroom-cover-card
        entity: cover.velux_living_room
        name: Living Room
        show_buttons_control: true
        show_position_control: true

      - type: custom:mushroom-cover-card
        entity: cover.velux_bedroom
        name: Bedroom
        show_buttons_control: true
        show_position_control: true

      - type: custom:mushroom-cover-card
        entity: cover.velux_kitchen
        name: Kitchen
        show_buttons_control: true
        show_position_control: true

      - type: custom:mushroom-cover-card
        entity: cover.velux_office
        name: Office
        show_buttons_control: true
        show_position_control: true


  # ─────────────────────────────────────────────
  # VIEW 6 — SETTINGS / AUTOMATIONS
  # ─────────────────────────────────────────────
  - title: Settings
    path: settings
    icon: mdi:cog
    cards:

      - type: custom:mushroom-title-card
        title: Automation Toggles

      - type: grid
        columns: 2
        square: false
        cards:
          - type: custom:mushroom-entity-card
            entity: automation.blinds_open_at_sunrise
            name: Blinds at Sunrise
            icon: mdi:weather-sunrise

          - type: custom:mushroom-entity-card
            entity: automation.blinds_close_at_sunset
            name: Blinds at Sunset
            icon: mdi:weather-sunset

          - type: custom:mushroom-entity-card
            entity: automation.motion_lights_living_room
            name: Motion Lights
            icon: mdi:motion-sensor

          - type: custom:mushroom-entity-card
            entity: automation.ac_schedule
            name: AC Schedule
            icon: mdi:clock-outline

          - type: custom:mushroom-entity-card
            entity: automation.konfortvent_auto_mode
            name: Ventilation Auto
            icon: mdi:air-filter

          - type: custom:mushroom-entity-card
            entity: automation.presence_based_lighting
            name: Presence Lights
            icon: mdi:home-account

      - type: custom:mushroom-title-card
        title: System

      - type: custom:mushroom-entity-card
        entity: sensor.processor_use_percent     # if HA system monitor integration added
        name: HA CPU Usage
        icon: mdi:cpu-64-bit

      - type: custom:mushroom-entity-card
        entity: sensor.memory_use_percent
        name: HA Memory
        icon: mdi:memory
```

---

## Scenes to create in HA

Create these scenes at **Settings → Scenes → Add Scene**:

| Scene | Lights | AC | Blinds |
|-------|--------|----|--------|
| `scene.morning` | All lights 80% warm white | AC auto 21°C | Blinds open |
| `scene.evening` | Living room 40% warm amber | AC auto 22°C | Blinds closed |
| `scene.movie` | Living room 10% dim, TV backlight on | AC cool 22°C | Blinds closed |
| `scene.night` | All lights off | AC off | Blinds closed |
| `scene.away` | All lights off | AC off | Blinds 50% |

---

## Light groups to create in HA

Create these groups at **Settings → Devices & Services → Helpers → Group → Light group**:

| Group entity | Members |
|-------------|---------|
| `light.all_lights` | All light entities |
| `light.living_room` | All living room lights |
| `light.bedroom` | All bedroom lights |
| `light.kitchen` | All kitchen lights |

---

## Cover group to create in HA

In `configuration.yaml`:

```yaml
cover:
  - platform: group
    name: All VELUX Covers
    unique_id: all_velux_covers
    entities:
      - cover.velux_living_room
      - cover.velux_bedroom
      - cover.velux_kitchen
      - cover.velux_office
```
