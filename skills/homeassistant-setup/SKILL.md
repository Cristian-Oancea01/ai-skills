---
name: homeassistant-setup
description: Complete Home Assistant setup on QNAP NAS — Docker, integrations, dashboard, and automations for Hue, Govee, IKEA, VELUX, Samsung, LG, Vivax, and Google Home
license: MIT
compatibility: opencode
---

## Live system location

- **NAS host**: 192.168.68.106
- **HA config path on NAS**: `\\192.168.68.106\Public\HomeAssistantConfig\` (SMB)
- **HA web UI**: http://192.168.68.106:8123
- **Credentials**: stored locally at `C:\Users\oance\.config\opencode\local\nas_access.md` — never commit

## Accessing the NAS in an OpenCode session

Always mount the SMB share before reading/writing config files. Credentials are in `local/nas_access.md`.

```powershell
# Mount (get password from C:\Users\oance\.config\opencode\local\nas_access.md)
$cred = New-Object System.Management.Automation.PSCredential(
    "MCP",
    (ConvertTo-SecureString "<password>" -AsPlainText -Force)
)
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\192.168.68.106\Public" -Credential $cred -Persist
# HA config now at Z:\HomeAssistantConfig\

# Unmount when done
Remove-PSDrive Z
```

### Backup before any changes
```powershell
# Script saved at: C:\Users\oance\AppData\Local\Temp\nas_backup.ps1
powershell -ExecutionPolicy Bypass -File "C:\Users\oance\AppData\Local\Temp\nas_backup.ps1"
# Creates: Z:\HomeAssistantConfig_backup_<yyyyMMdd_HHmmss>\
```

### MCP server (for NAS system queries, not file editing)
- URL and token in `C:\Users\oance\.config\opencode\opencode.json`
- Reusable call script: `C:\Users\oance\AppData\Local\Temp\mcp_list_files.ps1`
- **Note**: MCP `list_files` times out on large shares — use SMB mount instead for file access

## Architecture

```
Internet
   │
   ▼
QNAP NAS (Container Station)
   │
   └── Docker: homeassistant (host network mode)
            │
            ├── Philips Hue Bridge  (local LAN)   ✅ working
            ├── Komfovent HRV       (Modbus)       ✅ working
            ├── Miele Dishwasher    (cloud)         ⚠️ token expired
            ├── Google Cast         (local)         ✅ working
            ├── Govee devices       (local LAN)     ❌ not set up
            ├── IKEA DIRIGERA       (local LAN)     ❌ not set up
            ├── VELUX               (cloud)         ❌ not set up
            ├── Samsung SmartThings (cloud)         ❌ not set up
            ├── LG ThinQ            (cloud)         ❌ not set up
            ├── Vivax AC            (Tuya local)    ❌ not set up
            └── Google Home         (manual OAuth)  ❌ not set up
```

## Goals

- Single dashboard to control all smart devices
- Good-looking Lovelace UI with Mushroom + custom cards
- Automations for comfort and energy efficiency
- Google Home voice control + HA controls Google Home devices
- No reliance on Nabu Casa — self-hosted with manual HTTPS

## Device Inventory

### Lighting
| Device | Brand | Integration |
|--------|-------|-------------|
| Hue bulbs, switches, sensors | Philips Hue | Local Hue Bridge (built-in) |
| LED strips, bulbs | Govee | Local LAN (`govee_lights_local` HACS) |
| TRADFRI bulbs, outlets | IKEA | Local DIRIGERA hub (built-in) |

### Climate
| Device | Brand | Integration |
|--------|-------|-------------|
| Vivax AC | Vivax | Tuya local or SmartIR fallback |
| Samsung AC | Samsung | SmartThings cloud (built-in) |
| LG AC | LG | LG ThinQ (`thinq2-ha` HACS) |

### Other
| Device | Brand | Integration |
|--------|-------|-------------|
| Komfovent HRV | Komfovent | Custom component (Modbus) — already integrated ✅ |
| VELUX blinds | VELUX | VELUX cloud (`velux` HACS) |
| Google Nest/Home | Google | Manual Google Assistant OAuth |

## Key Technical Decisions

1. **Host network mode** — HA Docker container must use `network_mode: host` for local device discovery (Hue mDNS, Govee UDP broadcast, DIRIGERA mDNS). Critical on QNAP.
2. **HACS** — Required for Govee LAN, VELUX, LG ThinQ, and all custom dashboard cards.
3. **Google Home manual** — Uses `google_assistant:` config in `configuration.yaml` with a manually created Google Cloud project. Requires HTTPS.
4. **QNAP reverse proxy** — QNAP's Application Portal (or nginx) exposes HA on HTTPS for Google Home.
5. **Vivax path** — First check if Tuya-based with `tinytuya scan`; if yes use `localtuya`; if no, use Broadlink RM4 Pro with `smartir`.

---

## Part 1 — Docker Setup on QNAP

### Prerequisites

- QNAP NAS with Container Station installed
- Static LAN IP assigned to QNAP (e.g. `192.168.1.10`)
- Dedicated folder: `/share/homeassistant/config`
- Port 8123 free on QNAP host

### docker-compose.yml

Create at `/share/homeassistant/docker-compose.yml`:

```yaml
version: "3.8"

