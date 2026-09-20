# Systems & Operations — Full Picture

**Product:** Property management (buildings / units / flats) + water sensors + **escrow rent collection**  
**Money rule:** Resident pays into a **platform escrow bank account**; platform settles to the building owner on a schedule (PM fee + SaaS fee explicit).  
**RRCO rule:** **Do not share rental owner roster / PII with RRCO.** Owner KYC stays internal; residents get money receipts; owners get settlement statements for their own PIT filing.  
**Also:** Building **cleaning ops** with biometric + CCTV proof — [`CLEANING_OPS.md`](./CLEANING_OPS.md).  
**Phasing:** [`PROJECT_FOCUS.md`](./PROJECT_FOCUS.md) — building IoT first; Pamtsho main source later.  
**Status:** Plan — systems & ops blueprint

---

## 1. Business model (one page)

```text
                    ┌─────────────────────────────┐
                    │   YOUR PLATFORM (SaaS+IoT)   │
                    │  Software · Alerts · Rent UI │
                    │  Fee: subscription ± % fee   │
                    │  Holds ESCROW bank account   │
                    └─────────────┬───────────────┘
                                  │
         ┌────────────────────────┼────────────────────────┐
         ▼                        ▼                        ▼
   Building Owner           Property Manager          Residents
   (receives settlement)    (ops only — no rent     (pay into ESCROW,
   bank KYC internal)       wallet)                  see water, tickets)
         ▲
         │ scheduled payout (not via RRCO)
   Escrow ledger ← Payment rails into ONE merchant/escrow account
```

| Who | Pays / receives | For what |
|---|---|---|
| **Resident** | Pays rent into **escrow** | Occupancy of unit/flat |
| **Building owner** | **Receives settlement** from escrow (net of PM/platform fees) | Ownership; KYC only inside our DB |
| **Property manager** | Visible management fee from escrow split | Operating the building — never holds rent |
| **Your platform** | Subscription ± fee; operates escrow | Software, sensors, collection rails |
| **Payment provider** | Card/bank fees into platform merchant | Processing |
| **RRCO** | **No owner roster export from us** | Owners self-file PIT with their statements |

**Critical legal/ops rule:** Platform (or escrow legal entity) is **collector of record** into escrow, then remits to owners. Platform does not treat rent as SaaS revenue. **Owner directory is never exported to RRCO.** Recommend local counsel for TDS/PIT; this is a data-boundary design, not tax advice.

---

## 1b. Why this exists: stop “manager eating the money”

This is the #1 ops failure for many buildings today:

| Old (broken) flow | What goes wrong |
|---|---|
| Resident pays **cash/transfer to manager** | Manager under-reports, delays remittance, invents “vacancies”, keeps change |
| Owner only sees what manager *says* was collected | Owner cannot prove arrears vs theft |
| Manual Excel / WhatsApp receipts | Easy to fake, delete, or alter |
| Manager “holds” repairs money | Inflated vendor costs, no proof |

### Product rule (non-negotiable)

```text
Resident MUST pay into PLATFORM ESCROW (not manager personal account)
        → escrow ledger: held → settled
        → scheduled payout to BUILDING OWNER bank (internal KYC)
Manager NEVER receives rent into personal wallet/bank as the default path
Manager fee is a VISIBLE escrow split — never “whatever is left”
Owner sees expected vs collected vs outstanding in real time — independent of manager
NO owner roster / rent roll API or file to RRCO
```

### Hard controls in software

| Control | How it works |
|---|---|
| **Escrow only** | Card/bank/USSD → **one** platform merchant/escrow account |
| **No silent “mark paid”** | Manager cannot clear an invoice without a payment webhook **or** owner-approved cash exception |
| **Cash exception (rare)** | Log cash → photo proof + **owner OTP/approve** → then marked paid; still auditable |
| **Immutable ledger** | `escrow_ledger_entries` cannot be deleted; void/refund with reason + actor |
| **Expected rent board** | Every active lease auto-invoices; vacancies are owner-visible |
| **Manager fee capped** | Config: fixed % or flat; shown on every receipt |
| **Dual visibility** | Owner + resident both see same receipt / escrow payment id |
| **Arrears alerts to owner** | Overdue notifies owner directly — not only manager |
| **RRCO boundary** | No export of owner PII, unit roster, or rent rolls to RRCO |
| **Audit log** | Who changed lease, rent, vacancy, fee — forever |
| **Bank mismatch report** | Escrow inflows vs owner payouts — weekly |

### What the manager *is* allowed to do

- Chase overdue residents (reminders)
- Log maintenance / water issues
- Propose rent/lease changes (**owner approves**)
- Record *owner-approved* cash exceptions
- Earn a **transparent management fee**

### What the manager must *never* control alone

