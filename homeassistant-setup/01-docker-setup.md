# Home Assistant Setup — Docker on QNAP Container Station

## Prerequisites

- QNAP NAS with Container Station installed and running
- A static LAN IP assigned to the QNAP (e.g. `192.168.1.10`)
- A dedicated folder on the NAS for HA config (e.g. `/share/homeassistant/config`)
- Port 8123 free on the QNAP host

---

## 1. Create the config directory

In QNAP File Station (or SSH), create the directory:

```
/share/homeassistant/config
```

This is where all HA configuration files will live (`configuration.yaml`, `automations.yaml`, etc.).

---

## 2. docker-compose.yml

Create the file at `/share/homeassistant/docker-compose.yml`:

```yaml
version: "3.8"

services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/home-assistant:stable
    volumes:
      - /share/homeassistant/config:/config
      - /etc/localtime:/etc/localtime:ro
    network_mode: host        # REQUIRED for local device discovery (Hue, Govee, DIRIGERA)
    restart: unless-stopped
    privileged: true          # Required for some USB/Bluetooth integrations
    environment:
      - TZ=Europe/Zagreb      # Change to your timezone
```

> **Why `network_mode: host`?**
> Philips Hue bridge discovery uses mDNS/Bonjour. Govee LAN uses UDP broadcast.
> IKEA DIRIGERA uses mDNS. None of these work through Docker bridge NAT.
> Host mode makes HA behave as if it runs directly on the QNAP network interface.

---

## 3. Deploy via Container Station UI

### Option A — Container Station GUI (recommended for QNAP)

1. Open Container Station in QNAP web UI
2. Go to **Create** → **Create Application**
3. Paste the `docker-compose.yml` content above
4. Set the application name: `homeassistant`
5. Click **Create**

### Option B — SSH

SSH into the QNAP and run:

```bash
cd /share/homeassistant
docker compose up -d
```

---

## 4. First boot

After the container starts (takes ~60 seconds on first run):

1. Open a browser and navigate to `http://<QNAP-IP>:8123`
2. Complete the onboarding wizard (create admin account, set location/timezone)
3. HA will auto-discover some devices already (Hue bridge, DIRIGERA if on same LAN)

---

## 5. HTTPS setup for Google Home (required)

Google Home webhook requires a publicly accessible HTTPS endpoint.

### Using QNAP Application Portal (built-in reverse proxy)

1. In QNAP web UI → **Control Panel** → **Application Portal** → **Reverse Proxy**
2. Add a new rule:
   - Source: `https://yourdomain.com:443` (your public domain)
   - Destination: `http://localhost:8123`
3. Enable **Let's Encrypt** certificate in **Control Panel** → **Security** → **SSL Certificate & Private Key**
4. Make sure port 443 is forwarded on your router to the QNAP IP

### Alternative — nginx on QNAP

If you prefer nginx (install via Container Station):

```nginx
server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate     /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;

    location / {
        proxy_pass http://localhost:8123;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

---

## 6. Update configuration.yaml for reverse proxy

After setting up HTTPS, add this to `/share/homeassistant/config/configuration.yaml`:

```yaml
homeassistant:
  external_url: "https://yourdomain.com"
  internal_url: "http://192.168.1.10:8123"   # your QNAP LAN IP

http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - ::1
```

---

## 7. Keeping HA updated

To update to the latest stable image:

```bash
cd /share/homeassistant
docker compose pull
docker compose up -d
```

Or use the **Watchtower** container for automatic updates (optional).

---

## Directory layout after setup

```
/share/homeassistant/
├── docker-compose.yml
└── config/
    ├── configuration.yaml
    ├── automations.yaml
    ├── scenes.yaml
    ├── scripts.yaml
    ├── custom_components/     ← HACS integrations installed here
    └── www/                   ← custom frontend resources
```
