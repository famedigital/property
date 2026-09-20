# Velora — Architecture Plan

**Brand:** [`BRAND.md`](./BRAND.md) — *Property, settled.*  
**Status:** Proposed (plan only — no implementation yet)  
**Date:** 2026-08-30  
**Goal:** Multi-platform **building-first** property ops: **unit listing / vacancies / photos (+ optional 3D) / pay security → secured lease**, **escrow rent**, **cleaning with biometric/CCTV proof**, **per-building water IoT**. Pamtsho WTP ERP later. No owner roster to RRCO.

**Related:**
- **Brand** → [`BRAND.md`](./BRAND.md) / [`.html`](./BRAND.html)
- **Focus & phasing** → [`PROJECT_FOCUS.md`](./PROJECT_FOCUS.md)
- **Listing & leasing funnel** → [`LISTING_AND_LEASING.md`](./LISTING_AND_LEASING.md)
- Escrow + ops → [`SYSTEMS_AND_OPERATIONS.md`](./SYSTEMS_AND_OPERATIONS.md)
- **Cleaning biometric/CCTV** → [`CLEANING_OPS.md`](./CLEANING_OPS.md)
- **Pamtsho WTP ERP (later)** → [`PAMTSHO_WTP_ERP_BLUEPRINT.md`](./PAMTSHO_WTP_ERP_BLUEPRINT.md)
- Building / corridor LoRaWAN → [`THIMPHU_LORAWAN_PIPELINE.md`](./THIMPHU_LORAWAN_PIPELINE.md)
- Water alerts → [`WATER_SOURCE_ALERTS.md`](./WATER_SOURCE_ALERTS.md)
- City comparison → [`CITY_WATER_MONITORING_COMPARISON.md`](./CITY_WATER_MONITORING_COMPARISON.md)
- **Full AI dashboards & clients** → [`AI_AND_API_KEYS.md`](./AI_AND_API_KEYS.md)
- **UI look · page inventory · needs** → [`UI_PAGES_AND_LOOK.md`](./UI_PAGES_AND_LOOK.md) / [`.html`](./UI_PAGES_AND_LOOK.html) (Darkmatter, ~52 screens)
- **Visual plan** → [`COMPLETE_SYSTEM_PLAN.html`](./COMPLETE_SYSTEM_PLAN.html)

---

## 1. What global PMS products teach us

Leading platforms (AppFolio, Yardi, Entrata, RealPage, Buildium, Facilio, ODIN) converge on the same product spine:

| Capability | Why it matters for this product |
|---|---|
| Portfolio → property → building → floor → unit hierarchy | Core data model |
| Roles: owner, PM, staff, resident, vendor | RBAC must be first-class |
| Maintenance / work orders | Water alerts must create tickets |
| Tenant / resident portal | Mobile + web for residents |
| Accounting / leasing (later) | Phase 2+; do not block MVP |
| Open APIs + IoT / BMS connectors | Sensors are not a bolt-on |
| Realtime alerts + condition-based maintenance | Facilio / ODIN pattern |

**PropTech data standards (adopt conceptually, not full RDF graphs on day 1):**

- **RealEstateCore** — spaces, leases, portfolio (buildings, floors, units)
- **Brick Schema** — equipment + sensors (water tanks, level points)
- **Project Haystack / ASHRAE 223P** — optional later for BMS interoperability

**Product positioning:** **Building-first PMS.** Vacancy → list → photos (+ optional 3D) → pay security in escrow → secured lease; then monthly escrow rent, cleaning proof, building water IoT. Pamtsho municipal ERP later.

---

## 2. Recommended product scope (MVP → later)

### MVP — NOW (individual buildings)
- Orgs, buildings, **units + vacancy board**  
- **Listings** with **standard photo slots** (+ optional 3D/360 embed)  
- Applications → **pay security deposit into escrow** → **secured** lease  
- Ongoing **escrow rent** + settlements (**no RRCO owner roster**)  
- **Cleaning** + biometric/CCTV proof  
- **Water IoT per building** + alerts  
- Work orders; admin web + mobile  

