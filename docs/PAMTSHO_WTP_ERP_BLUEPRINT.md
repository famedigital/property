# Pamtsho WTP Water Utility ERP — Groundwork Blueprint

**Anchor site:** Pamtsho / Jungzhina–Pamtsho Thimphu Water Treatment Plant + distribution  
(~1.4 MLD WTP, reservoirs for Pamtsho–Jungzhina / Jagom, ~23 km network, planned meters)  
**Buyer:** Thimphu Thromde  
**Positioning:** Independent **complete Water Utility ERP** for Thromde — **implement after** building product (escrow + cleaning + building water IoT) is live. See [`PROJECT_FOCUS.md`](./PROJECT_FOCUS.md).  
Thromde “SCADA” is paper-only; this platform becomes the operational system of record **when municipal phase starts**.  
**Isolation:** Municipal tenant (`org_type = municipal`) never sees private property escrow/landlord data.

---

## 1. Physical architecture (site groundwork)

### 1.1 Zones on the ground

```text
INTAKE / SOURCE          WTP COMPOUND                    DISTRIBUTION
┌──────────────┐         ┌─────────────────────┐         ┌──────────────────┐
│ Raw intake   │────────►│ Process units       │────────►│ Reservoirs       │
│ Level/flow   │         │ MCC / pumps         │         │ (Pamtsho/Jagom)  │
│ Pump status  │         │ Lab · Stores        │         │ LoRa nodes       │
│ CCTV         │         │ Control room        │         │ Customer meters  │
└──────────────┘         │ Edge cabinet (OT)   │         └──────────────────┘
                         │ CCTV plant          │
                         └─────────────────────┘
```

| Zone | Physical kit | Power | Purpose |
|---|---|---|---|
| **Intake / source** | Level + flow transmitters, pump dry-contacts, IP67 LoRa node, CCTV | Solar + battery (grid if available) | Source health + security |
| **WTP process hall** | 4–20 mA / Modbus → PLC or I/O → edge; residual Cl₂, turbidity where fitted | Grid + UPS 30–60 min | Process visibility |
| **MCC / pump room** | CT / run-status, optional vibration, leak rope | Grid + UPS | Pump health |
| **Lab** | Bench PC/tablet → ERP lab module; barcode sample IDs | Grid | Quality ERP |
| **Stores** | Rugged tablet/handheld | Grid | GRN / issue |
| **Control room** | 1–2 operator PCs, wall display, optional SMS modem | UPS | Alarm console |
| **Edge cabinet (OT)** | Industrial PC, LoRaWAN gateway, 4G router, switch, PSU | UPS | Local brain |
| **Reservoirs** | Ultrasonic/radar level, outlet flow, LoRa node, optional CCTV | Solar | Tank balance |
| **Network chambers** | Pressure (+ flow where possible), IP67 node | Battery/solar | Leak localization |

### 1.2 Edge cabinet bill of materials (standard bay)

- IP55/IP65 outdoor-rated cabinet, DIN rail  
- 24 V PSU + UPS (LiFePO4 or equivalent)  
- Industrial switch (VLAN-capable)  
- LoRaWAN gateway + outdoor antenna on mast  
- 4G/LTE router with **dual-SIM** failover (Bhutan telecom)  
- Edge computer (Ubuntu LTS): MQTT bridge agent, local SQLite buffer, ingest service, **WireGuard** client  
- Optional: small PLC / remote I/O for hardwired 4–20 mA  
- Surge protection + earthing per local electrical code  

### 1.3 Control room

- Operator desks with ERP web UI  
- Large display: plant overview + alarm list  
- **IT VLAN only** (no direct internet browsing of OT devices)  
- Printer for shift reports / lab certificates  

### 1.4 Civil / install sequence

1. Site survey (RF, power, 4G/fiber, chamber access)  
2. Mast + gateway + edge cabinet at WTP  
3. Instruments + nodes: intake → plant → reservoirs  
4. Control room network drop  
5. Commissioning: OTAA join, thresholds, **alarm drill** with operators  
6. Expand chamber nodes along feeders  

---

## 2. Network architecture (OT / IT / cloud)

### 2.1 Segmentation

```text
OT field: Sensors (LoRa/Modbus) → Gateway → Edge PC
IT plant: Operator PCs, Lab PC  ──────────────────────────┐
WAN:      dual-SIM 4G (+ fiber if Thromde provides)        │
Cloud:    ChirpStack → MQTT → ERP API → Postgres RLS      │
          AI Gateway (enrich only)                        │
          ◄──────────── WireGuard (outbound from edge) ───┘
```

| Plane | Technology | Notes |
|---|---|---|
| **Field RF** | LoRaWAN **AS923** (confirm Bhutan allocation before bulk buy) | Sensors only |
| **Field serial** | RS485 Modbus → edge I/O | Existing plant instruments |
| **Video** | 4G / LAN / NVR — **never LoRaWAN** | WTP + intake CCTV |
| **Plant LAN** | VLAN **OT** vs VLAN **IT** | Firewall; default deny |
| **Backhaul** | Primary 4G; secondary SIM; fiber later | WireGuard to cloud |
| **Cloud** | ChirpStack + MQTT + ERP (Vercel/Supabase/Hetzner) | Municipal org |
| **Remote admin** | VPN jump only; MFA | No open RDP on edge |

