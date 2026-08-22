# 🌍 GeoDispatch

**Autonomous crisis dispatcher for the MENA region.**

GeoDispatch is a real-time disaster response pipeline that detects seismic events, locates affected people via network APIs, and autonomously dispatches SMS alerts and rescue teams — in under 4 seconds.

---

## How it works

```
Disaster Sensor  →  Go Supervisor  →  Python AI Agent  →  SMS + Rescue Dispatch
                          │                                        │
                    Nokia NaC CAMARA                      React Dashboard
                    (location + network)                  (live gov map)
```

When a disaster event is detected, the pipeline:

1. Fetches the location and network reachability of every device in the affected radius via the **CAMARA Network APIs** (Nokia NaC)
2. Assigns each device a danger zone (`red` / `orange` / `green`) using haversine distance from the epicenter
3. Sends batches of triaged devices to the **AI agent**, which decides the action per device: SMS evacuation message, rescue flag, or both
4. Dispatches SMS and rescue flags in parallel, streaming live updates to the **government dashboard**

---

## Repositories

| Repo | Description | Stack |
|---|---|---|
| [supervisor](https://github.com/geoDispatch/supervisor) | Pipeline orchestration — CAMARA API calls, zone assignment, AI batching, SMS dispatch | Go |
| [agent](https://github.com/geoDispatch/agent) | AI triage agent — receives device batches, decides actions, generates situation reports | Python |
| [dashboard](https://github.com/geoDispatch/dashboard) | Live government map — real-time device dots, zone counters, AI narratives | SolidJS + Leaflet |
| [contracts](https://github.com/geoDispatch/contracts) | Shared JSON schemas for all service boundaries | JSON Schema |
| [deploy](https://github.com/geoDispatch/deploy) | Docker Compose and deployment configuration | Docker |

---

## Architecture

```
                        ┌─────────────────────────────────┐
                        │         Go Supervisor           │
  POST /sensor ────────▶│                                 │
  (disaster event)      │  ┌─ Goroutine A ─────────────┐ │
                        │  │  CAMARA Geofencing + QoS   │ │
                        │  └───────────────────────────┘ │
                        │  ┌─ Goroutine B ─────────────┐ │
                        │  │  DB: 3 nearest shelters    │ │
                        │  └───────────────────────────┘ │
                        │  ┌─ Per-device (semaphore 50) ┐ │
                        │  │  Location + Reachability   │ │
                        │  │  → Haversine → Zone        │ │
                        │  └───────────────────────────┘ │
                        │         ↓ sorted by distance   │
                        │    Batches of 20 devices       │
                        └────────────┬────────────────────┘
                                     │ POST /decide
                                     ▼
                        ┌─────────────────────────────────┐
                        │       Python AI Agent           │
                        │  per device: sms|rescue|both    │
                        │  picks best shelter             │
                        │  generates gov narrative        │
                        └────────────┬────────────────────┘
                                     │
                    ┌────────────────┼────────────────┐
                    ▼                ▼                ▼
               SMS Dispatch    Rescue Flag      WebSocket
               (per device)    (red zone)    → Dashboard
```

---

## Key design rules

- **Go calculates zones. AI decides actions. Never reversed.**
- All phones E.164 format (`+212XXXXXXXXX`)
- All timestamps Unix milliseconds (`int64`)
- All zones lowercase: `red` | `orange` | `green`
- Rescue flagging is red zone only
- The `reasoning` field in AI decisions is internal audit only — never shown to end users

---

## Built for

**MENA Hackathon** — a real-time disaster response system built on top of the Nokia Network as Code (NaC) CAMARA APIs, combining network intelligence with AI triage to save lives faster.

---

*Morocco 🇲🇦 · [iliaszahr.professional@gmail.com](mailto:iliaszahr.professional@gmail.com)*