### Later
- In-app 3D capture SDK, e-sign, waitlists  
- Main source / **Pamtsho WTP complete ERP** for Thromde  
- Cleaning payroll from verified minutes

---

## 3. Full-stack recommendation (decision)

### Verdict

| Layer | Choice | Why |
|---|---|---|
| Monorepo | **pnpm + Turborepo** | Shared types, one CI, Expo + Next together |
| Web | **Next.js (App Router)** | SSR admin, API routes / server actions, Vercel-ready |
| Mobile (iOS + Android) | **Expo (React Native)** | One codebase, EAS builds, OTA updates |
| API | **tRPC or Hono on Next** (MVP); extract Nest/Fastify later if needed | Fast TypeScript end-to-end |
| Auth | **Supabase Auth** (or Clerk if you prefer managed UX) | JWT + RLS-friendly |
| Primary DB | **PostgreSQL 16+ with RLS** | RBAC + multi-tenant isolation in DB |
| Telemetry store | **Same Postgres + TimescaleDB extension** (or plain hypertables via partitioning if Timescale not available) | Join sensors ↔ units in SQL; one ops surface |
| Cache / pubsub | **Redis** (Upstash or self-hosted) | Presence, rate limits, alert fan-out |
| Device protocol | **LoRaWAN (sensors) + MQTT bridge into cloud**; Wi‑Fi/4G only where needed | Long-range Thimphu hills; CCTV on separate 4G/fiber |
| App realtime | **WebSockets / Supabase Realtime / SSE** | Push level updates to apps |
| ORM / migrations | **Drizzle ORM** | Typed schema shared with mobile types |
| Object storage | **S3-compatible** (Cloudflare R2 / MinIO / AWS S3) | Photos, docs, firmware |
| Maps | Mapbox or Google Maps | Building location |
| Push notifications | Expo Push + FCM/APNs | Critical water alerts |
| AI | **Full control plane** — Vercel AI SDK + Gateway; dashboard + client copilots with RBAC tools | Escrow/biometric/sensors stay source of truth |
| Observability | OpenTelemetry + Grafana | Device + API health |
| CI/CD | GitHub Actions + EAS + deploy hooks | Code management |

**Not recommended for MVP:** separate microservices per domain, Kafka, dual Influx+Postgres (ops cost), Flutter + separate web stack (duplicates work), Mongo as primary (weak relational RBAC).

### Repo shape

```text
apps/
  web/                 # Next.js admin + resident web
  mobile/              # Expo iOS/Android
  mqtt-bridge/         # MQTT → DB / Redis / push
packages/
  db/                  # Drizzle schema, migrations, RLS SQL
  shared/              # Zod schemas, roles, domain types
  ui/                  # shared design tokens (optional)
infra/
  docker-compose.yml   # local Postgres, Redis, EMQX, MinIO
  terraform/ or pulumi/ # cloud (optional)
docs/
  ARCHITECTURE.md      # this file
```

---

## 4. Database decision (systematic)

### Primary: PostgreSQL + Row Level Security (RBAC in DB)

**Model:** shared schema, multi-tenant SaaS with `org_id` on every tenant-scoped table. Isolation enforced by **Postgres RLS**, not only app `WHERE` clauses.

**Why Postgres wins here**
- Property graphs are relational (org → property → building → unit → lease → sensor)
- RBAC + RLS is mature and used by AWS / Supabase patterns
- Timescale hypertables keep water readings in the same engine → cheap joins
- One backup / one migration story

**Avoid for core data:** MongoDB, DynamoDB-only, Firebase as system of record.

### RBAC model (in DB)

Roles (examples):

| Role | Typical access |
|---|---|
| `platform_admin` | All orgs (support) |
| `org_owner` | Full org |
| `property_manager` | Assigned properties |
| `maintenance_staff` | Work orders + sensors on assigned sites |
| `resident` | Own unit + shared building tanks (read) |
| `viewer` | Read-only dashboards |