- Destination bank for rent
- Deleting payments
- Declaring vacancy without owner visibility
- Changing rent mid-cycle without audit
- Holding “repair float” without work-order + receipt

### Immediate ops change (even before full app)

1. Residents pay **only** via official owner/platform link — never manager’s personal account
2. “I gave it to the manager” is **not** proof of payment
3. Owner reviews weekly: invoiced − paid − outstanding
4. Pay manager a clear fee (e.g. %) — not leftover cash

---

## 2. Organogram (customer side — who the app serves)

```text
                         BUILDING OWNER / SOCIETY BOARD
                         (legal recipient of rent)
                                    │
                    ┌───────────────┴───────────────┐
                    │     Property Management Org     │
                    │   (may be owner or hired firm)  │
                    └───────────────┬───────────────┘
                                    │
        ┌───────────────┬───────────┼───────────┬───────────────┐
        │               │           │           │               │
   Org Admin      Property      Facilities   Accounts/      Site
   (RBAC,         Manager       Head         Rent Clerk     Supervisor
    setup)        (ops)         (water+       (invoices,     (on ground)
                                 maintenance)  receipts)
        │               │           │           │               │
        └───────┬───────┴─────┬─────┴─────┬─────┴───────┬───────┘
                │             │           │             │
           Maintenance    Water/IoT    Vendors      RESIDENTS /
              Staff        Gateway     (plumber,     TENANTS
                           (device)    tanker)       (pay + report)
```

### Role → function matrix

| Role | Core function | App modules |
|---|---|---|
| **Building owner** | Owns asset; receives rent; sees portfolio & water risk | Owner dashboard, payouts, reports |
| **Org admin** | Creates buildings, units, users, roles, payment KYC link | Admin, RBAC, settings |
| **Property manager** | Occupancy, tickets, chase rent — **does not hold rent money** | Leases, units, work orders, reminders (no payout bank) |
| **Accounts / rent clerk** | Issue invoices, chase arrears, reconcile | Rent ledger, receipts, dunning |
| **Facilities head** | Water tanks, pumps, vendors, SLAs | Sensors, alerts, assets |
| **Maintenance staff** | Execute work orders | Mobile work orders |
| **Site supervisor** | Acknowledge alerts, dispatch tanker | Alerts, local ops |
| **Resident** | Pay rent, see dues, see shared tank, raise issue | Resident app |
| **Vendor** | Complete assigned jobs | Limited ticket portal |
| **Platform support (you)** | Onboard orgs, fix sensors, billing for SaaS | Internal tools |

---

## 3. Your company organogram (vendor)

```text
CEO / Founder
 ├── Product
 ├── Engineering (Web · Mobile · Backend · MQTT/IoT)
 ├── Payments & Compliance (KYC, payouts, chargebacks)
 ├── Customer Success / Onboarding
 ├── Field IoT (install/calibrate tanks)
 ├── Sales / Partnerships (societies, PM firms, hardware)
 └── Support
```

---

## 4. End-to-end systems map

```text
┌──────────────────────────────────────────────────────────────────┐
│                         CLIENT APPS                              │
│  Web (Next.js)     Mobile (Expo or Flutter / native)             │
│  Owner · Admin · Manager · Accounts · Resident                   │
└─────────────┬───────────────────────────────┬────────────────────┘
              │ HTTPS / tRPC                   │ Push (FCM/APNs)
              ▼                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                      APPLICATION PLATFORM                        │
│  Auth (Supabase/Clerk)  ·  API  ·  RBAC context  ·  Notifications│
│  Rent engine  ·  Work-order engine  ·  Alert rules               │
└───────┬───────────────────┬───────────────────┬──────────────────┘
        │                   │                   │
        ▼                   ▼                   ▼
┌───────────────┐   ┌───────────────┐   ┌──────────────────────────┐
│  PostgreSQL   │   │ Redis         │   │ Payment provider         │
│  + RLS RBAC   │   │ cache/pubsub  │   │ Local rails → ESCROW     │
│  + readings   │   │               │   │ escrow_ledger + payouts  │
│  (Timescale)  │   │               │   │ Webhooks → ledger        │
│  No RRCO sync │   │               │   │ Owner KYC internal only  │
└───────▲───────┘   └───────────────┘   └──────────────────────────┘
        │
        │ writes readings / last_level
        │
┌───────┴──────────────────────────────────────────────────────────┐
│                     IoT PLANE                                    │
│  Sensors → MQTT (EMQX) → mqtt-bridge                             │
│  Optional building gateway: local store + online sync            │
└──────────────────────────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────────────────────────┐
│  Object storage (R2/S3) · Email/SMS · Expo Push · Grafana        │
└──────────────────────────────────────────────────────────────────┘
```

### Core data domains

