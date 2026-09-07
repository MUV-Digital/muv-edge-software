# MUV Edge Software

Node-RED flow images that implement **MUV Bridge** on edge devices. Each flow connects a field protocol or vendor controller to the MUV cloud over MQTT.

## How a Bridge works

On startup the flow looks up its own Node-RED tab, then binds to the matching entry in `global.cloud.bridges` (the flow **label** is the bridge ID). Typical stages:

1. **Connections initialization** — load controller config and open the protocol client
2. **Virtual Read** — evaluate derived / virtual datapoints
3. **Protocol Read** — poll Modbus, BACnet, HTTP, or vendor APIs
4. **Update Metrics** — publish image health
5. **Execute Commands** — apply write/command requests from the cloud
6. **Handle errors** — report failures upstream

MQTT topics used by most images:

| Topic | Purpose |
| --- | --- |
| `cloud/upstream/data` | Telemetry |
| `cloud/upstream/error` | Faults |
| `cloud/upstream/response` | Command acknowledgements |

## Repository layout

Flows live under `nodered/`, grouped by vendor or site, then by protocol.

```
nodered/
├── generic/          Reusable Edge / Bridge images
│   ├── bacnet/       Generic BACnet Bridge
│   ├── edge/         Platform helpers (remote access, Ultra IoT i.MX8P)
│   └── modbus/       Generic Modbus Edge / Bridge (plus serial/ and tcp/)
├── muv/              Simulator, HTTP API, and E2E test images
├── automation/       AutomationDirect A1PM
├── calanda/          Site-specific Modbus (Weinkeller Vaduz)
├── carel/            Carel Mastercella
├── danfoss/          AK-SM-800A / AK-SM-880A (+ XML alarm)
├── eastron/          SDM630 energy meter
├── lasco/            LA150P
├── shelly/           PRO3EM
├── triopan/          Anlage, display, logo
├── wago/             PFC200
└── zrh/              Zurich site flows (BACnet pilots, AOP watchdog)
```

## Image naming

Generic Modbus / BACnet filenames encode the variant:

| Token | Meaning |
| --- | --- |
| `MUV-EDGE` | Earlier generic image family |
| `MUV-BRIDGE` | Current Bridge image family |
| `BUS-NN` | Bus poll interval in seconds (`05`, `30`, `60`, `90`) |
| `BLOCK-READ` | Multi-register / multi-object read |
| `SINGLE-READ` | One register / object at a time |
| `FLEXIBLE-CHANNEL` | Channel selected from bridge config |
| `CHANNEL-1` / `CHANNEL-2` | Fixed serial/TCP channel |
| `FLOAT-SWAP` | Word-swapped IEEE-754 floats |
| `serial` / `tcp` | Transport (folder under `generic/modbus/`) |

Prefer `nodered/generic/modbus/{serial,tcp}/MUV-BRIDGE-*.json` and `nodered/generic/bacnet/MUV-BRIDGE.json` for new deployments. Vendor and `zrh/` flows are device- or site-specific.

## Using a flow

1. Import the JSON into Node-RED on the edge device (or the image build that ships that flow).
2. Set the flow **label** to the bridge ID assigned in the cloud.
3. Provide controller settings on `global.cloud.bridges` for that ID (`bridge.image.controller`).

Do not treat placeholder values in the JSON as production config. Flow labels (`123123123123`, Mongo-style IDs from a previous export) and IP addresses (`127.0.0.1`, `192.168.x.x`, interface/broadcast addresses) are replaced when the image is bound to a bridge.
