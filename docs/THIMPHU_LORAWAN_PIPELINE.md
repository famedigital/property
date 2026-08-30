# Thimphu Source → Building Tank — LoRaWAN + CCTV Pipeline

**Scope:** Monitor the **entire water journey** from Thimphu main source to the **building owner tank**, with process sensors + CCTV at critical points.  
**Networking:** **LoRaWAN for sensors** · **separate backhaul for CCTV** (video cannot ride LoRaWAN).  
**Fits product:** Same property platform (owner dashboard, RBAC, alerts, rent later).

---

## 1. What you are building

```text
SOURCE (Thimphu intake / WTP / reservoir)
    │  sensors + CCTV
    ▼
TRANSMISSION / MAINS (hills → city)
    │  pressure + flow nodes along line
    ▼
ZONE / INTERMEDIATE TANKS
    │  level + outflow
    ▼
BUILDING INLET / METER
    │  flow into property
    ▼
OWNER ROOFTOP / GROUND TANK
    │  level (your current product heart)
    ▼
FLATS / UNITS
```

**Goal:** Owner and ops see *where water is* and *where it was lost* — not only “tank empty.”

Thimphu Thromde already uses SCADA / flow monitoring on some schemes (e.g. Jungzhina–Pamtsho, Chamgang). Your product can start as **building + private feeder** monitoring and optionally **align/export** with municipal SCADA later — do not assume you replace Thromde’s network on day one.

---

## 2. Networking rule (critical)

| Traffic | Network | Why |
|---|---|---|
| Level, flow, pressure, turbidity, battery, valve state | **LoRaWAN** | Long range, low power, cheap, mountain-friendly |
| CCTV live / recorded video | **4G/5G, fiber, or Wi‑Fi** — **not LoRaWAN** | Video needs Mbps; LoRaWAN is bytes–kilobytes |
| App alerts / rent / dashboards | Internet (same cloud as today) | MQTT/HTTPS into your platform |

```text
Sensors ──LoRa──► LoRaWAN Gateway ──4G/Ethernet──► Network Server ──► Your cloud
CCTV    ──4G/fiber/NVR──────────────────────────► VMS / object storage ──► Your cloud (clips + links)
```

---

## 3. Process sensor map (source → owner tank)

### Stage A — Main source / intake / WTP outlet

| Sensor / device | Measures | Alert when |
|---|---|---|
| Ultrasonic / radar level | Reservoir / intake level | Too low / overflow |
| Electromagnetic / ultrasonic flow | Outflow from source | Sudden drop (cut) or spike |
| Turbidity / basic quality (optional) | Dirt / flood contamination | Above threshold |
| Pump run status (dry contact) | Pump on/off | Fail while commanded on |
| **CCTV** | Intake, fence, pump house | Motion / tamper / visual verify |

### Stage B — Transmission line (source → city / zone)

Place nodes every **reasonable hydraulic segment** (e.g. every 0.5–2 km, or at every valve chamber / ridge / valley):

| Sensor | Measures | Alert when |
|---|---|---|
| Pressure transducer | Line pressure | Drop = leak/burst; spike = valve slam |
| Flow (where chambers allow) | Volume rate | In−out imbalance between nodes = leak zone |
| Manhole/chamber door (optional) | Tamper | Opened |
| Node battery / solar | Power health | Low battery |

**Leak idea:** compare flow/pressure between consecutive nodes; localize between node *i* and *i+1*.

### Stage C — Intermediate / zone tanks

| Sensor | Measures |
|---|---|
| Tank level | % / liters |
| Outlet flow | To distribution |
| Inlet flow (optional) | From mains |
| CCTV (optional) | Tank compound security |

### Stage D — Building owner property

| Sensor | Measures |
|---|---|
| Building inlet flow / pulse meter | Water entering property |
| **Owner tank level** (ultrasonic/pressure) | Usable water in tank |
| Overflow float / leak rope (optional) | Waste / basement leak |
| Pump to overhead tank status | Filling cycle |
| Optional small CCTV | Tank room / compound |

### Stage E — Inside building (later)

Unit meters / shared riser — Phase 2.

---

## 4. LoRaWAN architecture (Thimphu-ready)

```text
┌─────────────┐   LoRa RF (AS923 — Asia / Bhutan planning)
│ End nodes   │─────────────────────────────────────┐
│ (sensors)   │                                     │
└─────────────┘                                     ▼
                                          ┌──────────────────┐
                                          │ LoRaWAN Gateway  │
                                          │ (hill / rooftop) │
                                          │ + solar + 4G     │
                                          └────────┬─────────┘
                                                   │ backhaul
                                                   ▼
                                          ┌──────────────────┐
                                          │ Network Server   │
                                          │ ChirpStack /     │
                                          │ TTN / Helium /   │
                                          │ private NS       │
                                          └────────┬─────────┘
                                                   │ MQTT / HTTPS
                                                   ▼
                                          ┌──────────────────┐
                                          │ Your platform    │
                                          │ mqtt-bridge →    │
                                          │ Postgres + alerts│
                                          └──────────────────┘
```

### Recommended choices