| Domain | Entities | Purpose |
|---|---|---|
| **Portfolio** | org, property, building, floor, unit | Physical structure |
| **People** | users, memberships, roles | RBAC |
| **Leasing / listing** | unit status, listings, media, applications, handover | Vacancy → list → secure |
| **Rent / escrow** | invoice, escrow_*, owner_payouts, security deposits | Escrow money |
| **IoT** | asset (tank), sensor, reading, alert | Water ops |
| **Ops** | work_order, vendor, SLA | Maintenance |
| **Audit** | audit_log, webhook_event | Compliance |

---

## 5. Rent collection — escrow money flow (owner receives settlement)

### Chosen pattern: **Platform escrow account**

```text
Resident pays invoice (card / bank / USSD / etc.)
        │
        ▼
Payment rail → PLATFORM ESCROW bank (merchant of record)
        │
        ├─ ledger: payment held (invoice paid / escrow_held)
        ├─ split recorded: owner_amount, pm_fee, platform_fee
        │
        ▼
Scheduled settlement job (e.g. T+1 or weekly)
        ├──► Owner bank payout          ≈  owner_amount
        ├──► PM bank (optional)         ≈  management fee
        └──► Platform SaaS fee          ≈  collection fee

RRCO ◄── X ── no owner roster / rent-roll feed from platform
Owner ──self-files PIT──► RRCO (using settlement PDF + receipts)
```

**Platform never treats escrow rent as SaaS revenue.** Tables:

- `escrow_accounts` — platform escrow bank metadata  
- `invoices` — owed by resident  
- `escrow_ledger_entries` — inflows, holds, fees, voids  
- `owner_payouts` — settlement batches to owner banks  
- `receipt` — resident money receipt; owner settlement statement  

### RRCO data boundary

| Allowed | Not allowed |
|---|---|
| Money receipt to resident | Bulk export of landlords to RRCO |
| Settlement PDF to owner for their PIT | Sharing unit-by-unit rent rolls with RRCO |
| Aggregate platform accounting (our books) | Payment-provider KYC that forces public owner directory to tax office via our APIs |

### Operational rent cycle

```text
1. Lease active on unit
2. Cron: generate monthly invoice
3. Notify resident
4. Resident pays → funds hit escrow → webhook → ledger held
5. Settlement job: payout to owner bank from internal KYC
6. Unpaid → dunning → arrears → PM workflow (owner still sees board)
7. Partial / dispute / refund → accounts + audit
```

### Owner onboarding (KYC) — internal only

1. Admin links **building → legal owner entity**  
2. Owner completes **internal** KYC (ID, bank) stored encrypted under RLS  
3. Status `payouts_enabled` → escrow collection unlocked  
4. Without KYC: offline/manual invoice logging only  
5. **No step** that uploads owner list to RRCO  

### Regional payment note

| Region | Typical rails into escrow |
|---|---|
| Bhutan | Local bank / mobile rails into **one** platform merchant account |
| Elsewhere | Same pattern: single merchant → escrow ledger → owner payouts |
| Always | `PaymentsPort` abstraction; prefer escrow over per-owner Connect |

---

## 6. Operations playbooks (full picture)

### A) Building onboarding

1. Create org → invite owner + admin  
2. Add building, floors, units  
3. Attach owner legal entity + payment KYC  
4. Map tanks/sensors; set low/critical thresholds  
5. Import or create leases + residents  
6. Go-live checklist: first invoice dry-run, sensor heartbeat OK  

### B) Listing → security → secured (standard PMS)

1. Unit `vacant` (after move-out / cleaning)  
2. Create **listing** + **standard photo slots** (+ optional 3D URL)  
3. Prospect inquires / applies  
4. PM/Owner approves → `reserved`  
5. Prospect **pays security into escrow** → lease created → unit **secured**  
6. Handover checklist; then monthly escrow rent  

See [`LISTING_AND_LEASING.md`](./LISTING_AND_LEASING.md).

### B2) Move-in / lease (legacy short form)

1. Unit marked vacant → create lease (or via listing funnel above)  
2. Resident user invited; **security deposit** recorded on escrow ledger  
3. Access to resident app; rent schedule starts  

### C) Monthly rent ops

1. Auto-generate invoices (D-3 before due)  
2. Autopay if resident opted in  
3. Accounts reviews failures / arrears board  
4. Owner sees “collected vs outstanding” per building  

### D) Water / sensor ops

1. Sensor publishes level via MQTT (or gateway sync)  
2. Rules: `% < low` → alert PM + facilities; `% < critical` → push + optional SMS + auto work order  
3. Staff acknowledge → dispatch pump/tanker vendor  
4. Resolve → close WO; reading returns above threshold  

### D2) Cleaning ops

