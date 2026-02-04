# Victron Node-RED

Node-RED flows for monitoring Victron Energy systems via MQTT.

**Created by Lubos Strejcek during [Victron Energy Training Center Brno](https://www.victronenergy.com/) - Node-RED Training**

> This repository is public and free to use for inspiration or as a starting point for your own Victron monitoring setup.

## Overview

Node-RED flows that:
- Subscribe to Victron MQTT topics from Ekrano GX
- Parse and validate energy data
- Write to InfluxDB time-series database
- Provide HTTP API for Shelly device control
- Monitor system health and errors

## Quick Start

```bash
# Run with Docker
docker run -d --name victron-nodered \
  -p 1880:1880 \
  -v $(pwd):/data \
  nodered/node-red:latest

# Access UI
open http://localhost:1880
```

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

- [victron-influxdb](https://github.com/streyda/victron-influxdb) - InfluxDB configuration
- [victron-grafana](https://github.com/streyda/victron-grafana) - Grafana dashboards

## License

MIT License

## Acknowledgments

- [Victron Energy Training Center Brno](https://www.victronenergy.com/)