| Piece | Recommendation |
|---|---|
| **Frequency** | Plan for **AS923** (Asia); confirm Bhutan regulatory allowance before RF buy |
| **Network server** | **ChirpStack** on your VPS (private, cheap, full control) |
| **Gateway** | Outdoor IP67, 4G backhaul, solar + battery (hills / unreliable grid) |
| **Nodes** | Battery or solar LoRaWAN nodes; IP67; 1–4 analog/digital inputs or Modbus |
| **Payload** | Tiny binary: `node_id, stage, value, battery, seq` every 5–15 min + event uplink |
| **Security** | OTAA join, AppKey per device, no shared default keys |

### Coverage planning (Thimphu hills)

- Gateways on **high points** (ridge, WTP roof, tall building) beat dense street placement  
- Expect **2–10+ km** line-of-sight; less in deep valleys — site-survey with a test kit  
- One gateway can serve many nodes; start with **1–2 gateways** on the source→tank corridor you care about  

### Cost-aware mode

Same as earlier plan: gateway buffers if backhaul drops; nodes keep sampling; sync when 4G returns. Critical alarms can add **SMS modem at gateway** as backup.

---

## 5. CCTV architecture (main source + key points)

**Do not put cameras on LoRaWAN.**

```text
Camera (source / pump house / owner tank)
    → NVR on site  OR  4G camera → cloud VMS
    → Event clips (motion / alarm-linked) uploaded to R2/S3
    → App shows: live (if bandwidth) + last clip + deep link
```

| Placement | Purpose |
|---|---|
| Main source / intake | Tamper, flood, visual confirm “source dry vs pipe break” |
| Pump house | Theft, pump running verification |
| Optional zone tank | Compound security |
| Owner tank compound | Local disputes / overflow evidence |

**Privacy / law:** CCTV on municipal source may need Thromde / authority permission. Private building tanks = owner consent. Store retention policy (e.g. 7–30 days).

**Link to sensors:** When pressure collapses or tank empty, auto-attach **nearest camera clip** to the alert ticket.

---

## 6. How this joins your property app

| Platform concept | Pipeline meaning |
|---|---|
| `org` / owner | Building owner who sees *their* tank + *their* feeder segment |
| `asset` | Source, main segment, zone tank, building tank |
| `sensor` | LoRaWAN device_eui mapped to asset |
| `sensor_readings` | Level / pressure / flow time series |
| `alerts` | Empty, leak zone, pump fail, camera offline |
| `work_orders` | Dispatch tanker / plumber / Thromde escalate |
| RBAC | Owner sees own building path; city ops role (later) sees corridor |

**Product MVP for this theme (hardware + software):**

1. Owner tank level (LoRaWAN)  
2. Building inlet flow  
3. One upstream pressure/flow node (or zone tank level)  
4. One CCTV at source *or* pump (if you have access)  
5. Dashboard: **source → … → my tank** status strip  

Full city mains = partnership / permit with Thromde — plan private feeder first if access is blocked.

---

## 7. Build-your-own sensor node (practical BOM sketch)

For a DIY / local-assembly node (prototype):

| Part | Role |
|---|---|
| MCU + LoRa (e.g. STM32WL / ESP32 + LoRa module / ready LoRaWAN node) | Radio + logic |
| Ultrasonic (JSN-SR04T / better industrial) or submersible pressure | Tank / reservoir level |
| 4–20 mA pressure transmitter | Pipeline pressure |
| Pulse / Modbus flow meter | Flow |
| Solar 10–20 W + LiFePO4 | Off-grid power |
| IP67 box + gland | Weather |
| Optional RS485 | Talk to existing meters |

**Production path:** prefer certified LoRaWAN industrial nodes (Ellenex-class / Dragino / local SI) once pilot works — less RF headache in Bhutan winters.

Firmware duties: sample → encode → OTAA uplink → sleep; downlink for threshold config; local store if ADR/link fails.

---

## 8. Ops: entire process monitoring

```text
Normal:     Source OK → Pressure OK along path → Zone OK → Building inlet flowing → Owner tank healthy
Problem A:  Source OK + mid-line pressure collapse     → LEAK between nodes (dispatch line crew)
Problem B:  Source low + all downstream empty          → SOURCE shortage (not building fault)
Problem C:  Inlet flow OK + owner tank not rising      → Local pump/float/valve fault
Problem D:  Tank empty + CCTV shows source full        → Distribution fault (prove with data + video)
```

This stops blame games: **manager / Thromde / building** — data shows which stage failed.

**Alerts from source:** see [`WATER_SOURCE_ALERTS.md`](./WATER_SOURCE_ALERTS.md) — triggers, who is notified, push/SMS, escalation.

---

## 9. Phased rollout (Thimphu)

1. **Pilot one building:** owner tank level + inlet meter + ChirpStack + 1 gateway  
2. **Add one upstream:** nearest zone tank or feeder pressure node  
3. **Add CCTV** at accessible pump/source point (permits!)  
4. **Corridor densify:** more pressure/flow chambers along *your* feeder  
5. **Integrate / coexist** with Thromde SCADA if they open APIs or Modbus  

---

## 10. Summary

- **LoRaWAN** = nervous system for the whole water path (tiny telemetry, long range, solar).  
- **CCTV** = eyes at main source (and key sites) on **4G/fiber**, linked to alerts.  
- **App** = owner sees full strip from source → tank; rent/anti-skimming stays separate money plane.  
- Start with **owner tank + feeder**, expand upstream as access and gateways allow.