Tables (sketch):

```text
orgs
users
memberships          (user_id, org_id, role)
properties
buildings
floors
units
unit_occupancies
assets               (tanks, pumps — Brick-like equipment)
sensors
sensor_readings      (time-series / hypertable)
alerts
work_orders
audit_logs
permissions / role_permissions  (optional fine-grained)
```

**RLS pattern**
1. App sets `SET LOCAL app.user_id` / `app.org_id` per transaction
2. Policies: row visible if membership exists AND role allows resource
3. `FORCE ROW LEVEL SECURITY` on all tenant tables
4. Index leading columns `(org_id, …)` so RLS stays cheap
5. Sensor write path uses a **device credential** role with narrow insert policy on `sensor_readings`

### Telemetry: TimescaleDB (preferred) or partitioned Postgres

Water level is low-cardinality time series (one series per sensor). **TimescaleDB on Postgres** is the best fit:

- Hypertables for `sensor_readings(time, sensor_id, level_pct, liters, battery, rssi)`
- Continuous aggregates (1m / 5m / 1h rollups)
- Retention: raw 30–90 days; rollups 1–2 years
- Alerts query latest + thresholds without a second database

If managed Timescale is too expensive: use **native Postgres partitioning by month** + compressed archives to object storage. Skip InfluxDB unless you later hit millions of high-frequency series.

---

## 5. Water sensor realtime vs cost

### Requirement
“Instant” for water = **seconds**, not sub-millisecond industrial control. MQTT + push is enough.

### Tier A — True near-realtime (recommended when online is reliable)

```text
Sensor/ESP32 → MQTT (QoS 1) → mqtt-bridge →
  1) write sensor_readings + update sensors.last_*
  2) if threshold crossed → alerts + Expo push
  3) publish to Redis / Realtime channel → apps
```

- Sample every 30–60s normally; **event on delta ≥ 2% or pump on/off**
- Critical low: immediate publish + push
- Cost: EMQX/Mosquitto on a small VPS is inexpensive; avoid AWS IoT Core early (pricing surprises)

### Tier B — Cost-saver: **Edge local store + online sync** (recommended default for many buildings)

When cellular/Wi-Fi is expensive or unreliable (common for tanks on rooftops):

```text
Building gateway (Raspberry Pi / ESP32-S3 gateway)
  - local SQLite / LevelDB seed of last N days
  - local MQTT broker for sensors on LAN
  - rule engine: local siren / SMS modem on critical
  - sync when online:
      • batch upload readings (compressed)
      • pull config / thresholds / RBAC device certs
      • conflict: server wins for config; device wins for readings (append-only)
```

**When to prefer Tier B**
- High cloud IoT / egress cost
- Intermittent connectivity
- Need on-site alerts even if internet is down

**Hybrid (best product story):** Tier B gateway for reliability + Tier A cloud for portfolio dashboards. Local is source of truth for last-known; cloud is source of truth for org config and multi-site analytics.

### What not to do
- HTTP poll every 1s from phones (kills battery and cost)
- Store every 1 Hz reading forever in hot Postgres
- Full Kafka for <10k sensors

---

## 6. Deployment & cloud cost strategy

### Recommended path: **cheap core + optional AWS**

| Environment | Recommendation | Approx early monthly |
|---|---|---|
| **MVP / cost-first** | **Hetzner** (API + Postgres + Redis + EMQX) + **Cloudflare** (CDN, R2, DNS, Tunnel) + **Expo EAS** | Often **~$25–80** for small fleets |
| **Managed DX** | **Supabase** (Auth, Postgres, Realtime, Storage) + **Vercel** (Next.js) + **Upstash Redis** + small VPS for MQTT | **~$40–150** |
| **Enterprise / compliance** | **AWS**: ECS/Fargate or App Runner, RDS Postgres, ElastiCache, S3, ALB; optionally IoT Core later | Higher; use when contracts demand it |

