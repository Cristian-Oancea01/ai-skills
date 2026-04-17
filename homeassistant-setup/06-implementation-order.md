# Home Assistant Setup — Implementation Order

This is the ordered checklist for an agent (or human) to follow when setting up the system from scratch.
Each step has a clear prerequisite and a validation test. Do not skip ahead — steps build on each other.

---

## Phase 1 — Infrastructure

### Step 1: Prepare QNAP storage

- [ ] Create directory `/share/homeassistant/config` on the QNAP NAS
- [ ] Create `/share/homeassistant/docker-compose.yml` using the content in `01-docker-setup.md`
- **Validation:** `ls /share/homeassistant/` shows `docker-compose.yml` and `config/`

### Step 2: Deploy Home Assistant container

- [ ] In Container Station: create application from `docker-compose.yml` (see `01-docker-setup.md` section 3)
- [ ] Wait for container to start (~60 seconds)
- [ ] Navigate to `http://<QNAP-IP>:8123` in browser
- **Validation:** HA onboarding wizard appears

### Step 3: Complete HA onboarding

- [ ] Create admin user account
- [ ] Set home location (used for sunrise/sunset automations)
- [ ] Set timezone (must match `TZ` in docker-compose.yml)
- **Validation:** HA main dashboard loads after onboarding

### Step 4: Configure reverse proxy for HTTPS