services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/home-assistant:stable
    volumes:
      - /share/homeassistant/config:/config
      - /etc/localtime:/etc/localtime:ro
    network_mode: host        # REQUIRED for local device discovery
    restart: unless-stopped
    privileged: true
    environment:
      - TZ=Europe/Zagreb      # Change to your timezone
```

### Deploy

**Container Station GUI:** Create → Create Application → paste compose → name `homeassistant` → Create

**SSH:**
```bash
cd /share/homeassistant
docker compose up -d
```

### HTTPS for Google Home

Using QNAP Application Portal:
1. Control Panel → Application Portal → Reverse Proxy
2. Source: `https://yourdomain.com:443` → Destination: `http://localhost:8123`
3. Enable Let's Encrypt in Control Panel → Security → SSL Certificate

Add to `configuration.yaml`:
```yaml
homeassistant:
  external_url: "https://yourdomain.com"
  internal_url: "http://192.168.1.10:8123"

http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - ::1
```

---

## Part 2 — Integrations

### 1. Philips Hue (Local)

1. Settings → Devices & Services → Add Integration → `Philips Hue`
2. HA auto-discovers the bridge → click it
3. Press physical button on top of Hue Bridge within 30 seconds
4. All lights, sensors, scenes, switches imported automatically

### 2. Govee LAN (HACS)

Enable LAN Control in Govee Home app for each device, then:
1. HACS → Integrations → search `Govee LAN` → Install → Restart HA
2. Settings → Add Integration → `Govee LAN`

Fallback (devices without LAN): Use HACS `Govee` (cloud) with API key from Govee app → Profile → About Us → Request API Key.

### 3. IKEA DIRIGERA (Local)

1. Settings → Add Integration → `IKEA` → select IKEA DIRIGERA
2. Press action button on DIRIGERA hub when prompted
3. All devices imported (lights → `light.*`, plugs → `switch.*`, remotes → `event.*`)

### 4. VELUX (Cloud, HACS)

1. HACS → Integrations → search `VELUX` → Install → Restart HA
2. Settings → Add Integration → `VELUX` → log in with VELUX account
3. Blinds appear as `cover.*` entities

### 5. Samsung AC (SmartThings)

1. Create token at https://account.smartthings.com/tokens (all scopes)
2. Settings → Add Integration → `SmartThings` → enter token
3. AC appears as `climate.*` with heat/cool/fan/dry/auto modes

### 6. LG AC (ThinQ, HACS)

1. HACS → Integrations → search `LG ThinQ` → Install → Restart HA
2. Settings → Add Integration → `LG ThinQ` → log in with LG account

### 7. Vivax AC

**Path A — Tuya (try first):**
```bash
pip install tinytuya
python -m tinytuya scan
```
If Vivax appears: install HACS `localtuya`, get Device ID and Local Key from https://iot.tuya.com, map DPs (1=on/off, 2=target temp, 3=current temp, 4=mode, 5=fan speed).