1. Schedule job for zone/unit  
2. Worker **biometric check-in** + CCTV clip (or QR+geo selfie)  
3. Work + optional checklist photos  
4. **Biometric check-out** + proof  
5. Below min duration or missing proof → failed / no-show alert to PM (owner optional)  

See [`CLEANING_OPS.md`](./CLEANING_OPS.md).

### E) Maintenance ops

1. Source: resident report **or** sensor alert **or** scheduled PPM  
2. Work order created with priority, unit/building, assignee  
3. Vendor/staff complete + photo proof  
4. Owner/PM can see cost notes (optional)  

### F) Move-out

1. Final invoice + deposit settlement  
2. Lease end; unit vacant  
3. Revoke resident access; keep historical ledger  

### G) Incident / escalation

| Severity | Example | Who acts | Channel |
|---|---|---|---|
| P1 | Tank empty / pump fail / payment outage | Facilities + PM | Push + SMS |
| P2 | Rent overdue 7+ days | Accounts + PM | In-app + email |
| P3 | Sensor offline 24h | Facilities | App alert |
| P4 | Cosmetic ticket | Staff | Queue |

---

## 7. RBAC (operations × money × water)

| Capability | Owner | Admin | PM | Accounts | Facilities | Staff | Resident |
|---|---|---|---|---|---|---|---|
| Manage buildings/units | R | RW | RW* | R | R | — | — |
| Manage users/roles | R | RW | — | — | — | — | — |
| Leases | R | RW | RW | R | — | — | own R |
| Create invoices | R | RW | R | RW | — | — | — |
| Pay rent | — | — | — | — | — | — | RW |
| See payout / owner money | RW | R | R† | R | — | — | — |
| Sensors / alerts | R | RW | RW | — | RW | R | shared R |
| Work orders | R | RW | RW | — | RW | RW assigned | create |

\* assigned properties only · † may hide bank details  

Enforced in **Postgres RLS** + app checks; payment webhooks use service role with audited writes only.

---

## 8. System of record vs system of engagement

| Concern | System of record | Engagement |
|---|---|---|
| Who lives where | `leases`, `units` | Resident app home |
| What is owed | `invoices`, `ledger` | Pay button, reminders |
| Where money went | Payment webhooks + `payments` | Owner payout dashboard |
| Tank level now | `sensors.last_*` + readings | Live gauge, alerts |
| Work done | `work_orders` | Mobile checklist |

**Offline / edge:** gateway keeps local sensor seed; rent **always requires online** for card/bank pay (cash payments can be logged offline by accounts and synced).

---

## 9. Daily / weekly / monthly ops rhythm

| Cadence | Owner | PM | Accounts | Facilities | Platform (you) |
|---|---|---|---|---|---|
| **Daily** | Glance alerts & collections | Clear P1/P2 tickets | Failed payments | Sensor health | Uptime / support |
| **Weekly** | Arrears & water risk review | Unit vacancy | Reconciliation | Vendor scorecards | CS check-ins |
| **Monthly** | Payout statement | Occupancy report | Close rent period | PPM schedule | SaaS invoice |
| **Quarterly** | Portfolio review | Lease renewals | Audit sample | Calibrate tanks | Roadmap |

---

## 10. Compliance & risk (rent + IoT)

- Owner KYC (internal) before live escrow collection  
- Clear fee disclosure (platform fee + processor fee)  
- Receipts and audit logs immutable  
- Chargebacks: freeze disputed invoice; owner notified  
- **Escrow / client money** account at bank; recommend local counsel for Bhutan rules  
- **No owner roster to RRCO**; owners self-file PIT with settlement PDFs  
- Sensor data is operational, not payment PII — still org-scoped via RLS  

---

## 11. MVP vs later (updated with rent)

### MVP (systems that must work together)
- Portfolio + vacancy board + **listings / photos / optional 3D**  
- Applications + **pay security → secured lease**  
- Escrow rent + cleaning biometric/CCTV proof + building water IoT  
- Web admin + mobile for prospect browse/pay + staff alerts  

### Later
- Autopay; cleaning payroll from verified minutes; in-app 3D capture  
- **Pamtsho / main-source WTP ERP** for Thromde  
- Multi-owner splits on one building  

---

## 12. One-picture summary

```text
RESIDENT ──pays──► ESCROW BANK ──settlement──► BUILDING OWNER
                       │
                       ├── PM fee ──► Manager
                       └── SaaS fee ──► PLATFORM

RRCO ◄── X ── no owner roster from platform
Owner ──self-files PIT──► RRCO

SENSORS ──MQTT──► PLATFORM ──alerts──► FACILITIES / PM ──WO──► STAFF/VENDOR
```

**Business in one sentence:** You run escrow rent + property ops for owners, and a separate **Pamtsho WTP ERP** for Thromde; rent money settles to owners without exposing their roster to RRCO.
