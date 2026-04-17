# Home Assistant Setup — HACS & Custom Cards

## 1. Install HACS (Home Assistant Community Store)

HACS is required for several integrations and all custom dashboard cards.

### Steps

1. SSH into the QNAP or use the HA terminal add-on (if available):

```bash
wget -O - https://get.hacs.xyz | bash -
```

Or manually:

1. Download the latest HACS release from [https://github.com/hacs/integration/releases](https://github.com/hacs/integration/releases)
2. Extract and copy the `hacs` folder to `/config/custom_components/hacs/`
3. Restart Home Assistant

### Activate HACS in HA

1. In HA: **Settings → Devices & Services → Add Integration** → search `HACS`
2. Follow the GitHub OAuth flow to link your GitHub account (free)
3. HACS appears in the left sidebar

---

## 2. Required HACS Integrations

Install each via **HACS → Integrations → Explore & download repositories**:

| Integration | Repository | Purpose |
|-------------|-----------|---------|
| Govee LAN | `govee_lights_local` | Local LAN control for Govee lights |
| VELUX | `velux` | VELUX blinds cloud integration |
| LG ThinQ | `thinq2-ha` | LG AC and appliances |
| LocalTuya | `localtuya` | Vivax AC (if Tuya-based) |
| SmartIR | `smartir` | Vivax AC (IR blaster fallback) |

After installing each, restart HA before configuring.

---

## 3. Required HACS Frontend Cards

Install each via **HACS → Frontend → Explore & download repositories**:

### 3.1 Mushroom Cards

**Repository:** `piitaya/lovelace-mushroom`

The primary card set for the dashboard. Clean, minimal, and highly customizable.

Cards used:
- `mushroom-title-card` — section headers
- `mushroom-light-card` — individual light control
- `mushroom-climate-card` — AC/climate entities
- `mushroom-cover-card` — VELUX blinds
- `mushroom-fan-card` — Konfortvent fan
- `mushroom-entity-card` — generic sensors/switches
- `mushroom-chips-card` — compact status row at top of dashboard
- `mushroom-template-card` — custom display with Jinja2 templates

### 3.2 Button Card

**Repository:** `custom-cards/button-card`

Highly customizable button cards. Used for scene tiles, AC quick-action buttons, and room navigation cards.

### 3.3 Mini Graph Card

**Repository:** `kalkih/mini-graph-card`

Compact sparkline graphs for temperature, humidity, and energy sensors.

Used for:
- Room temperature history
- Konfortvent temperature supply/extract history
- AC energy monitoring (if available)

### 3.4 Bubble Card

**Repository:** `Clooos/Bubble-Card`

Modern pop-up style cards with slide-up sub-menus. Used for:
- AC detailed control pop-ups (full mode/fan/temp control)
- Konfortvent detailed pop-up
- Light group pop-ups (all lights in a room)

### 3.5 Simple Thermostat Card

**Repository:** `nervetattoo/simple-thermostat`

Better climate card than the default HA thermostat. Used for all AC units.

Features:
- Large temperature display
- Mode selector buttons (cool, heat, auto, fan, dry)
- Fan speed selector
- Current vs target temperature

### 3.6 Lovelace Swipe Navigation

**Repository:** `maykar/lovelace-swipe-navigation`

Enables swipe gestures between dashboard views on mobile. Strongly recommended for phone use.

---

## 4. Themes

### 4.1 Install a theme via HACS

Recommended: **Mushroom Themes** (pairs with Mushroom cards)

1. In HACS: **Frontend** → search `Mushroom Themes` → Install
2. In HA: **Settings → Dashboard → Themes** → select `Mushroom` or a variant
3. For automatic dark/light mode:

Add to `configuration.yaml`:
```yaml
frontend:
  themes: !include_dir_merge_named themes/
```

Then in **Profile → Theme** select the desired theme and enable **Auto**.

### 4.2 Recommended themes

| Theme | Style | Notes |
|-------|-------|-------|
| Mushroom (built-in with Mushroom cards) | Clean minimal | Best match for Mushroom cards |
| iOS Dark Mode | Dark, iOS-like | Popular choice |
| Google Home Dark | Dark, Material | Good if you use Google Home a lot |

---

## 5. After all installs

Restart HA once more after installing all frontend resources, then clear the browser cache:

```
Ctrl + Shift + R  (hard refresh)
```

Verify all resources are loaded:
**Settings → Dashboards → Resources** — all cards should appear in the list.
