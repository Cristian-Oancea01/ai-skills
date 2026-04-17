# Home Assistant Setup — Overview

## Architecture

```
Internet
   │
   ▼
QNAP NAS (Container Station)
   │
   └── Docker: homeassistant (host network mode)
            │
            ├── Philips Hue Bridge  (local LAN)
            ├── Govee devices       (local LAN)
            ├── IKEA DIRIGERA       (local LAN)
            ├── VELUX               (cloud)
            ├── Samsung SmartThings (cloud)
            ├── LG ThinQ            (cloud)
            ├── Vivax AC            (Tuya local / SmartIR fallback)
            ├── Konfortvent         (already integrated)
            └── Google Home         (manual OAuth — bidirectional)
```

## Goals

- Single dashboard to control all smart devices
- Good-looking Lovelace UI with Mushroom + custom cards
- Automations for comfort and energy efficiency
- Google Home voice control + HA controls Google Home devices
- No reliance on Nabu Casa — self-hosted with manual HTTPS

## Device Inventory

### Lighting
| Device | Brand | Integration Method |
|--------|-------|--------------------|
| Hue bulbs, switches, sensors | Philips Hue | Local Hue Bridge (built-in HA integration) |
| LED strips, bulbs | Govee | Local LAN (HACS: `govee_lights_local`) |
| TRADFRI bulbs, outlets | IKEA | Local DIRIGERA hub (built-in HA integration) |

### Climate — Air Conditioning
| Device | Brand | Integration Method |
|--------|-------|--------------------|
| Vivax AC | Vivax | Tuya local (if Tuya-based) or SmartIR fallback |
| Samsung AC | Samsung | SmartThings cloud (built-in HA integration) |
| LG AC | LG | LG ThinQ (HACS: `thinq2-ha`) |

### Ventilation
| Device | Brand | Integration Method |
|--------|-------|--------------------|
| Konfortvent HRV unit | Konfortvent | Already integrated and working |

### Blinds / Shades
| Device | Brand | Integration Method |
|--------|-------|--------------------|
| VELUX blinds | VELUX | VELUX cloud (HACS: `velux`) |

### Voice / Ecosystem
| Device | Brand | Integration Method |
|--------|-------|--------------------|
| Google Nest / Home speakers | Google | Manual Google Assistant OAuth (bidirectional) |

## Key Technical Decisions

1. **Host network mode** — HA Docker container must use `network_mode: host` so it can discover local devices (Hue bridge mDNS, Govee LAN broadcast, DIRIGERA mDNS). This is critical on QNAP.
2. **HACS** — Home Assistant Community Store is required for Govee LAN, VELUX, LG ThinQ, and all custom dashboard cards.
3. **Google Home manual** — Uses `google_assistant:` config in `configuration.yaml` with a manually created Google Cloud Console project. Requires HTTPS with a valid certificate accessible from the internet.
4. **QNAP reverse proxy** — QNAP's built-in Application Portal (or nginx) is used to expose HA on HTTPS for Google Home webhook.
5. **Vivax path** — First check if the AC is Tuya-based (check with `tinytuya` scan); if yes use `localtuya` HACS integration; if no, use a Broadlink RM4 Pro IR blaster with `smartir`.

## Folder Structure (this repo)

```
homeassistant-setup/
├── 00-overview.md               ← this file
├── 01-docker-setup.md           ← QNAP Container Station + docker-compose
├── 02-integrations.md           ← all integration setup steps
├── 03-hacs-and-custom-cards.md  ← HACS install + card list
├── 04-dashboard.md              ← Lovelace dashboard YAML
├── 05-automations.md            ← automation YAML examples
└── 06-implementation-order.md   ← ordered checklist for agent execution
```
