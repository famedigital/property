# Unit Listing & Leasing Funnel

**Gap fixed:** Standard property-management leasing — not only collecting rent on existing tenants.  
**Scope (NOW):** Vacant units → public/private listings → photos (+ optional 3D) → interest → **pay security deposit into escrow** → lease secured → keys/access.

---

## 1. Funnel (like any PMS)

```text
Unit vacant
    → Listing published (rent, deposit, amenities, rules)
    → Media: standard photo set (+ optional 3D scan / tour if they like)
    → Prospect browses / favorites / inquiries / applications
    → Owner/PM approves applicant
    → Prospect pays SECURITY DEPOSIT (+ optional first rent) into ESCROW
    → Payment confirmed → unit SECURED (lease active, vacancy closed)
    → Handover checklist + access / keys
    → Ongoing escrow rent (existing module)
```

---

## 2. Unit & vacancy states

| Status | Meaning |
|---|---|
| `occupied` | Active lease |
| `notice` | Tenant giving notice; may pre-list |
| `vacant` | Empty; ready to list |
| `listed` | Published for prospects |
| `reserved` | Application approved; awaiting security payment |
| `secured` | Security paid; lease starting / active |
| `maintenance` | Offline for repairs / cleaning before list |

Vacancy board for Owner/PM: counts + days vacant + listed vs not listed.

---

## 3. Listing

### Fields (MVP)
- Unit identity (building, floor, unit code)  
- Type / beds / baths / area  
- Asking rent, currency, available-from date  
- **Security deposit** amount (and whether first month due together)  
- Amenities, pets, furnished flag  
- Description  
- Contact = PM/owner via in-app inquiry (no owner PII to public / RRCO)  
- Visibility: public marketplace / invite-only link  

### Actions
- Publish / unpublish / mark reserved / withdraw  
- Clone listing template per building  

### Public listing detail (prospect)

Target experience: **light**, portal-familiar (address · beds/baths · rent · big gallery · CTA), but **better than** lead-gen sites:

| Portal-typical | PEMA |
|---|---|
| Heavy third-party agency strip | Quiet **PEMA** chrome |
| Enquire → spam leads | **Apply → escrow security → secured** |
| Random photo dump | **Standard photo slots** + optional **3D** |
| Dead end after click | Same product as rent, cleaning, water, Ask PEMA |

Visual mock: [`UI_PAGES_AND_LOOK.html`](./UI_PAGES_AND_LOOK.html#listing-detail).

---

## 4. Media — standard photos + optional 3D

### Standard photo options (required set, guided upload)
Preset slots so listings look consistent:

| Slot | Example |
|---|---|
| Exterior / entrance | Building face |
| Living | Living room |
| Kitchen | Kitchen |
| Bedroom(s) | Per bedroom |
| Bathroom(s) | Per bath |
| View / balcony | Optional |
| Other | Extra shots |

Rules: min N photos to publish; max size; watermark optional; stored in R2/S3.

### Optional 3D scan / virtual tour
- If prospect/owner **likes**: upload Matterport / Kuula / similar embed URL, **or** upload 360° package  
- Listing flag `has_3d_tour`  
- Not required for MVP publish — **photos are enough**; 3D is upsell / later polish  

---

## 5. Application → pay security → get secured

```text
Prospect applies (ID + basic KYC fields)
    → PM/Owner approve or reject
    → On approve: status = reserved; payment request created
    → Prospect pays SECURITY into ESCROW (same escrow rail as rent)
    → Webhook: deposit held on ledger (deposit_type=security)
    → System:
         • create lease
         • unit status = secured / occupied
         • close listing
         • issue money receipt (resident)
         • owner sees deposit on settlement board (internal KYC only)
    → Handover: checklist, meter photos, key log / access code
```

**Secured means:** deposit cleared in escrow + lease record active + vacancy cleared — not “manager said they paid.”

Refund / forfeit of security on move-out: audited escrow ledger movements (later rules engine).

---

## 6. Data model (sketch)

```text
units                   (+ status vacancy machine)
unit_listings           (unit_id, rent, deposit, available_from, published_at, …)
listing_media           (listing_id, slot, url, kind: photo|360|3d_embed)
listing_inquiries       (listing_id, prospect_user_id, message)
applications            (listing_id, prospect_id, status, reviewed_by)
security_payments       → escrow_ledger_entries (type=security_deposit)
leases                  (unit_id, resident_id, start, deposit_ledger_id)
handover_checklists     (lease_id, items, photos)
```

RBAC: public read of **published** listings (no owner bank/PII); apply requires auth; approve = PM/Owner; escrow = server only.

---

## 7. Screens (MVP)

| Role | Screens |
|---|---|
| **Prospect** | Browse listings, photo gallery, 3D if present, apply, pay security |
| **PM** | Vacancy board, create listing, photo uploader (standard slots), approve apps |
| **Owner** | Vacancy + listed units, deposit received, secured leases |
| **Resident** | After secured: rent dues, receipts, tickets, water |

---

## 8. MVP vs later

### MVP (NOW — with escrow / cleaning / building water)
- Vacancy statuses + board  
- Listings with **standard photo slots**  
- Inquiries + applications  
- **Pay security into escrow → secured lease**  
- Optional 3D/360 embed URL field  

### Later
- In-app guided 3D capture SDK  
- Credit checks / e-sign contracts  
- Waitlist / auto-match  
- Deposit dispute workflows  

---

## 9. Relation to other NOW modules

| Module | Link |
|---|---|
| Escrow rent | Same escrow; security is first ledger type before monthly rent |
| Cleaning | Vacant unit often triggers cleaning job before `listed` |
| Building water | Resident app after secured |
| RRCO | Still **no owner roster export**; receipts to payer only |
