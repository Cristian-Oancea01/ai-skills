---
name: homeassistant-setup
description: Home Assistant on QNAP NAS — Docker, integrations, Lovelace dashboard, automations
license: MIT
compatibility: opencode
---

## Access

- SSH: `plink -ssh MCP@<nas-ip> -pw "<password>" -batch "<cmd>"`
- Docker binary: `/share/CACHEDEV1_DATA/.qpkg/container-station/bin/docker`
- HA container: `home-assistant`
- HA config via SMB: `Z:\HomeAssistantConfig\` = `/share/Public/HomeAssistantConfig/`
- Credentials: `C:\Users\oance\.config\opencode\local\nas_access.md` — **never commit**

## Lessons learned (keep updated)

### Lovelace yaml mode
- Set `lovelace: mode: yaml` in `configuration.yaml` — do **not** add `resources:` there
- Resources must be declared in `ui-lovelace.yaml` under a top-level `resources:` key
- `.storage/lovelace_resources` is ignored in yaml mode
- `.storage/lovelace_dashboards` must have `"mode": "yaml"` — if it says `"storage"` the old UI dashboard loads instead
- `.storage/lovelace.lovelace` holds the old storage-mode dashboard — safe to ignore
- **Lovelace yaml changes require a container restart** — file watcher does not detect SMB changes

### Container restart
```powershell
plink -ssh MCP@<nas-ip> -pw "<password>" -batch `
  "/share/CACHEDEV1_DATA/.qpkg/container-station/bin/docker restart home-assistant"
# Wait ~45s, then check logs:
plink -ssh MCP@<nas-ip> -pw "<password>" -batch `
  "/share/CACHEDEV1_DATA/.qpkg/container-station/bin/docker logs --tail 10 home-assistant 2>&1"
```

### Custom cards not loading ("configuration error" on every card)
Two causes found:
1. `resources: []` in `configuration.yaml` blanks out all card JS — remove it
2. `lovelace_dashboards` mode set to `storage` — patch with `sed -i s/storage/yaml/ /share/Public/HomeAssistantConfig/.storage/lovelace_dashboards` (stop container first)

### Mushroom cards
- HACS installs `piitaya/lovelace-mushroom` — single file `mushroom.js` (~712KB for v5.1.1)
- Card types: `custom:mushroom-light-card`, `custom:mushroom-entity-card`, `custom:mushroom-title-card`, `custom:mushroom-template-card`, `custom:mushroom-climate-card`, `custom:mushroom-cover-card`, `custom:mushroom-chips-card`
- Card names are NOT visible as literal strings in the minified JS — they are assembled from variables. This is normal.

### ui-lovelace.yaml structure
```yaml
title: Home
resources:
  - url: /hacsfiles/lovelace-mushroom/mushroom.js
    type: module
  - url: /hacsfiles/button-card/button-card.js
    type: module
  - url: /hacsfiles/mini-graph-card/mini-graph-card-bundle.js
    type: module
views:
  - !include lovelace/views/01_home.yaml
  - !include lovelace/views/02_lighting.yaml
  - !include lovelace/views/03_climate.yaml
  - !include lovelace/views/04_ventilation.yaml
  - !include lovelace/views/05_blinds.yaml
  - !include lovelace/views/06_settings.yaml
```

### Komfovent pymodbus fix
- pymodbus 3.11+ renamed `slave=` → `device_id=`
- Fixed in `/config/custom_components/komfovent/modbus.py` — all 4 call sites use `device_id=self._unit`

### Real Komfovent entity IDs
- `climate.komfovent`
- `select.komfovent_current_mode` (options: normal / boost / away)
- `select.komfovent_scheduler_mode`, `select.komfovent_temperature_control`, `select.komfovent_flow_control`, `select.komfovent_eco_heat_recovery`
- `sensor.komfovent_supply_temperature`, `sensor.komfovent_extract_temperature`, `sensor.komfovent_outdoor_temperature`
- **No humidity sensor installed** — do not add humidity cards or automations

### Real Hue entity IDs
`light.living_jos`, `light.living_sus`, `light.spot_living_1..4`, `light.bec_living`, `light.bec_insula_jos`, `light.bec_insula_sus`, `light.dormitor_sus`, `light.dormitor_sus_1`, `light.dormitor_sus_2`, `light.dormitor_victor`, `light.birou_cristi`, `light.birou_cristi_2`, `light.birou_georgi`, `light.hol_sus`, `light.hol_sus_2`, `light.baie_sus`, `light.bec_scara`, `light.spot_scara_1`, `light.spot_scara_2`, `light.bec_masa_jos`, `light.bec_masa_sus`, `light.hue_aurelle_panel_4`, `light.hue_aurelle_panel_5`, `light.hue_flourish_pendant_1`, `light.hue_white_lamp_2`, `light.hue_white_lamp_3`, `light.dimmable_light_1`, `light.dimmable_light_1_2`

### Outdoor temperature
Use `sensor.komfovent_outdoor_temperature` — no separate weather integration needed.

### Automations
- Files in `automations/` loaded via `!include_dir_merge_list` (files are lists, not single dicts)
- Real automation IDs: `blinds_open_at_sunrise`, `blinds_close_at_sunset`, `blinds_close_on_heat`, `ac_schedule_weekday_morning`, `ac_off_when_window_open`, `good_night_routine`, `konfortvent_away_mode`, `konfortvent_return_home`, `la_apus_aprinde_lumina_la_insula`

### Known broken integrations (deferred)
- **Miele**: `flatdict==4.0.1` fails on Python 3.14 (`pkg_resources` missing) — wait for upstream fix or remove
- **govee_light_ble**: archived repo — remove via HACS UI
- **AC units** (Samsung, LG, Vivax): not yet integrated — climate view shows placeholder template cards

### Verifying real entity IDs
```bash
# List all entities of a domain from registry
plink ... "docker exec home-assistant grep -o 'sensor.komfovent[a-z_]*' /config/.storage/core.entity_registry | sort -u"
```

## File structure
```
Z:\HomeAssistantConfig\
├── configuration.yaml          # lovelace: mode: yaml (no resources: key)
├── ui-lovelace.yaml            # resources + !include views
├── automations\
│   ├── existing.yaml           # UI-created automations
│   ├── blinds.yaml
│   ├── climate.yaml
│   ├── lighting.yaml
│   └── ventilation.yaml
├── integrations\
│   ├── groups.yaml             # person group
│   └── cover_groups.yaml      # VELUX placeholder cover group
├── custom_components\
│   └── komfovent\
│       └── modbus.py           # FIXED: device_id= for pymodbus 3.11+
└── lovelace\views\
    ├── 01_home.yaml            # uses real Komfovent + Hue entities
    ├── 02_lighting.yaml        # all real Hue entity IDs
    ├── 03_climate.yaml         # placeholder template cards (AC not integrated)
    ├── 04_ventilation.yaml     # real Komfovent entities, no humidity card
    ├── 05_blinds.yaml          # placeholder VELUX covers
    └── 06_settings.yaml        # real automation entity IDs only
```

## Next steps
1. Remove `govee_light_ble` — HACS UI → remove archived repo
2. Generate new Long-Lived Access Token in HA (old one expired)
3. Govee: install `govee_lights_local` HACS, enable LAN in Govee app
4. Samsung AC: SmartThings PAT → Settings → Add Integration → SmartThings
5. LG AC: HACS `smartthinq_sensors` → Settings → Add Integration
6. Vivax: tinytuya scan → if found use LocalTuya, else SmartIR
7. VELUX: requires KLF200 hardware interface
8. Miele: wait for flatdict fix or pin older HA image
