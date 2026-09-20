# PEMA — UI Inventory (Pages, Look, Needs)

**Brand:** [PEMA](./BRAND.md) — Property, Estate, Management Application · *Property, settled.*  
**Theme:** [Northern Lights](https://tweakcn.com/r/themes/northern-lights.json) via  
`npx shadcn@latest add https://tweakcn.com/r/themes/northern-lights.json`  
**Apps:** Next.js web (Owner/PM + public listings) · Expo mobile (Resident + Cleaning worker)  
**Default appearance:** **Light** (Northern Lights light tokens). Dark mode optional via same CSS vars.

---

## 1. How the UI will look (Northern Lights)

| Token | Role in UI |
|---|---|
| **Background** | Soft cool-white (`oklch ~0.98`) — settled light ops feel |
| **Foreground** | Charcoal text |
| **Primary** | Aurora **green** — CTAs, active nav, “Pay security”, Ask PEMA |
| **Secondary** | Cool **blue** — secondary buttons, water chips |
| **Accent** | Violet-blue aurora — highlights, AI strip edge |
| **Muted** | Mid-gray panels / rows |
| **Destructive** | Warm red — delete / P0 only |
| **Sidebar** | Same as background, bordered |
| **Radius** | `0.5rem` |
| **Fonts** | **Plus Jakarta Sans** (UI) · **JetBrains Mono** (IDs) · Source Serif 4 optional |

### Layout language
- **Owner/PM web:** left sidebar (**PEMA** wordmark) + top bar (“Ask PEMA”) + main canvas  
- **Prospect:** light listing browse + **detail page** in the same genre as portals (address, specs, big gallery, CTA) — cleaner, escrow-native  
- **Resident / Worker mobile:** bottom tab bar, large green primary CTAs  
- Prefer **tables + dense lists** for ops; **cards only** for listing gallery and proof packs  
- Status badges: `vacant` `listed` `reserved` `secured` `overdue` `P0` in mono chips  

### Prospect listing detail (vs typical portals)

Same job: light page · address · beds/baths · rent · dominant photo gallery · apply.  
**Better in PEMA:** no heavy agency brand bar; PEMA chrome; security amount + escrow copy; Apply → pay security → secured; guided photo slots + optional 3D; Ask PEMA; same system as rent/cleaning/water (not a dead-end lead form).

See visual mock in [`UI_PAGES_AND_LOOK.html`](./UI_PAGES_AND_LOOK.html#listing-detail). 

### Shell sketch (Owner/PM)

```text
┌──────────┬─────────────────────────────────────────────┐
│ PEMA     │  Top: Building switcher · Notifs · Ask PEMA │
│──────────┼─────────────────────────────────────────────────┤
│ Dash     │  Main: KPI row + AI risk strip + tables         │
│ Units    │                                                 │
│ Listings │                                                 │
│ Rent     │                                                 │
│ Cleaning │                                                 │
│ Water    │                                                 │
│ People   │                                                 │
│ Settings │                                                 │
└──────────┴─────────────────────────────────────────────────┘
```

---

## 2. Page count summary (NOW MVP)

| Surface | Pages (routes/screens) | Notes |
|---|---|---|
| **Auth & shared** | **5** | Login, signup, forgot, invite accept, 404 |
| **Public / prospect web** | **6** | Browse, detail, apply, pay security, status, favorited |
| **Owner / PM web** | **28** | Full ops console |
| **Resident mobile** | **8** | Home, pay, receipts, tickets, water, profile, AI, notifs |
| **Cleaning worker mobile** | **5** | Jobs today, job detail, check-in, history, profile |
| **Overlays** (not full pages) | AI drawer, confirm dialogs, upload sheets | Counted separately |
| **TOTAL screens to build (NOW)** | **52** | Auth + prospect + console + mobile |

**Later (Pamtsho WTP ERP):** +18–25 municipal screens — out of NOW scope.

---

## 3. Full page inventory

### A. Auth & shared (5)

| # | Route / screen | Who | UI look / purpose |
|---|---|---|---|
| 1 | `/login` | All | Centered card, PEMA wordmark, green primary button |
| 2 | `/signup` | Prospect / invite | Same shell; role from invite token |
| 3 | `/forgot-password` | All | Email reset |
| 4 | `/invite/[token]` | Staff/owner | Accept membership |
| 5 | `/404` | All | Minimal |

### B. Public / prospect (6)

| # | Route | UI look / purpose |
|---|---|---|
| 6 | `/listings` | Filter bar + **photo card grid** (not ops tables) |
| 7 | `/listings/[id]` | Gallery (standard slots), optional 3D embed, rent/deposit, Apply CTA |
| 8 | `/listings/[id]/apply` | Multi-step form (ID, contacts) |
| 9 | `/listings/[id]/pay-security` | Checkout — escrow copy, amount, pay |
| 10 | `/applications/[id]/status` | Timeline: applied → approved → pay → **secured** |
| 11 | `/favorites` | Saved listings (optional MVP — count included) |

### C. Owner / PM web console (28)

**Shell:** `/app/(console)/…` with Northern Lights sidebar.

| # | Route | Purpose / UI |
|---|---|---|
| 12 | `/app` | **Command dashboard** — KPI tiles, AI risk strip, Ask PEMA |
| 13 | `/app/buildings` | Table of buildings |
| 14 | `/app/buildings/new` | Create building form |
| 15 | `/app/buildings/[id]` | Building overview (units, water, cleaning) |
| 16 | `/app/units` | **Vacancy board** — status filters, days vacant |
| 17 | `/app/units/new` | Add unit |
| 18 | `/app/units/[id]` | Unit detail + status machine |
| 19 | `/app/listings` | All listings table |
| 20 | `/app/listings/new` | Create listing (rent, deposit, amenities) |
| 21 | `/app/listings/[id]` | Edit listing |
| 22 | `/app/listings/[id]/media` | **Photo slot uploader** + optional 3D URL |
| 23 | `/app/applications` | Inbox — pending approvals |
| 24 | `/app/applications/[id]` | Approve/reject + jump to pay status |
| 25 | `/app/leases` | Active / past leases |
| 26 | `/app/leases/[id]` | Lease + **handover checklist** |
| 27 | `/app/rent` | Escrow board: expected / collected / outstanding |
| 28 | `/app/rent/invoices/[id]` | Invoice + ledger lines |
| 29 | `/app/rent/payouts` | Owner settlement batches |
| 30 | `/app/cleaning` | Schedule calendar / list |
| 31 | `/app/cleaning/jobs/[id]` | Job + **proof pack** (biometric events + CCTV/selfie) |
| 32 | `/app/cleaning/workers` | Workers / enrollment status |
| 33 | `/app/water` | Building tanks overview |
| 34 | `/app/water/[buildingId]` | Live level, chart, alerts |
| 35 | `/app/alerts` | Alert inbox (water + cleaning no-show) |
| 36 | `/app/work-orders` | WO list |
| 37 | `/app/work-orders/[id]` | WO detail |
| 38 | `/app/people` | Members + roles |
| 39 | `/app/settings` | Org profile + escrow + notification prefs (tabs) |

*Overlays (not separate pages): Ask PEMA drawer, notification popover, confirm dialogs.*

*Dashboard + buildings 4 · units 3 · listings 4 · applications 2 · leases 2 · rent 3 · cleaning 3 · water 2 · alerts 1 · WO 2 · people 1 · settings 1 → **28**.*

### D. Resident mobile (8)

| # | Screen | UI |
|---|---|---|
| 40 | Home | Dues chip, tank gauge, Ask PEMA, quick pay |
| 41 | Pay rent | Escrow checkout |
| 42 | Receipts | List + PDF |
| 43 | Tickets | Create / list |
| 44 | Water | Tank status + history |
| 45 | AI chat | Resident assistant |
| 46 | Notifications | List |
| 47 | Profile | Account |

### E. Cleaning worker mobile (5)

| # | Screen | UI |
|---|---|---|
| 48 | Jobs today | List with time windows |
| 49 | Job detail | Checklist + map/address |
| 50 | Check-in / out | Biometric prompt + CCTV/alt proof capture |
| 51 | History | Past jobs |
| 52 | Profile | Worker profile |

**Canonical count for planning:**

| Group | Count |
|---|---|
| Auth/shared | 5 |
| Prospect | 6 |
| Owner/PM | 28 |
| Resident | 8 |
| Worker | 5 |
| **Total NOW** | **52** |

---

## 4. Things needed (to build UI)

### 4.1 Theme & design system
- [ ] Next.js app with Tailwind v4 / shadcn init  
- [ ] `npx shadcn@latest add https://tweakcn.com/r/themes/northern-lights.json`  
- [ ] Plus Jakarta Sans + JetBrains Mono (+ Source Serif 4 optional)  
- [ ] Dark default; theme toggle optional  
- [ ] Design tokens documented in `globals.css`  

### 4.2 shadcn components (install set)

```text
button input label textarea select checkbox radio-group switch
form dialog sheet drawer dropdown-menu popover
table card badge avatar separator tabs
calendar date-picker (or react-day-picker)
command (Ask PEMA palette)
sonner / toast
sidebar navigation-menu breadcrumb
scroll-area skeleton progress alert
chart (for tank / rent graphs)
```

### 4.3 Custom / domain components
- `VacancyStatusBadge`  
- `PhotoSlotGrid` (standard slots)  
- `ListingGallery`  
- `Embed3D` (iframe URL)  
- `EscrowPayButton`  
- `TankLevelGauge`  
- `ProofPackViewer` (biometric events + media)  
- `AiChatPanel` / `AiRiskStrip`  
- `KpiStat`  

### 4.4 Infra for UI
- Object storage (R2/S3) for photos, CCTV clips, proof selfies  
- Escrow payment checkout (web)  
- Push notifications (Expo)  
- Biometric device SDK or supervised mobile capture  
- Maps (optional for worker address)  
- `AI_GATEWAY_API_KEY` for Ask PEMA  

### 4.5 Roles → default home

| Role | Lands on |
|---|---|
| Owner | `/app` dashboard |
| PM | `/app/units` vacancy board or `/app` |
| Accounts | `/app/rent` |
| Prospect | `/listings` |
| Resident | Mobile Home |
| Cleaning worker | Jobs today |

---

## 5. MVP build order (UI)

1. Auth + shell (sidebar Northern Lights)  
2. Buildings / units / vacancy board  
3. Listings + photo slots + public detail  
4. Apply → pay security → secured status  
5. Escrow rent board + resident pay  
6. Cleaning jobs + proof viewer + worker check-in  
7. Building water gauge + alerts  
8. Ask PEMA drawer wired to tools  

---

## 6. Later UI (not NOW)

Pamtsho WTP ERP: control-room overview, process tags, CMMS, stores, lab, network map (~20+ pages) — see `PAMTSHO_WTP_ERP_BLUEPRINT.md`.

---

## 7. One-line summary

**52 screens NOW** on **Northern Lights** (light, Plus Jakarta Sans, aurora green primary): public listings → security escrow → Owner/PM console → resident + worker apps, with Ask PEMA on dashboards and clients.
