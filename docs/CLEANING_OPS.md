# Cleaning Operations — Biometric + Proof of Presence

**Problem:** Cleaning workers are unreliable (no-shows, short visits, fake “done”). Managers cannot prove attendance to owners.  
**Solution:** Scheduled cleaning jobs with **biometric check-in/out** and **CCTV or alternate proof** when workers arrive and leave.  
**Product phase:** **Now** (with rental escrow + building water IoT). Main-source / Pamtsho WTP ERP is **later**.

---

## 1. Groundwork order (locked)

```text
1) Local building IoT  ← you already have / prioritize first
2) Building product NOW:
      • Escrow rental
      • Cleaning + biometric + CCTV/proof
      • Water IoT per individual building (tank / inlet)
3) Main source / Pamtsho WTP complete ERP  ← later (blueprint kept)
```

---

## 2. Cleaning workflow

```text
Schedule created (building / common area / unit)
        │
        ▼
Worker arrives → BIOMETRIC check-in (fingerprint / face device or app+approved device)
        │         + auto CCTV snapshot / clip start (or GPS+selfie proof if no cam)
        ▼
Job in progress (timer, optional checklist photos)
        │
        ▼
Worker leaves → BIOMETRIC check-out
        │         + CCTV end clip / checkout selfie
        ▼
Job = completed with proof pack
        │
        ▼
Owner / PM can review: who · when · duration · media
        │
        ▼
No biometric or proof → job cannot close as “done” (stays failed / incomplete)
```

---

## 3. Proof stack (required)

| Proof | Role | When |
|---|---|---|
| **Biometric** | Identity of the worker (not a borrowed phone) | Check-in and check-out |
| **CCTV** | Presence at building (preferred) | Motion/clip linked to job window |
| **Alternate proof** | If no CCTV yet | Geo-fenced selfie + timestamp + optional QR at site |
| **Checklist photos** | Work quality (optional) | Mid/end of job |

**Rule:** Closing a cleaning job requires **biometric in + biometric out** and **at least one** of: CCTV clip spanning visit **or** approved alternate proof. Duration must meet minimum (config per building).

---

## 4. Hardware / site kit (per building)

| Item | Notes |
|---|---|
| Biometric terminal | Fingerprint and/or face; PoE or 4G; offline buffer |
| Or: supervised mobile biometric | App with liveness + device attestation (weaker than fixed terminal) |
| CCTV | Entrance / service area; 4G or LAN NVR; clips to object storage |
| QR plaque | Alternate check-in anchor at service entrance |
| Local IoT gateway | Same building gateway used for water sensors |

---

## 5. Data model (sketch)

```text
cleaning_zones          (building_id, name, type: common|unit|stair)
cleaning_schedules      (zone_id, cron/rrule, worker_or_crew_id, min_minutes)
cleaning_jobs           (schedule_id, date, status: scheduled|in_progress|done|failed|no_show)
cleaning_workers        (user_id, biometric_enrolled, vendor_id)
cleaning_events         (job_id, type: in|out, biometric_ok, device_id, at)
cleaning_proofs         (job_id, kind: cctv|selfie|qr|checklist, url, at)
```

RBAC: Owner sees proofs for their buildings; PM assigns crews; workers see only own jobs; **no RRCO involvement**.

---

## 6. Alerts

| Event | Notify |
|---|---|
| No-show (no check-in by start+grace) | PM + Owner optional |
| Check-in without check-out | PM |
| Duration below minimum | PM (job auto-failed) |
| Biometric mismatch | PM + lock job |
| CCTV offline on job day | PM (force alternate proof mode) |

---

## 7. MVP vs later

### MVP
- Schedules + jobs  
- Biometric in/out (fixed device or supervised app)  
- CCTV clip link **or** geo-selfie+QR alternate  
- Owner/PM proof viewer  
- No-show alerts  

### Later
- Vendor payroll from verified minutes  
- AI checklist scoring from photos  
- Multi-building crew routing  

---

## 8. Relation to other modules

| Module | Link |
|---|---|
| **Escrow rent** | Same building org; cleaning fee can be line item on owner statement |
| **Building water IoT** | Same local gateway / CCTV backbone |
| **Pamtsho WTP ERP** | Out of scope until building product is stable |