- [ ] Set up QNAP Application Portal reverse proxy rule OR nginx (see `01-docker-setup.md` section 5)
- [ ] Obtain and install SSL certificate (Let's Encrypt recommended)
- [ ] Add `external_url`, `internal_url`, and `http:` config to `configuration.yaml` (see `01-docker-setup.md` section 6)
- [ ] Restart HA
- **Validation:** `https://yourdomain.com` opens HA login page without certificate warnings

---

## Phase 2 — HACS & Custom Cards

### Step 5: Install HACS

- [ ] Install HACS custom component (see `03-hacs-and-custom-cards.md` section 1)
- [ ] Restart HA
- [ ] Complete GitHub OAuth link in HA UI
- **Validation:** HACS appears in HA left sidebar

### Step 6: Install HACS integrations

Install in this order (each requires a restart after the last one):

- [ ] `govee_lights_local`
- [ ] `velux`
- [ ] `thinq2-ha` (LG ThinQ)
- [ ] `localtuya` (for Vivax — Path A)
- [ ] `smartir` (for Vivax — Path B fallback, only if needed)
- [ ] Restart HA once after all are installed
- **Validation:** All integrations appear under Settings → Devices & Services → Add Integration search

### Step 7: Install HACS frontend cards

- [ ] Mushroom Cards (`piitaya/lovelace-mushroom`)
- [ ] Button Card (`custom-cards/button-card`)
- [ ] Mini Graph Card (`kalkih/mini-graph-card`)
- [ ] Bubble Card (`Clooos/Bubble-Card`)
- [ ] Simple Thermostat (`nervetattoo/simple-thermostat`)
- [ ] Lovelace Swipe Navigation (`maykar/lovelace-swipe-navigation`)
- [ ] Install a theme: Mushroom Themes
- [ ] Restart HA + hard-refresh browser
- **Validation:** Settings → Dashboards → Resources shows all installed cards

---

## Phase 3 — Integrations (Local)

### Step 8: Philips Hue

- [ ] Confirm Hue Bridge is on the same LAN as QNAP
- [ ] Add integration: Settings → Devices & Services → Add Integration → Philips Hue
- [ ] Press button on Hue Bridge when prompted
- **Validation:** All Hue lights, sensors, and scenes appear as entities in HA

### Step 9: IKEA DIRIGERA

- [ ] Confirm DIRIGERA hub is on the same LAN as QNAP, connected via Ethernet
- [ ] Add integration: Settings → Devices & Services → Add Integration → IKEA
- [ ] Press action button on DIRIGERA hub when prompted
- **Validation:** IKEA lights, plugs, and sensors appear as entities

### Step 10: Govee LAN

- [ ] Enable LAN Control in Govee Home app for each compatible device
- [ ] Add integration: Settings → Devices & Services → Add Integration → Govee LAN
- **Validation:** Govee lights appear as `light.*` entities; test toggle from HA

---

## Phase 4 — Integrations (Cloud)

### Step 11: Samsung SmartThings (AC)

- [ ] Create SmartThings Personal Access Token (see `02-integrations.md` section 5)
- [ ] Add integration: Settings → Devices & Services → Add Integration → SmartThings
- [ ] Enter token
- **Validation:** Samsung AC appears as `climate.*` entity; test mode change from HA

### Step 12: LG ThinQ (AC)

- [ ] Add integration: Settings → Devices & Services → Add Integration → LG ThinQ
- [ ] Log in with LG ThinQ account
- **Validation:** LG AC appears as `climate.*` entity; test temperature set from HA

### Step 13: VELUX

- [ ] Add integration: Settings → Devices & Services → Add Integration → VELUX
- [ ] Log in with VELUX account
- **Validation:** VELUX blinds appear as `cover.*` entities; test open/close from HA

### Step 14: Vivax AC — Path A (Tuya check)

- [ ] Run `python -m tinytuya scan` on a computer on the same LAN
- [ ] If Vivax devices appear: proceed with LocalTuya setup (see `02-integrations.md` section 7, Path A)
  - [ ] Get Device ID and Local Key from iot.tuya.com
  - [ ] Add integration: LocalTuya
  - [ ] Map data points for climate control
  - **Validation:** Vivax AC appears as `climate.*`; test from HA
- [ ] If Vivax NOT found by tinytuya: proceed to Path B

### Step 15: Vivax AC — Path B (SmartIR fallback, only if Step 14 failed)

- [ ] Install Broadlink RM4 Pro and add to HA (auto-discovered)
- [ ] Download Vivax IR code file to `/config/codes/climate/`
- [ ] Add SmartIR climate config to `configuration.yaml`
- [ ] Restart HA
- **Validation:** Vivax AC appears as `climate.*`; test mode/temp commands

### Step 16: Google Home (Bidirectional)

- [ ] Create Google Cloud project and enable HomeGraph API (see `02-integrations.md` section 8)
- [ ] Create Service Account and download JSON key to `/config/google_credentials.json`
- [ ] Create Actions on Google project (Smart Home type)
- [ ] Set Fulfillment URL to `https://yourdomain.com/api/google_assistant`
- [ ] Configure Account Linking (OAuth client ID/secret, auth/token URLs)
- [ ] Add `google_assistant:` config to `configuration.yaml`
- [ ] Restart HA
- [ ] Link in Google Home app (Add device → Works with Google)
- **Validation:** HA devices appear in Google Home; test "Hey Google, turn on the living room lights"

---

## Phase 5 — Dashboard

### Step 17: Create helper entities

- [ ] Create `input_boolean.vacation_mode` helper (Settings → Helpers → Toggle)
- [ ] Create light groups (Settings → Helpers → Group → Light group):
  - `light.all_lights`
  - `light.living_room`
  - `light.bedroom`
  - `light.kitchen`
- [ ] Add cover group to `configuration.yaml` (see `04-dashboard.md` bottom section)
- [ ] Add `group.all_people` to `configuration.yaml` with all `person.*` entities
- [ ] Restart HA

### Step 18: Create scenes

- [ ] Create scenes in HA: Morning, Evening, Movie, Night, Away
  (Settings → Automations & Scenes → Scenes → Add Scene)
- [ ] Test each scene from the HA UI

### Step 19: Apply dashboard YAML

- [ ] Create new dashboard: Settings → Dashboards → Add Dashboard
- [ ] Open → Edit → Raw configuration editor
- [ ] Paste full YAML from `04-dashboard.md`
- [ ] Replace all placeholder entity IDs with real entity IDs
- [ ] Save and verify all 6 views render correctly
- **Validation:** All cards show device states; controls work (toggle lights, set AC temp, etc.)

---

## Phase 6 — Automations

### Step 20: Add automations

- [ ] Open `/config/automations.yaml`
- [ ] Paste all automation YAML from `05-automations.md`
- [ ] Replace all placeholder entity IDs with real entity IDs
- [ ] Reload automations: Developer Tools → YAML → Reload Automations
- **Validation:** Each automation appears in Settings → Automations & Scenes → Automations

### Step 21: Test each automation

- [ ] Manually trigger **blinds_open_at_sunrise** from the HA UI — verify blinds open
- [ ] Manually trigger **blinds_close_at_sunset** — verify blinds close
- [ ] Walk in front of motion sensor — verify **motion_lights_living_room** activates
- [ ] Simulate high humidity (temporarily lower threshold) — verify **konfortvent_boost_on_humidity**
- [ ] Leave home (or set person state to not_home in Developer Tools) — verify **konfortvent_away_mode**
- [ ] Manually run **good_night_routine** — verify lights off, blinds close, AC adjusts

---

## Phase 7 — Polish & Validation

### Step 22: Final checks

- [ ] All integrations show green (connected) in Settings → Devices & Services
- [ ] Google Home voice commands work for lights, AC, and blinds
- [ ] Dashboard looks correct on desktop and mobile
- [ ] Swipe navigation works on mobile between views
- [ ] Scenes all work correctly
- [ ] Automations toggle correctly from the Settings view on the dashboard
- [ ] Konfortvent sensors show real-time data and mode selector works

### Step 23: Optional — System Monitor

- [ ] Add HA System Monitor integration (built-in): Settings → Add Integration → System Monitor
- [ ] Enables CPU, memory, disk sensors for the Settings dashboard view

---

## Summary timeline estimate

| Phase | Estimated time |
|-------|---------------|
| Phase 1 — Infrastructure | 30–60 min |
| Phase 2 — HACS & Cards | 20–30 min |
| Phase 3 — Local integrations | 20–30 min |
| Phase 4 — Cloud integrations | 60–120 min (Google Home is complex) |
| Phase 5 — Dashboard | 30–60 min |
| Phase 6 — Automations | 20–30 min |
| Phase 7 — Polish | 20–30 min |
| **Total** | **3–6 hours** |

The majority of time is spent on Google Home manual OAuth setup and Vivax AC path determination.
