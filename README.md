# Victron Node-RED

Node-RED flows for monitoring Victron Energy systems via MQTT.

**Created by Lubos Strejcek during [Victron Energy Training Center Brno](https://www.victronenergy.com/) - Node-RED Training**

> This repository is public and free to use for inspiration or as a starting point for your own Victron monitoring setup.

## Overview

Node-RED with **flows pre-loaded** (one configuration step needed):
- All flows ready to use - just configure your Ekrano GX IP address
- Subscribes to Victron MQTT topics
- Parses and writes data to InfluxDB
- Includes Shelly device control API

## Quick Start

```bash
# Clone and run
git clone https://github.com/lubosstrejcek/victron-nodered.git
cd victron-nodered
cp .env.example .env
docker compose up -d

# Access UI
open http://localhost:1880
```

## Configuration (One-Time Setup)

**Only step needed:** Set your Ekrano GX IP address.

1. Open Node-RED: http://localhost:1880
2. Double-click any **MQTT node** (e.g., "Battery SoC")
3. Click the pencil icon next to the broker
4. Change **Server** to your Ekrano GX IP (ask your instructor)
5. Click **Update** → **Deploy**

Each Victron training lab has a different Ekrano GX IP address.

> **Note:** InfluxDB connection is pre-configured to `host.docker.internal:8086`

## Flow Structure

| Tab | Purpose |
|-----|---------|
| 1. MQTT Input | Subscribe to Victron topics |
| 2. Data Processing | Parse JSON, validate, transform |
| 3. InfluxDB Output | Write to database |
| 4. Digital Inputs | Monitor switches/sensors |
| 5. Shelly Control | HTTP API for Shelly |
| 6. Errors | Error catching and logging |

## MQTT Topics

```
N/{portal_id}/battery/256/#        # Battery (BMV-702)
N/{portal_id}/solarcharger/258/#   # MPPT Solar
N/{portal_id}/vebus/257/#          # Inverter
N/{portal_id}/grid/30/#            # Grid Meter
N/{portal_id}/pvinverter/32/#      # AC PV Inverter
N/{portal_id}/system/0/#           # System Overview
N/{portal_id}/tank/#               # Tank Sensors
N/{portal_id}/temperature/22/#     # RuuviTag
N/{portal_id}/digitalinput/#       # Digital Inputs
```

## InfluxDB Measurements

| Measurement | Fields |
|-------------|--------|
| `battery` | soc, voltage, current, power |
| `solar` | power, pv_voltage |
| `grid` | power_total, power_l1/l2/l3 |
| `inverter` | state, ac_power |
| `tank` | level_1, level_2 |
| `ruuvi` | temperature, humidity, pressure |
| `shelly` | output_state, power |

## Shelly Control API

```bash
curl -X POST http://localhost:1880/shelly/on
curl -X POST http://localhost:1880/shelly/off
curl -X POST http://localhost:1880/shelly/toggle
```

## Related Repositories

- [victron-influxdb](https://github.com/lubosstrejcek/victron-influxdb) - InfluxDB configuration
- [victron-grafana](https://github.com/lubosstrejcek/victron-grafana) - Grafana dashboards

## License

MIT License

## Acknowledgments

- [Victron Energy Training Center Brno](https://www.victronenergy.com/)
