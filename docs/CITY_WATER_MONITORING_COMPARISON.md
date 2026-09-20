# City / State Water Monitoring — Seoul, Tokyo (+ Singapore) vs Thimphu

**Question:** How do mature mega-city water monitoring systems work, and how does that compare to Thimphu’s source situation?  
**Use:** Set expectations for what your LoRaWAN + source-alert product can (and cannot) replace vs complement.

---

## 1. What “state / city-level” monitoring usually means

Mature utilities run **four layers**:

| Layer | What it does | Mega-city example |
|---|---|---|
| **1. Production** | Intake + WTP (treatment) SCADA: flow, chemicals, quality | Seoul Arisu WTPs; Tokyo purification plants |
| **2. Transmission / distribution** | Pressure, flow, reservoir levels, pump stations 24/7 | Tokyo Water Supply Operation Center |
| **3. District / DMA** | Zone balances to find leaks | Singapore DMAs + smart sensors |
| **4. Customer edge** | Smart meters, indoor leak alerts, apps | Seoul remote metering; Tokyo smart-meter trials |

Plus: **control room humans** (not only apps), GIS pipe maps, work crews, and increasingly **AI / digital twin**.

---

## 2. Seoul (Arisu / Seoul Waterworks)

**Maturity: very high (world-class utility brand).**

| Capability | Status (recent public direction) |
|---|---|
| Treatment plants | Full SCADA; moving to **AI-assisted / semi-autonomous** plant ops (from ~2025 TF / pilots; expand later decade) |
| Water quality | Heavy lab + continuous monitoring culture (Arisu markets hundreds of test items publicly) |
| Distribution | Digitized supply, pipe rehab programs, predictive leak / pipe management |
| Customer | **Smart remote metering** scaling (hundreds of thousands → multi-year path to citywide); **leak alert** service to households |
| Who operates | City utility (Arisu HQ) — not private building apps |

**Alert style:** Utility control room + citizen services (e.g. indoor leak notify). Residents trust tap brand “Arisu.”

---

## 3. Tokyo (Bureau of Waterworks)

**Maturity: very high; centralized ops.**

| Capability | Status |
|---|---|
| Control center | **Water Supply Operation Center** — 24h SCADA monitoring of pressure, flow, facilities |
| Scale | On order of **tens of thousands of data points** across **hundreds of facilities / pipeline points** (public descriptions cite ~24,000 data items / 177 facilities / 313 pipelines — order-of-magnitude sense of density) |
| Leak / anomaly | Flow + pressure anomalies trigger warnings for pipeline accidents |
| Customer edge | **Smart meter trial → mass rollout path** (hourly data, visualization, abnormality / leak / backflow detection; full deployment aimed into 2030s) |
| Who operates | Metropolitan Bureau of Waterworks |

**Alert style:** Central ops detects network events; customer apps/services grow with smart meters.

---

## 4. Singapore (PUB) — useful peer for “smart grid”

Not asked as primary, but useful benchmark:

- Island **Smart Water Grid**: sensors + analytics for leak localization, pressure, quality  
- **DMAs** (district metered areas) for NRW  
- Cloud anomaly tools (e.g. leak finder on large WDNs with many smart sensors)  
- Drainage also heavily instrumented (levels, flows, CCTV)

**Maturity: reference “digital utility” model.**

---

## 5. Thimphu reality + our phasing

**Thromde SCADA on paper ≠ live.** We keep a full Pamtsho WTP ERP blueprint for **later**.

**NOW:** individual buildings — escrow rent, cleaning proof, local water IoT (you already have local IoT).

| Layer | Now | Later |
|---|---|---|
| Building gateway + tank/inlet | **Yes** | Expand |
| Cleaning biometric + CCTV proof | **Yes** | Payroll from minutes |
| Escrow rent | **Yes** | Autopay |
| Pamtsho / main source ERP | Blueprint only | Thromde sale |

```text
NOW:   BUILDING IoT + escrow + cleaning proof
LATER: PAMTSHO WTP ERP (become Thromde ops system)
```

See [`PROJECT_FOCUS.md`](./PROJECT_FOCUS.md) and [`PAMTSHO_WTP_ERP_BLUEPRINT.md`](./PAMTSHO_WTP_ERP_BLUEPRINT.md).

---

## 6. Side-by-side (context only)

| Dimension | Seoul / Tokyo | Thimphu | Our NOW fit |
|---|---|---|---|
| Control room | Dense SCADA | Paper SCADA | Building dashboards first |
| Last mile | AMI | Weak | **Building water IoT** |
| Labour proof | Mature FM | Cleaning no-shows | **Biometric + CCTV** |
| Rent | Separate | Manager leakage | **Escrow** |

---

## 7. Strategy

1. Finish **local building** stack first.  
2. Do not block on main-source access.  
3. When ready, present Pamtsho ERP as separate Thromde platform.  

---

## 8. Bottom line

Mega-cities = dense utility grids. Thimphu paper SCADA is not your day-one dependency. **Win buildings first** (rent + cleaning proof + tank IoT), then municipal source ERP.