**Path B — SmartIR (fallback):**
- Hardware: Broadlink RM4 Pro
- HACS: install `smartir`
- Find Vivax IR code at https://github.com/smartHomeHub/SmartIR/tree/master/codes/climate
- Add to `configuration.yaml`:
```yaml
smartir:

climate:
  - platform: smartir
    name: Vivax AC Living Room
    unique_id: vivax_ac_living_room
    device_code: 1180
    controller_data: remote.broadlink_rm4_pro
    temperature_sensor: sensor.living_room_temperature
```

### 8. Google Home (Bidirectional — Manual OAuth)

Requires public HTTPS endpoint.

**Step 1 — Google Cloud project:**
1. https://console.cloud.google.com → new project `HomeAssistant`
2. Enable HomeGraph API
3. Create Service Account → download JSON key → save to `/config/google_credentials.json`

**Step 2 — Actions on Google:**
1. https://console.actions.google.com → new project → Smart Home type
2. Fulfillment URL: `https://yourdomain.com/api/google_assistant`
3. Account Linking: set OAuth Client ID/Secret, auth URL, token URL

**Step 3 — configuration.yaml:**
```yaml
google_assistant:
  project_id: YOUR_GOOGLE_CLOUD_PROJECT_ID
  service_account: !include google_credentials.json
  report_state: true
  exposed_domains:
    - light
    - switch
    - climate
    - cover
    - fan
    - scene
    - script
```

**Step 4:** Google Home app → Add device → Works with Google → link with HA credentials.

### 9. Komfovent (Already Integrated)

No setup needed. Custom component `komfovent` installed via HACS. Modbus at 192.168.68.101:502.

Real entity IDs (verified from running system):
- `climate.komfovent` — main climate entity
- `select.komfovent_current_mode` — normal / boost / away (verify exact option labels in HA UI)
- `select.komfovent_scheduler_mode`
- `select.komfovent_temperature_control`
- `select.komfovent_flow_control`
- `select.komfovent_eco_heat_recovery`
- Humidity/temperature sensors: check Developer Tools → States → filter `komfovent`

---

## Part 3 — HACS & Custom Cards

### Install HACS

```bash
wget -O - https://get.hacs.xyz | bash -
```
Restart HA → Settings → Add Integration → `HACS` → GitHub OAuth.

### Required HACS Integrations

| Integration | HACS repo | Purpose |
|---|---|---|
| Govee LAN | `govee_lights_local` | Local Govee lights |
| VELUX | `velux` | VELUX blinds |
| LG ThinQ | `thinq2-ha` | LG AC |
| LocalTuya | `localtuya` | Vivax (Path A) |
| SmartIR | `smartir` | Vivax (Path B) |

### Required HACS Frontend Cards

| Card | Repo | Use |
|---|---|---|
| Mushroom Cards | `piitaya/lovelace-mushroom` | Primary card set |
| Button Card | `custom-cards/button-card` | Scene tiles, quick actions |
| Mini Graph Card | `kalkih/mini-graph-card` | Temperature/energy history |
| Bubble Card | `Clooos/Bubble-Card` | Pop-up detail controls |
| Simple Thermostat | `nervetattoo/simple-thermostat` | AC control cards |
| Lovelace Swipe Navigation | `maykar/lovelace-swipe-navigation` | Mobile swipe between views |

Theme: Install **Mushroom Themes** via HACS → Frontend.

---

## Part 4 — Lovelace Dashboard

The dashboard has 6 views: Home, Lighting, Climate, Ventilation, Blinds, Settings.

### Entity ID placeholders (replace with real IDs)

- `light.living_room`, `light.bedroom`, `light.kitchen` — light groups
- `climate.samsung_ac`, `climate.lg_ac`, `climate.vivax_ac` — AC entities
- `cover.velux_living_room`, `cover.velux_bedroom`, etc. — VELUX covers
- `climate.komfovent`, `select.komfovent_current_mode`, `sensor.komfovent_*`
- `sensor.outdoor_temperature` — outdoor temp sensor

### Apply dashboard

1. Settings → Dashboards → Add Dashboard → name it `Home`
2. Open → Edit → ⋮ → Raw configuration editor
3. Paste the YAML below

