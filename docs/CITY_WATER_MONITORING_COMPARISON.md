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

## 5. Thimphu (Thromde + sources)

**Maturity: early / emerging digital utility — not Seoul/Tokyo scale.**

| Capability | What exists publicly |
|---|---|
| New schemes | e.g. **Jungzhina–Pamtsho**: WTP + ~23 km network + **SCADA** + **~260 automated meters** (ADB-backed) |
| Other works | Chamgang / Motithang etc. upgrades; AMR pilots (e.g. Upper Motithang historically); climate-smart flow monitoring tenders at WTP |
| Wastewater | Babesa plant with PLC/SCADA (separate from drinking supply) |
| Citywide density | **Not** Tokyo-style 24k-point citywide ops center for every main |
| Building / private tanks | Often **outside** Thromde visibility — rooftop tanks, local pumps, manager ops |
| Pain today | Irregular supply in areas, limited end-to-end visibility, cash/manager opacity on buildings |

**Thimphu source reality:** Multiple intakes / WTPs / reservoirs serving zones (hills → city). Instrumentation is **project-by-project**, not yet one continuous “source → every building tank” citizen-facing strip like your product vision.

---

## 6. Side-by-side comparison

| Dimension | Seoul / Tokyo (+ SG) | Thimphu today | Your product fit |
|---|---|---|---|
| **Operator** | Strong municipal / national utility | Thromde + project packages | Private/building layer + optional Thromde partner |
| **Control room** | 24/7 SCADA centers | Local SCADA on new plants/schemes | Cloud dashboard + push/SMS (lighter) |
| **Sensor density** | Citywide pressure/flow + plants + meters | Sparse; growing on new lines | LoRaWAN corridor you can afford |
| **Smart meters** | Mass / multi-year full coverage | Hundreds–pilots | Building inlet + tank first |
| **AI / twin** | Active roadmap | Not the priority yet | Rules-based alerts first; AI later |
| **Who gets alert** | Utility staff → then citizen services | Mostly utility / site staff | **Building owner** from source stage |
| **CCTV** | Plants, some assets / drainage | Limited / site-specific | Source + pump house clips |
| **Money / rent** | Separate billing utilities | Thromde tariffs ≠ building rent | Your anti-skimming rent rail |
| **Capex** | Billions / multi-decade | ADB + RGoB project finance | Startup + solar LoRa nodes |

```text
Seoul/Tokyo:  SOURCE ════════════════════════ CITY GRID ════════ HOUSE METER
              (dense SCADA)                 (dense SCADA)      (smart meter)

Thimphu:      SOURCE ─── some SCADA ─── zones ─── ??? ─── BUILDING TANK
                                              ↑
                                    visibility gap (your wedge)

Your MVP:     SOURCE node ── LoRa path ── OWNER TANK + alerts to owner
```

---

## 7. What this means for Thimphu source strategy

1. **Do not try to clone Tokyo’s control center** on day one — wrong cost and mandate.  
2. **Do copy their logic:** source + pressure/flow anomalies → **automatic warning** (Tokyo already does this in ops).  
3. **Do copy Seoul’s citizen notify idea** — but target **building owners** (and optional residents) for **supply disruption from source**, not only indoor plumbing leaks.  
4. **Coexist with Thromde SCADA** on new WTPs: your system covers the **last mile + owner tank + source-path alerts** Thromde may not push to private owners.  
5. **LoRaWAN** is the right *affordable* density tech for Thimphu hills; Seoul/Tokyo often use utility fiber / dedicated telemetry / cellular AMI — different budget.

---

## 8. Realistic ambition ladder

| Level | Like… | Thimphu path |
|---|---|---|
| L1 | Building tank + inlet + push | Your MVP |
| L2 | Source/zone node + path alerts + CCTV clip | Your “from source” plan |
| L3 | Multi-building corridor + leak between nodes | Expand LoRaWAN |
| L4 | Data share / API with Thromde SCADA | Partnership |
| L5 | City DMA + AMI everywhere | Thromde / national program (Seoul/Tokyo scale) |

---

## 9. Bottom line

**Seoul & Tokyo** run **utility-owned, dense, 24/7 SCADA + expanding smart meters + AI roadmaps** — state/city systems of record for *public* water.

**Thimphu** is **building toward that** scheme-by-scheme (SCADA on new WTPs, AMR pilots) but still has a large **visibility gap** from municipal source to **private building tanks**, and almost no **owner-facing source alerts**.

**Your wedge:** affordable **source → tank** sensing (LoRaWAN) + **notifications to the building owner** — the layer mega-cities eventually cover with AMI/apps, but Thimphu owners need *now*, without waiting for a full Arisu/Tokyo-class grid.
