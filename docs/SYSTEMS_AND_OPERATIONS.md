# Systems & Operations — Full Picture

**Product:** Property management (buildings / units / flats) + water sensors + **rent collection**  
**Money rule:** Rent is paid by the resident and **settled to the building owner** (platform may take a small SaaS/processing fee only).  
**Status:** Plan — systems & ops blueprint

---

## 1. Business model (one page)

```text
                    ┌─────────────────────────────┐
                    │   YOUR PLATFORM (SaaS+IoT)   │
                    │  Software · Alerts · Rent UI │
                    │  Fee: subscription ± % fee   │
                    └─────────────┬───────────────┘
                                  │
         ┌────────────────────────┼────────────────────────┐
         ▼                        ▼                        ▼
   Building Owner           Property Manager          Residents
   (receives RENT)          (runs day-to-day)         (pay rent,
   connects bank            staff, vendors            see water,
   via payments KYC)                                  raise tickets)
         ▲
         │  payout
   Payment Provider (Stripe Connect / Paystack / Flutterwave / etc.)
```

| Who | Pays / receives | For what |
|---|---|---|
| **Resident** | Pays rent (+ optional utilities) | Occupancy of unit/flat |
| **Building owner** | **Receives rent** (net of optional PM fee / platform fee) | Ownership of building |
| **Property manager** (if separate) | May receive **management fee** split from rent or billed separately | Operating the building |
| **Your platform** | Subscription + optional small payment fee | Software, sensors, collection rails |
| **Payment provider** | Card/bank fees | Processing |

**Critical legal/ops rule:** Platform is a **facilitator**, not the landlord. Owner completes KYC on a **connected account**. Funds route **to owner** (destination charge / split). Platform does not “own” tenant rent balances as company revenue.

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
Resident MUST pay into the platform payment rail
        → money settles to BUILDING OWNER account
Manager NEVER receives rent into personal wallet/bank as the default path
Manager fee is a VISIBLE split (or separate invoice) — never “whatever is left”
Owner sees expected vs collected vs outstanding in real time — independent of manager
```

### Hard controls in software

| Control | How it works |
|---|---|
| **Pay-to-owner only** | Card/bank/USSD checkout → owner connected account |
| **No silent “mark paid”** | Manager cannot clear an invoice without a payment webhook **or** owner-approved cash exception |
| **Cash exception (rare)** | Log cash → photo proof + **owner OTP/approve** → then marked paid; still auditable |
| **Immutable ledger** | Payments cannot be deleted; only void/refund with reason + actor |
| **Expected rent board** | Every active lease auto-invoices; vacancies are owner-visible |
| **Manager fee capped** | Config: fixed % or flat; shown on every receipt |
| **Dual visibility** | Owner + resident both see same provider receipt ID |
| **Arrears alerts to owner** | Overdue notifies owner directly — not only manager |
| **Audit log** | Who changed lease, rent, vacancy, fee — forever |
| **Bank mismatch report** | Owner payouts vs invoices paid — weekly |

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
| **Property manager** | Occupancy, leases, tickets, escalation | Leases, units, work orders |
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
│  + RLS RBAC   │   │ cache/pubsub  │   │ Stripe Connect / regional│
│  + readings   │   │               │   │ Owner connected accounts │
│  (Timescale)  │   │               │   │ Webhooks → ledger        │
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
| **Leasing** | lease, occupant, deposit | Who lives where |
| **Rent** | invoice, line_item, payment, payout, arrears | Money to owner |
| **IoT** | asset (tank), sensor, reading, alert | Water ops |
| **Ops** | work_order, vendor, SLA | Maintenance |
| **Audit** | audit_log, webhook_event | Compliance |

---

## 5. Rent collection — money flow (owner receives funds)

### Chosen pattern: **Direct-to-owner (destination / split)**

```text
Resident pays invoice (card / bank / USSD / etc.)
        │
        ▼
Payment Provider charge
        │
        ├──► Building Owner connected account  ≈  rent amount
        ├──► (optional) PM connected account   ≈  management fee
        └──► Platform account                  ≈  small collection fee
                    │
                    ▼
           Owner bank payout (provider schedule)
```

**Platform never treats rent as company revenue.** Ledger records:

- `invoice` owed by resident for unit/period  
- `payment_intent` / charge id  
- `split`: owner_amount, pm_fee, platform_fee  
- `receipt` visible to resident + owner  

### Operational rent cycle

```text
1. Lease active on unit
2. Cron / calendar: generate monthly invoice (rent ± utilities ± late fee)
3. Notify resident (push + email/SMS)
4. Resident pays in app
5. Webhook: payment succeeded → mark invoice paid → update ledger
6. Owner balance increases on connected account → auto/manual payout to bank
7. If unpaid past due date → dunning (reminders) → arrears flag → PM workflow
8. Partial pay / dispute / refund → accounts clerk handles with audit trail
```

### Owner onboarding (KYC) — required before first rent

1. Admin links **building → legal owner entity**  
2. Owner completes payment-provider KYC (ID, bank)  
3. Status `payouts_enabled` → rent collection unlocked for that building  
4. Without KYC: invoices can be generated as **offline/manual** only (record cash/cheque), no card collection  

### Regional payment note

| Region | Typical rails |
|---|---|
| US/EU/UK | Stripe Connect |
| Africa (NG/GH/KE/etc.) | Paystack / Flutterwave + split settlements |
| India | Razorpay Route / similar |
| Always | Keep provider behind an internal `PaymentsPort` so you can swap |

---

## 6. Operations playbooks (full picture)

### A) Building onboarding

1. Create org → invite owner + admin  
2. Add building, floors, units  
3. Attach owner legal entity + payment KYC  
4. Map tanks/sensors; set low/critical thresholds  
5. Import or create leases + residents  
6. Go-live checklist: first invoice dry-run, sensor heartbeat OK  

### B) Move-in / lease

1. Unit marked vacant → create lease  
2. Resident user invited; deposit recorded  
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

- Owner KYC before live card collection  
- Clear fee disclosure (platform fee + processor fee)  
- Receipts and audit logs immutable  
- Chargebacks: freeze disputed invoice; owner notified  
- Trust accounting rules vary by country — if law requires **client trust account**, switch mode from direct-to-owner to **PM-managed trust** for that jurisdiction (config flag per org)  
- Sensor data is operational, not payment PII — still org-scoped via RLS  

---

## 11. MVP vs later (updated with rent)

### MVP (systems that must work together)
- Portfolio + RBAC  
- Leases + residents  
- Invoices + **pay-to-owner** collection + receipts  
- Water sensors + alerts + basic work orders  
- Web admin + mobile for resident pay + staff alerts  

### Later
- Autopay, late-fee policies, multi-currency  
- Full owner statements / tax exports  
- Predictive tank refill logistics  
- PMS/accounting export (QuickBooks, Xero)  
- Multi-owner splits on one building  

---

## 12. One-picture summary

```text
RESIDENT ──pays rent──► PAYMENTS RAIL ──payout──► BUILDING OWNER
    │                         │
    │                         └── small fee ──► PLATFORM
    │
    ├── uses app for dues / receipts / tickets
    └── sees shared water status

SENSORS ──MQTT──► PLATFORM ──alerts──► FACILITIES / PM ──WO──► STAFF/VENDOR

PM / ACCOUNTS ──operate leases, arrears, buildings──► same PLATFORM
OWNER ──sees money + portfolio + risk──► same PLATFORM
```

**Business in one sentence:** You sell software and sensor ops that help run buildings; **rent money belongs to the building owner**; you earn subscription (and optionally a thin collection fee), not the rent itself.
