# Home Assistant Setup — Integrations

All integrations are set up from within Home Assistant unless noted otherwise.
Navigate to **Settings → Devices & Services → Add Integration** for UI-based setups.

---

## 1. Philips Hue (Local Bridge)

**Type:** Built-in HA integration, local LAN, no cloud required.

### Steps

1. Make sure the Hue Bridge is on the same LAN as the QNAP.
2. In HA: **Settings → Devices & Services → Add Integration** → search `Philips Hue`
3. HA will auto-discover the bridge. Click on it.
4. **Press the physical button** on top of the Hue Bridge within 30 seconds when prompted.
5. HA will import all lights, rooms, scenes, and sensors automatically.

### Result

- All Hue lights appear as `light.*` entities
- Hue motion sensors appear as `binary_sensor.*` and `sensor.*` (motion, temperature, lux)
- Hue scenes appear and can be activated via `scene.*` entities
- Hue switches/dimmers appear as `event.*` entities

### Notes

- Local polling interval is fast (~1 second for state changes via event stream)
- No internet required after initial setup
- Hue rooms/zones are imported as HA areas automatically

---

## 2. Govee (Local LAN)

**Type:** HACS custom integration — `govee_lights_local`
**Requires:** HACS installed (see `03-hacs-and-custom-cards.md`)

### Enable LAN control on Govee devices