```yaml
title: Home
views:

  - title: Home
    path: home
    icon: mdi:home
    cards:
      - type: custom:mushroom-chips-card
        chips:
          - type: entity
            entity: sensor.outdoor_temperature
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

      - type: custom:mushroom-template-card
        primary: "{{ states('sensor.komfovent_temperature_supply') }}°C supply air"
        secondary: "Komfovent — {{ states('select.komfovent_current_mode') }} mode"
        icon: mdi:air-filter
        icon_color: >
          {% if not is_state('climate.komfovent', 'off') %}blue{% else %}grey{% endif %}
        tap_action:
          action: navigate
          navigation_path: /lovelace/ventilation

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
            show_brightness_control: true
            show_color_temp_control: true
            show_color_control: true
            collapsible_controls: true
          - type: custom:mushroom-light-card
            entity: light.living_room_strip
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
            show_brightness_control: true
            show_color_temp_control: true
            collapsible_controls: true
          - type: custom:mushroom-light-card
            entity: light.bedroom_nightstand
            name: Nightstand
            show_brightness_control: true
            collapsible_controls: true
      - type: custom:mushroom-title-card
        title: Kitchen
      - type: horizontal-stack
        cards:
          - type: custom:mushroom-light-card
            entity: light.kitchen
            show_brightness_control: true
            show_color_temp_control: true
            collapsible_controls: true
          - type: custom:mushroom-light-card
            entity: light.kitchen_counter
            name: Counter
            show_brightness_control: true
            collapsible_controls: true
      - type: custom:mushroom-entity-card
        entity: light.all_lights
        name: All Lights
        icon: mdi:lightbulb-group
        tap_action:
          action: toggle

  - title: Climate
    path: climate
    icon: mdi:thermostat
    cards:
      - type: custom:mushroom-title-card
        title: Air Conditioning
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
            heat: { name: Heat }
            cool: { name: Cool }
            heat_cool: { name: Auto }
            fan_only: { name: Fan }
            dry: { name: Dry }
          fan:
            auto: { name: Auto }
            low: { name: Low }
            medium: { name: Med }
            high: { name: High }
        step_size: 0.5
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
            heat: { name: Heat }
            cool: { name: Cool }
            heat_cool: { name: Auto }
            fan_only: { name: Fan }
            dry: { name: Dry }
          fan:
            auto: { name: Auto }
            low: { name: Low }
            medium: { name: Med }
            high: { name: High }
        step_size: 0.5
      - type: custom:simple-thermostat
        entity: climate.vivax_ac
        name: Vivax AC
        header:
          toggle:
            entity: climate.vivax_ac
        control:
          hvac:
            heat: { name: Heat }
            cool: { name: Cool }
            heat_cool: { name: Auto }
            fan_only: { name: Fan }
            dry: { name: Dry }
          fan:
            auto: { name: Auto }
            low: { name: Low }
            medium: { name: Med }
            high: { name: High }
        step_size: 0.5
      - type: custom:mini-graph-card
        entities:
          - entity: sensor.living_room_temperature
            name: Living Room
          - entity: sensor.bedroom_temperature
            name: Bedroom
          - entity: sensor.outdoor_temperature
            name: Outdoor
        hours_to_show: 24
        smoothing: true
        show:
          labels: true
          legend: true

  - title: Ventilation
    path: ventilation
    icon: mdi:air-filter
    cards:
      - type: custom:mushroom-title-card
        title: Komfovent HRV
        subtitle: Heat Recovery Ventilation
      - type: custom:mushroom-climate-card
        entity: climate.komfovent
        name: Komfovent
        hvac_modes:
          - auto
          - fan_only
          - "off"
        show_temperature_control: true
      - type: grid
        columns: 4
        square: false
        cards:
          - type: custom:button-card
            name: Normal
            icon: mdi:fan
            tap_action:
              action: call-service
              service: select.select_option
              service_data:
                entity_id: select.komfovent_current_mode
                option: normal
            styles:
              card:
                - border-radius: 12px
          - type: custom:button-card
            name: Boost
            icon: mdi:fan-plus
            tap_action:
              action: call-service
              service: select.select_option
              service_data:
                entity_id: select.komfovent_current_mode
                option: boost
            styles:
              card:
                - border-radius: 12px
          - type: custom:button-card
            name: Away
            icon: mdi:home-export-outline
            tap_action:
              action: call-service
              service: select.select_option
              service_data:
                entity_id: select.komfovent_current_mode
                option: away
            styles:
              card:
                - border-radius: 12px
      - type: horizontal-stack
        cards:
          - type: custom:mushroom-entity-card
            entity: sensor.komfovent_temperature_supply    # verify entity name
            name: Supply
            icon: mdi:thermometer-chevron-up
            icon_color: orange
          - type: custom:mushroom-entity-card
            entity: sensor.komfovent_temperature_extract   # verify entity name
            name: Extract
            icon: mdi:thermometer-chevron-down
            icon_color: blue
          - type: custom:mushroom-entity-card
            entity: sensor.komfovent_humidity_efficiency   # verify entity name
            name: Humidity
            icon: mdi:water-percent
            icon_color: teal
      - type: custom:mini-graph-card
        entities:
          - entity: sensor.komfovent_temperature_supply
            name: Supply air
            color: "#f6a623"
          - entity: sensor.komfovent_temperature_extract
            name: Extract air
            color: "#4a90d9"
        hours_to_show: 12
        smoothing: true
        show:
          labels: true
          legend: true

  - title: Blinds
    path: blinds
    icon: mdi:blinds
    cards:
      - type: custom:mushroom-title-card
        title: VELUX Blinds
      - type: horizontal-stack
        cards:
          - type: custom:button-card
            name: All Open
            icon: mdi:blinds-open
            tap_action:
              action: call-service
              service: cover.open_cover
              service_data:
                entity_id: group.all_velux_covers
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
            entity: automation.konfortvent_away_mode
            name: Ventilation Auto
            icon: mdi:air-filter
          - type: custom:mushroom-entity-card
            entity: automation.presence_based_lighting
            name: Presence Lights
            icon: mdi:home-account
      - type: custom:mushroom-title-card
        title: System
      - type: custom:mushroom-entity-card
        entity: sensor.processor_use_percent
        name: HA CPU Usage
        icon: mdi:cpu-64-bit
      - type: custom:mushroom-entity-card
        entity: sensor.memory_use_percent
        name: HA Memory
        icon: mdi:memory
```