**Decision rule**
1. Start on **Supabase + Vercel + Hetzner MQTT** for speed + RBAC RLS + low ops.
2. Keep Docker Compose so you can move API/DB to Hetzner or AWS without rewrite.
3. Move to AWS only for enterprise SSO, multi-region, or procurement requirements.

**Cheaper than AWS alternatives worth using:** Hetzner, Cloudflare R2 (egress), Fly.io/Railway for API if you want PaaS without AWS complexity.

### AWS-shaped reference (if required later)

- API: ECS Fargate or App Runner  
- DB: RDS PostgreSQL (+ Timescale AMI/extension where licensed)  
- Cache: ElastiCache Redis  
- Files: S3  
- MQTT: Amazon MQ (MQTT) or self-managed EMQX on EC2  
- Edge: IoT Core only at scale  
- CDN: CloudFront  

---

## 7. MCP + code management connections

Available / planned agent integrations for this repo:

| System | Use |
|---|---|
| **GitHub** | Source of truth, PRs, Actions CI, branch protection |
| **Supabase MCP** | Migrations, advisors, SQL, edge functions, types (already wired in agent environment) |
| **Vercel MCP** | Deploy web, env, logs (authenticate when ready) |
| **Cursor Cloud MCP** | Environment builds, run diagnostics |
| **Expo EAS** | iOS/Android build + submit (CLI in CI) |
| **Sentry / Grafana** | Errors + device metrics |

**Code management practices**
- Trunk: `main` protected; feature branches `cursor/<name>-****`
- Migrations only via versioned SQL (Drizzle Kit) — never dashboard-only schema drift
- Environments: `local` → `preview` → `staging` → `production`
- Secrets in Vercel/Supabase/EAS; never commit `.env`
- Device credentials rotated; MQTT ACL per `org_id/device_id`

---

## 8. Security checklist (non-negotiable)

- RLS on every tenant table; service role only on server
- Device auth: mutual TLS or signed JWT per sensor/gateway; no shared global keys
- Least privilege MQTT topics: `org/{orgId}/building/{id}/sensor/{id}/telemetry`
- Audit log for RBAC changes and alert acknowledgements
- PII minimization for residents; encrypt backups
- Rate-limit sensor ingest; reject out-of-range levels

---

## 9. Implementation phases (technical, not calendar)

1. **Foundation:** monorepo, Postgres schema + RLS RBAC, Auth, seed data  
2. **Property CRUD:** orgs, buildings, units, memberships — web admin  
3. **Sensors:** registry, MQTT bridge, readings hypertable, live web dashboard  
4. **Mobile:** Expo app for staff/residents + push alerts  
5. **Edge sync (Tier B):** gateway local DB + batch sync  
6. **Ops:** work orders from alerts, audit, Grafana  
7. **Hardening:** multi-region backup, AWS path if needed, PMS/BMS connectors  

---

## 10. Open decisions for you to confirm

1. **Cloud preference:** Supabase+Vercel+Hetzner MQTT (recommended) vs full AWS from day one?  
2. **Sensor hardware:** ESP32 ultrasonic / pressure transducers / vendor Modbus gateways?  
3. **Geography / data residency:** EU (Hetzner/FI) vs US vs multi-region?  
4. **MVP users:** only building managers, or residents in v1?  
5. **Billing:** single-org product or multi-tenant SaaS from the start? (schema assumes SaaS)

---

## 11. Summary recommendation

Build a **TypeScript monorepo (Next.js + Expo)** on **PostgreSQL with RLS RBAC**, store water telemetry in **Timescale/partitioned Postgres**, ingest via **MQTT**, and deploy initially on **Supabase + Vercel + cheap VPS for MQTT** (or Hetzner for all compute). Use **building gateways with local seed + online sync** where connectivity or cloud cost is the constraint. Align the domain model with **RealEstateCore (spaces) + Brick (sensors)** without requiring a full ontology stack in MVP.
