# Pema — UI Inventory (Pages, Look, Needs)

**Brand:** [Pema](./BRAND.md) — *Property, settled.*  
**Theme:** [Darkmatter](https://tweakcn.com/r/themes/darkmatter.json) via  
`npx shadcn@latest add https://tweakcn.com/r/themes/darkmatter.json`  
**Apps:** Next.js web (Owner/PM + public listings) · Expo mobile (Resident + Cleaning worker)  
**Default appearance:** **Dark** (Darkmatter dark tokens). Light mode supported via same CSS vars.

---

## 1. How the UI will look (Darkmatter)

| Token | Role in UI |
|---|---|
| **Background** | Near-black charcoal (`oklch ~0.18`) — dense ops feel |
| **Foreground** | Soft gray-white text |
| **Primary** | Warm amber/gold — CTAs, active nav, “Pay security”, AI accent |
| **Secondary** | Muted teal — secondary buttons, water chips |
| **Muted / accent** | Dark gray panels, hover rows |
| **Destructive** | Teal-tinted danger (theme) — use sparingly for delete/P0 |
| **Sidebar** | Slightly elevated black card |
| **Radius** | `0.75rem` — soft but not pill-heavy |
| **Fonts** | **Geist Mono** (UI) + **JetBrains Mono** (code/IDs) — technical PMS, not soft SaaS |

### Layout language
- **Owner/PM web:** left sidebar (**Pema** wordmark) + top bar (“Ask Pema”) + main canvas  
- **Prospect:** full-bleed listing gallery, minimal chrome  
- **Resident / Worker mobile:** bottom tab bar, large primary CTAs  
- Prefer **tables + dense lists** for ops; **cards only** for listing gallery and proof packs  
- Status badges: `vacant` `listed` `reserved` `secured` `overdue` `P0` in mono chips  

### Shell sketch (Owner/PM)

```text
┌──────────┬─────────────────────────────────────────────┐
│ Pema   │  Top: Building switcher · Notifs · Ask Pema │
│──────────┼─────────────────────────────────────────────┤
│ Dash     │  Main: KPI row + AI risk strip + tables     │
│ Units    │                                             │
│ Listings │                                             │
│ Rent     │                                             │
│ Cleaning │                                             │
│ Water    │                                             │
│ People   │                                             │
│ Settings │                                             │
└──────────┴─────────────────────────────────────────────┘
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
| **TOTAL screens to build (NOW)** | **~52** | 44 pages + shared auth + worker/resident |

**Later (Pamtsho WTP ERP):** +18–25 municipal screens — out of NOW scope.

---

## 3. Full page inventory

### A. Auth & shared (5)

| # | Route / screen | Who | UI look / purpose |
|---|---|---|---|
| 1 | `/login` | All | Centered card, mono logo, amber primary button |
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

**Shell:** `/app/(console)/…` with Darkmatter sidebar.

| # | Route | Purpose / UI |
|---|---|---|
| 12 | `/app` | **Command dashboard** — KPI tiles, AI risk strip, Ask AI |
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

*Overlays (not separate pages): Ask AI drawer, notification popover, confirm dialogs.*

*Dashboard + buildings 4 · units 3 · listings 4 · applications 2 · leases 2 · rent 3 · cleaning 3 · water 2 · alerts 1 · WO 2 · people 1 · settings 1 → **28**.*

### D. Resident mobile (8)

| # | Screen | UI |
|---|---|---|
| 40 | Home | Dues chip, tank gauge, Ask AI, quick pay |
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
- [ ] `npx shadcn@latest add https://tweakcn.com/r/themes/darkmatter.json`  
- [ ] Geist Mono + JetBrains Mono loaded  
- [ ] Dark default; theme toggle optional  
- [ ] Design tokens documented in `globals.css`  

### 4.2 shadcn components (install set)

```text
button input label textarea select checkbox radio-group switch
form dialog sheet drawer dropdown-menu popover
table card badge avatar separator tabs
calendar date-picker (or react-day-picker)
command (Ask AI palette)
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
- `AI_GATEWAY_API_KEY` for Ask AI  

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

1. Auth + shell (sidebar Darkmatter)  
2. Buildings / units / vacancy board  
3. Listings + photo slots + public detail  
4. Apply → pay security → secured status  
5. Escrow rent board + resident pay  
6. Cleaning jobs + proof viewer + worker check-in  
7. Building water gauge + alerts  
8. Ask AI drawer wired to tools  

---

## 6. Later UI (not NOW)

Pamtsho WTP ERP: control-room overview, process tags, CMMS, stores, lab, network map (~20+ pages) — see `PAMTSHO_WTP_ERP_BLUEPRINT.md`.

---

## 7. One-line summary

**52 screens NOW** on **Darkmatter** (dark, Geist Mono, amber primary): public listings → security escrow → Owner/PM console → resident + worker apps, with Ask AI on dashboards and clients.