### Scenes to create

| Scene | Lights | AC | Blinds |
|-------|--------|----|--------|
| `scene.morning` | All 80% warm white | Auto 21°C | Open |
| `scene.evening` | Living room 40% amber | Auto 22°C | Closed |
| `scene.movie` | Living room 10% dim | Cool 22°C | Closed |
| `scene.night` | All off | Off | Closed |
| `scene.away` | All off | Off | 50% |

### Groups to create in configuration.yaml

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

group:
  all_people:
    name: All People
    entities:
      - person.cristian_oancea
      - person.georgiana
```

---

## Part 5 — Automations

Add to `/config/automations.yaml`:

```yaml
# 1. Open blinds 30 min after sunrise
alias: blinds_open_at_sunrise
trigger:
  - platform: sun
    event: sunrise
    offset: "+00:30:00"
condition:
  - condition: state
    entity_id: input_boolean.vacation_mode
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

---

# 2. Close blinds at sunset
alias: blinds_close_at_sunset
trigger:
  - platform: sun
    event: sunset
action:
  - service: cover.close_cover
    target:
      entity_id:
        - cover.velux_living_room
        - cover.velux_bedroom
        - cover.velux_kitchen
        - cover.velux_office
mode: single

---

# 3. Close south-facing blinds when outdoor temp > 28°C
alias: blinds_close_on_heat
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
      position: 20
mode: single

---

# 4. Motion lights — living room
alias: motion_lights_living_room
trigger:
  - platform: state
    entity_id: binary_sensor.hue_motion_living_room
    to: "on"
condition:
  - condition: numeric_state
    entity_id: sensor.hue_motion_living_room_illuminance
    below: 50
action:
  - service: light.turn_on
    target:
      entity_id: light.living_room
    data:
      brightness_pct: 60
      color_temp: 3500
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

---

# 5. AC schedule — weekday morning pre-conditioning
alias: ac_schedule_weekday_morning
trigger:
  - platform: time
    at: "06:30:00"
condition:
  - condition: time
    weekday: [mon, tue, wed, thu, fri]