For each Govee device that supports LAN control:
1. Open the **Govee Home** app
2. Go to the device settings → **LAN Control** → Enable
3. Note: Not all Govee devices support LAN. Check the [Govee LAN compatibility list](https://app-h5.govee.com/user-manual/wlan-guide)

### Install the integration

1. In HACS: **Integrations** → search `Govee LAN` → Install → Restart HA
2. In HA: **Settings → Devices & Services → Add Integration** → `Govee LAN`
3. The integration will scan your LAN (UDP broadcast on port 4001/4002) and discover Govee devices automatically.

### Result

- Govee lights appear as `light.*` entities with brightness, color temp, and RGB support
- State updates via LAN push (~1 second latency)

### Fallback — Govee Cloud API

For devices that do not support LAN control:
1. Install HACS integration: `Govee` (cloud version)
2. Get your Govee API key: Govee app → Profile → **About Us** → **Request API Key**
3. Add integration in HA with the API key

---

## 3. IKEA DIRIGERA (Local)

**Type:** Built-in HA integration, local LAN.

### Prerequisites

- IKEA DIRIGERA hub connected to your LAN via Ethernet
- IKEA Home Smart app set up with all devices paired to the hub

### Steps

1. In HA: **Settings → Devices & Services → Add Integration** → search `IKEA`
2. Select **IKEA DIRIGERA**
3. HA will auto-discover the hub. Click on it.
4. **Press the action button** on the DIRIGERA hub once when prompted.
5. All devices paired to the hub are imported.

### Result

- IKEA lights → `light.*`
- IKEA smart plugs → `switch.*`
- IKEA motion sensors → `binary_sensor.*`
- IKEA remotes → `event.*`
- IKEA air purifiers → `fan.*`

### Notes

- Uses the local DIRIGERA REST API (no cloud, no IKEA account needed after pairing)
- Scenes created in the IKEA app are imported as `scene.*`

---

## 4. VELUX (Cloud)

**Type:** HACS custom integration
**Requires:** HACS installed, VELUX Active account, VELUX KIX 300 or compatible gateway

### Install

1. In HACS: **Integrations** → search `VELUX` → Install → Restart HA
2. In HA: **Settings → Devices & Services → Add Integration** → `VELUX`
3. Log in with your VELUX Active / VELUX account credentials

### Result

- VELUX blinds/shutters → `cover.*` entities (open, close, stop, set position)
- VELUX windows → `cover.*`
- Velux sensors (if any) → `sensor.*`

### Notes

- Cloud-dependent: requires internet and VELUX account
- State polling interval ~30 seconds (cloud latency)
- Position control (0–100%) is supported

---

## 5. Samsung AC (SmartThings)

**Type:** Built-in HA integration, SmartThings cloud.

### Prerequisites

- Samsung account with SmartThings set up
- Samsung AC added to the SmartThings app and working

### Create a SmartThings Personal Access Token

1. Go to [https://account.smartthings.com/tokens](https://account.smartthings.com/tokens)
2. Sign in with your Samsung account
3. Click **Generate new token**
4. Name it `HomeAssistant`
5. Select all scopes (or at minimum: Devices — read, execute)
6. Copy the generated token (shown only once)

### Steps in HA

1. In HA: **Settings → Devices & Services → Add Integration** → search `SmartThings`
2. Enter the Personal Access Token
3. HA imports all SmartThings devices including the Samsung AC

### Result

- Samsung AC → `climate.*` entity with:
  - HVAC modes: heat, cool, heat_cool, fan_only, dry, off
  - Target temperature
  - Fan speed
  - Swing mode

---

## 6. LG AC (ThinQ)

**Type:** HACS custom integration — `thinq2-ha`
**Requires:** HACS installed, LG ThinQ account

### Install

1. In HACS: **Integrations** → search `LG ThinQ` → Install → Restart HA
2. In HA: **Settings → Devices & Services → Add Integration** → `LG ThinQ`
3. Log in with your LG ThinQ account credentials
4. Select your country/region

### Result

- LG AC → `climate.*` entity with full mode/temp/fan control
- Some LG AC models also expose energy monitoring sensors

### Notes

- Cloud-based (LG ThinQ API)
- If login fails, try setting region to the country where your LG account was created

---

## 7. Vivax AC

**Type:** Depends on hardware — follow Path A first, fall back to Path B.

### Path A — Tuya-based (try this first)

Many Vivax ACs use Tuya/SmartLife internals. To check:

1. Install Python on a computer on the same LAN:
   ```bash
   pip install tinytuya
   python -m tinytuya scan
   ```
2. If Vivax devices appear in the scan output, they are Tuya-based. Proceed with `localtuya`.

#### Install localtuya (HACS)

1. In HACS: **Integrations** → search `localtuya` → Install → Restart HA
2. In HA: **Settings → Devices & Services → Add Integration** → `LocalTuya`
3. You need the **Device ID** and **Local Key** for each device:
   - Log in at [https://iot.tuya.com](https://iot.tuya.com)
   - Go to **Cloud → Development → Your Project → Devices**
   - Find the device, copy the Device ID and Local Key
4. Add the device in LocalTuya, map the data points (DPs):
   - DP 1: on/off (switch)
   - DP 2: target temperature (integer)
   - DP 3: current temperature (integer)
   - DP 4: mode (string: cold/hot/wind/wet/auto)
   - DP 5: fan speed (string: auto/low/middle/high)

#### Result

- Vivax AC → `climate.*` entity, fully local, no cloud

---

### Path B — SmartIR + IR Blaster (fallback if not Tuya)

Use this if the Vivax AC is not Tuya-based or if local Tuya control is unreliable.

#### Hardware required

- **Broadlink RM4 Pro** (or RM4 Mini) IR blaster — place in line of sight of the AC unit

#### Setup

1. Add Broadlink to HA: **Settings → Devices & Services → Add Integration** → `Broadlink`
   - HA will auto-discover it on the LAN
2. In HACS: **Integrations** → search `SmartIR` → Install → Restart HA
3. Find the Vivax AC IR code file:
   - Browse [https://github.com/smartHomeHub/SmartIR/tree/master/codes/climate](https://github.com/smartHomeHub/SmartIR/tree/master/codes/climate)
   - Look for Vivax or a compatible code file (many use generic Midea/GMCC codes)
   - Download the JSON file to `/config/codes/climate/`
4. Add to `configuration.yaml`:

```yaml
smartir:

climate:
  - platform: smartir
    name: Vivax AC Living Room
    unique_id: vivax_ac_living_room
    device_code: 1180           # replace with correct code number for Vivax
    controller_data: remote.broadlink_rm4_pro   # your Broadlink entity ID
    temperature_sensor: sensor.living_room_temperature   # optional
    humidity_sensor: sensor.living_room_humidity         # optional
    power_sensor: binary_sensor.ac_power                 # optional
```

5. Restart HA.

#### Result

- Vivax AC → `climate.*` entity controllable via IR

---

## 8. Google Home (Bidirectional — Manual OAuth)

**Type:** Built-in HA `google_assistant` manual configuration.
**Requires:** Public HTTPS endpoint (see `01-docker-setup.md` section 5).

This is the most complex integration. It enables:
- Controlling HA devices via Google Assistant / Google Home app
- HA can trigger Google Home routines (via `google_assistant` service calls)

### Step 1 — Create a Google Cloud project

1. Go to [https://console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project: `HomeAssistant`
3. Enable the **HomeGraph API**:
   - APIs & Services → Library → search `HomeGraph API` → Enable
4. Create a Service Account:
   - APIs & Services → Credentials → Create Credentials → Service Account
   - Name: `homeassistant`
   - Download the JSON key file → save to `/config/google_credentials.json`

### Step 2 — Create an Actions on Google project

1. Go to [https://console.actions.google.com](https://console.actions.google.com)
2. New project → select the Google Cloud project created above
3. **Smart Home** type
4. Set Fulfillment URL: `https://yourdomain.com/api/google_assistant`
5. Account Linking:
   - OAuth Client ID: any string (e.g. `homeassistant`)
   - OAuth Client Secret: a long random string (save both — needed in HA config)
   - Authorization URL: `https://yourdomain.com/auth/authorize`
   - Token URL: `https://yourdomain.com/auth/token`

### Step 3 — Configure HA

Add to `/config/configuration.yaml`:

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
  entity_config:
    light.living_room:
      name: Living Room Light
      expose: true
    climate.samsung_ac:
      name: Samsung AC
      expose: true
    cover.velux_bedroom:
      name: Bedroom Blinds
      expose: true
```

### Step 4 — Link in Google Home app

1. Open the Google Home app on your phone
2. **Add device** → **Works with Google** → search your Actions project name
3. Sign in with your HA credentials
4. Devices from HA appear in Google Home

### Notes

- `report_state: true` enables real-time state sync from HA → Google Home
- Any HA device not in `exposed_domains` or explicitly set to `expose: false` will not appear in Google Home
- Voice commands like "Hey Google, turn on the living room lights" will work after linking

---

## 9. Konfortvent (Already Integrated)

No integration steps needed — it is already working in HA.

**Focus:** Dashboard design only. See `04-dashboard.md` for the Konfortvent card layout.

The integration exposes entities such as:
- `fan.konfortvent` — fan speed control
- `sensor.konfortvent_temperature_supply` — supply air temperature
- `sensor.konfortvent_temperature_extract` — extract air temperature
- `sensor.konfortvent_humidity` — humidity
- `select.konfortvent_mode` — ventilation mode (normal, boost, eco, away)

> Adjust entity IDs above to match your actual Konfortvent entity IDs in HA.
