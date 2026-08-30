# Water Issue Alerts — From Source

**Need:** When something is wrong at the **Thimphu water source** (or on the path from source), the right people get **instant notifications** — not only when the building tank is already empty.

---

## 1. Why “from source” matters

| If you only alert on owner tank | What you miss |
|---|---|
| Tank empty at 6am | Source failed at midnight — hours of no warning |
| Residents blame building manager | Problem was upstream (source / main) |
| Emergency tanker every time | Could have known “city supply down” earlier |

**Rule:** Alert as soon as the **source stage** fails, and tag every alert with **where it started** (`SOURCE` | `MAIN` | `ZONE` | `BUILDING`).

```text
Source sensor trip
    → alert engine
    → notify Owner + Facilities + PM  (and optional Residents: “supply disruption”)
    → optional auto work order
    → link CCTV clip at source
```

---

## 2. Source alert types (what triggers)

| Code | Condition at source | Severity | Meaning |
|---|---|---|---|
| `SRC_LEVEL_LOW` | Intake/reservoir level &lt; low % | **P1** | Supply shortage starting |
| `SRC_LEVEL_CRITICAL` | Level &lt; critical % | **P0** | Source nearly dry |
| `SRC_FLOW_DROP` | Outflow drop &gt; X% vs baseline (e.g. 30 min) | **P1** | Cut, valve closed, or pump fail |
| `SRC_FLOW_ZERO` | Flow ≈ 0 while demand window | **P0** | No water leaving source |
| `SRC_PUMP_FAIL` | Pump commanded ON but status OFF / no current | **P0** | Equipment failure |
| `SRC_TURBIDITY_HIGH` | Quality over limit (if sensor fitted) | **P1** | Contaminated / flood event |
| `SRC_NODE_OFFLINE` | No LoRa uplink &gt; N hours | **P2** | Blind — treat as risk |
| `SRC_CCTV_OFFLINE` | Camera unreachable | **P3** | No visual backup |
| `SRC_TAMPER` | Door/motion at intake (optional) | **P1** | Security |

Downstream (still “from source path,” not only building):

| Code | Condition | Severity |
|---|---|---|
| `MAIN_PRESSURE_DROP` | Pressure collapse on feeder | P0/P1 |
| `MAIN_LEAK_ZONE` | Flow/pressure mismatch between nodes | P1 |
| `ZONE_LEVEL_LOW` | Intermediate tank low | P1 |
| `BLDG_INLET_DRY` | Building inlet flow zero while upstream OK | P1 |
| `TANK_LEVEL_LOW` / `TANK_CRITICAL` | Owner tank | P1 / P0 |

---

## 3. Notification pipeline

```text
LoRaWAN source node uplink
        │
        ▼
ChirpStack → MQTT → mqtt-bridge / alert-service
        │
        ├─ write sensor_readings
        ├─ evaluate rules (thresholds + rate-of-change)
        ├─ dedupe (same code + asset: min interval e.g. 15–30 min)
        ├─ create alerts row (stage=SOURCE, severity, message)
        │
        ├─ Push  → Owner, Facilities, PM apps (FCM / APNs / Expo)
        ├─ SMS   → P0/P1 phones (Twilio / local Bhutan SMS gateway)
        ├─ In-app inbox + banner
        ├─ Optional: Telegram/WhatsApp ops group (gateway bot)
        └─ Auto work_order if P0/P1 and rule says so
                └─ attach last CCTV snapshot/clip URL
```

**Latency target:** sensor sample 1–5 min at source under stress; notify within **&lt; 1 minute** of rule fire after uplink.

---

## 4. Who gets which alert

| Role | Source P0 | Source P1 | Source P2/P3 | Building tank only |
|---|---|---|---|---|
| **Building owner** | Push + SMS | Push | In-app | Push (+ SMS if P0) |
| **Facilities / water ops** | Push + SMS | Push + SMS | Push | Push |
| **Property manager** | Push | Push | In-app | Push |
| **Residents** (optional) | Broadcast: “Water supply issue from source — expect low pressure” | Same digest (not every sensor blip) | — | “Building tank low” only if shared tank |
| **Platform support** | P0 webhook | — | Offline sensors | — |

Residents get **human messages**, not raw sensor codes — avoids panic spam.

---

## 5. Example notification copy

**Owner / Facilities (P0):**
> Water source alert: **Motithang / [Source name]** flow is zero. Stage: SOURCE. Opened work order #182. [View CCTV] [Ack]

**Owner (P1):**
> Source level low (18%). Your building tank may drop in the next few hours. Stage: SOURCE.

**Resident (broadcast):**
> Temporary water supply disruption reported at the main source. We are monitoring. Tank status in app.

---

## 6. Acknowledge & escalate

```text
Alert created (status=open)
    → notify recipients
    → Facilities/PM taps Acknowledge (status=acked, who, when)
    → if not acked in 15 min (P0) / 60 min (P1)
          → escalate SMS to Owner + backup phone
          → optional second ops contact
    → Resolve when readings back to normal for M minutes
          OR manual resolve with note
```

Owner always sees **open source alerts** on home screen even if manager acks — stops “manager ate the info.”

---

## 7. Data model (alerts)

```text
alerts
  id, org_id, asset_id, sensor_id
  stage          -- SOURCE | MAIN | ZONE | BUILDING
  code           -- SRC_FLOW_ZERO, ...
  severity       -- P0 | P1 | P2 | P3
  title, body
  reading_value, threshold
  status         -- open | acked | resolved
  acked_by, acked_at
  work_order_id
  cctv_clip_url
  created_at, resolved_at

alert_subscriptions
  user_id, org_id, stages[], min_severity, channels[]  -- push,sms,email

notification_log
  alert_id, user_id, channel, sent_at, provider_id, status
```

RBAC: source-stage alerts visible to all owners/buildings **fed by that source** (map `building.source_id`).

---

## 8. Rules engine (simple MVP)

1. On each source reading: compare to thresholds in `assets.alert_config`  
2. Rate-of-change: flow drop &gt; 40% in 30 minutes → `SRC_FLOW_DROP`  
3. Hysteresis: resolve only after level &gt; low + 5% for 20 minutes (no flap)  
4. Quiet hours: still send **P0**; delay P3 to morning  
5. Storm control: max 1 SMS per code per asset per 30 minutes  

---

## 9. Channels for Bhutan / Thimphu

| Channel | Use |
|---|---|
| **Mobile push** | Default for all P1+ |
| **SMS** | P0 + unacked P1 (local SMS API or Twilio if available) |
| **In-app** | History + ack |
| **Optional call** | P0 only, later |
| **Ops WhatsApp/Telegram group** | Facilities team mirror |

No reliance on manager WhatsApp as the only channel — system notifies **owner directly from source**.

---

## 10. MVP to ship first

1. Source (or nearest upstream) level **or** flow sensor on LoRaWAN  
2. Rules: low / critical / flow zero  
3. Push to owner + facilities  
4. SMS on P0  
5. Alert list in app with stage badge **SOURCE**  
6. Auto-clear when healthy  

Then add CCTV link + resident broadcast.

---

## 11. One-line product promise

**If the source fails, the building owner is notified from the source — before the rooftop tank runs dry.**