action:
  - service: climate.set_temperature
    target:
      entity_id: climate.lg_ac
    data:
      temperature: 22
      hvac_mode: >
        {% if states('sensor.outdoor_temperature') | float > 20 %}cool{% else %}heat{% endif %}
mode: single

---

# 6. AC off when window open 5+ minutes
alias: ac_off_when_window_open
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
mode: single

---

# 7. Komfovent boost on high humidity
alias: konfortvent_boost_on_humidity
trigger:
  - platform: numeric_state
    entity_id: sensor.komfovent_humidity_efficiency   # verify exact entity name
    above: 65
action:
  - service: select.select_option
    target:
      entity_id: select.komfovent_current_mode
    data:
      option: boost
  - wait_for_trigger:
      - platform: numeric_state
        entity_id: sensor.komfovent_humidity_efficiency
        below: 55
    timeout:
      minutes: 30
  - service: select.select_option
    target:
      entity_id: select.komfovent_current_mode
    data:
      option: normal
mode: single

---

# 8. Komfovent away when everyone leaves
alias: konfortvent_away_mode
trigger:
  - platform: state
    entity_id: group.all_people
    to: "not_home"
action:
  - service: select.select_option
    target:
      entity_id: select.komfovent_current_mode
    data:
      option: away
mode: single

---

alias: konfortvent_return_home
trigger:
  - platform: state
    entity_id: group.all_people
    to: "home"
action:
  - service: select.select_option
    target:
      entity_id: select.komfovent_current_mode
    data:
      option: normal
mode: single

---

# 9. Welcome home after dark
alias: presence_welcome_home
trigger:
  - platform: state
    entity_id: person.your_name
    to: "home"
condition:
  - condition: sun
    after: sunset
action:
  - service: scene.turn_on
    target:
      entity_id: scene.evening
mode: single

---

# 10. Good night routine
alias: good_night_routine
trigger:
  - platform: time
    at: "23:00:00"
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
        {% if states('sensor.outdoor_temperature') | float > 18 %}cool{% else %}off{% endif %}
mode: single
```

---

## Part 6 — Implementation Order

### Phase 1 — Infrastructure
1. Create `/share/homeassistant/config` on QNAP
2. Create `docker-compose.yml` and deploy via Container Station
3. Complete HA onboarding at `http://<QNAP-IP>:8123`
4. Configure HTTPS reverse proxy + update `configuration.yaml`

### Phase 2 — HACS & Cards
5. Install HACS + GitHub OAuth
6. Install HACS integrations (govee, velux, thinq2-ha, localtuya, smartir)
7. Install HACS frontend cards (Mushroom, Button Card, Mini Graph, Bubble Card, Simple Thermostat, Swipe Nav, Mushroom Themes)

### Phase 3 — Local Integrations
8. Philips Hue — add integration, press bridge button
9. IKEA DIRIGERA — add integration, press hub button
10. Govee LAN — enable LAN in Govee app, add integration

### Phase 4 — Cloud Integrations
11. Samsung SmartThings — create PAT, add integration
12. LG ThinQ — add integration, log in
13. VELUX — add integration, log in
14. Vivax AC — try Path A (tinytuya scan → localtuya), fall back to Path B (SmartIR)
15. Google Home — Google Cloud project + Actions on Google + configure HA + link in app

### Phase 5 — Dashboard
16. Create helper entities (vacation_mode toggle, light groups, cover group, person group)
17. Create scenes (Morning, Evening, Movie, Night, Away)
18. Apply dashboard YAML, replace all placeholder entity IDs

### Phase 6 — Automations
19. Paste automations into `automations.yaml`, replace entity IDs
20. Test each automation manually

### Phase 7 — Polish
21. Verify all integrations green in Settings → Devices & Services
22. Test Google Home voice commands
23. Verify dashboard on mobile + swipe navigation
24. (Optional) Add System Monitor integration for CPU/memory sensors

### Timeline estimate

| Phase | Time |
|-------|------|
| Infrastructure | 30–60 min |
| HACS & Cards | 20–30 min |
| Local integrations | 20–30 min |
| Cloud integrations | 60–120 min (Google Home is complex) |
| Dashboard | 30–60 min |
| Automations | 20–30 min |
| Polish | 20–30 min |
| **Total** | **3–6 hours** |