### 2.2 Addressing & security

- OT private net e.g. `10.20.0.0/24`  
- Edge initiates **outbound-only** WireGuard  
- Device OTAA keys in secrets manager — never in mobile apps  
- MQTT ACL prefix: `thromde/pamtsho/...`  
- TLS on IT/cloud; LoRaWAN air encryption  

### 2.3 Failure modes

| Failure | Behavior |
|---|---|
| 4G down | Edge buffers; local SMS on P0 if modem present |
| Gateway down | Nodes retry; gateway health alarm |
| Cloud down | Local alarm / SMS; sync on reconnect |
| AI down | Template alarms still fire |

---

## 3. Software architecture (Water Utility ERP)

### 3.1 Logical apps

| App | Role |
|---|---|
| `apps/web` | Next.js **Thromde workspace** (plant ERP UI) |
| `apps/mobile` | Expo field app for network/maintenance crews |
| `apps/mqtt-bridge` | ChirpStack/MQTT → DB + alert-service |
| `packages/db` | Drizzle schema + RLS |
| `packages/shared` | Zod types, alarm codes, roles |

### 3.2 ERP modules (system of record)

| Module | Core entities | MVP |
|---|---|---|
| **Org & RBAC** | plant, roles: PlantMgr, Operator, Lab, Stores, Network, Admin | Yes |
| **Plant ops** | process tags, last values, trends, shift log | Yes |
| **Alarms** | code, severity, ack, escalate, WO link | Yes |
| **Assets / CMMS** | asset, PPM, work_order, downtime | Yes |
| **Stores** | item, stock, GRN, issue, reorder | Yes |
| **Procurement** | vendor, PR, PO, approval | Phase 2 |
| **Lab / quality** | sample, result, limit, NCR | Yes (manual first) |
| **Distribution** | reservoir, node, zone balance | Yes |
| **Metering / billing hooks** | meter, reading, export to Thromde billing | Phase 2 |
| **HR light** | roster, attendance | Phase 2 |
| **Reports** | daily production, chemical use, alarms | Yes |
| **AI assist** | explain, brief, WO draft | After alarms stable |
| **CCTV links** | camera, clip on alarm | Pilot camera |

### 3.3 Data model sketch (municipal)

```text
orgs (type=municipal)
  └── plants
        ├── process_areas → tags / sensors
        ├── assets → work_orders
        ├── store_items → stock_moves
        ├── lab_samples → lab_results
        ├── reservoirs → network_nodes
        ├── alarms / shift_logs
        └── meters → meter_readings
```

All RLS by `org_id`. Property escrow tables are **unreachable** from municipal roles.

### 3.4 Alert codes (WTP-focused)

- `WTP_TURBIDITY_HIGH`, `WTP_CL2_LOW`, `WTP_PUMP_FAIL`, `WTP_POWER_LOSS`  
- `RES_LEVEL_LOW`, `MAIN_PRESSURE_DROP`, `MAIN_LEAK_ZONE`  
- `GW_OFFLINE`, `CAM_OFFLINE`  
- Source-path codes from [`WATER_SOURCE_ALERTS.md`](./WATER_SOURCE_ALERTS.md) where applicable  

**Notify:** Plant Manager + Network crew (+ Thromde duty phone on P0).

### 3.5 Integration map

| External | Direction | MVP |
|---|---|---|
| LoRaWAN / ChirpStack | In | Yes |
| Lab instruments | CSV / manual | Yes |
| Thromde billing / GIS | Export API | Phase 2 |
| Thromde accounting | PO / invoice export | Phase 2 |
| Property escrow SaaS | **None** (isolated) | — |
| RRCO | **No landlord data** (WTP ERP unrelated) | — |

---

## 4. Cloud / deploy blueprint

| Component | Placement |
|---|---|
| ERP web + API | Vercel (Hetzner/on-prem if Thromde mandates) |
| Postgres + RLS | Supabase or Hetzner Postgres |
| ChirpStack + MQTT | Hetzner VPS (VPN to edge) |
| Object storage | R2/S3 — CCTV clips, lab PDFs |
| AI | Vercel AI Gateway (`AI_GATEWAY_API_KEY` server-only) |
| Monitoring | Grafana + edge/gateway uptime |

**On-prem option:** Docker Compose in Thromde server room; edge stays on site; support contract for ops.

---

## 5. Phased rollout (Pamtsho)

1. Survey + cabinet + gateway + control room link  
2. ERP MVP: RBAC, plant tags, alarms, assets/WO, stores, reservoirs  
3. Instrument source + plant + one reservoir  
4. Alarm drill with Thromde operators  
5. Feeder nodes + leak logic  
6. Lab module + daily reports  
7. Meters / billing hooks  
8. Multi-plant expand (other Thimphu WTPs) on same ERP  

---

## 6. Product split reminder

| Product | Buyer | Scope |
|---|---|---|
| Property + escrow rent | Building owners / PM | Units, escrow, private tanks |
| **Pamtsho Water Utility ERP** | Thromde | This blueprint — complete plant + network ERP |

Do **not** plan “coexist with Thromde SCADA.” We **are** the operational monitoring and ERP system; pitch Thromde a live pilot at Pamtsho.
