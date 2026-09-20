# Full AI Control — Dashboards & Clients

**Goal:** AI is not a side feature — it **operates dashboards and client experiences** across listing, escrow rent, cleaning, and building water.  
**Stack:** Vercel AI SDK + **AI Gateway** (`AI_GATEWAY_API_KEY` server-only).  
**Hard rule:** Deterministic systems still win for money + safety (escrow webhooks, biometric proof, P0 water alarms). AI **controls UX, prioritization, drafting, and copilots** — it does not invent paid/secured states.

---

## 1. What “full AI” means here

| Layer | AI does | AI must NOT do alone |
|---|---|---|
| **Dashboards (Owner / PM)** | Rank risks, explain widgets, auto-layout focus, morning brief, natural-language queries | Change ledger balances without tools + audit |
| **Clients (Prospect / Resident)** | Listing Q&A, apply help, rent FAQ, water status in plain language, multilingual | Bypass escrow payment or fake “secured” |
| **Ops (Cleaning / Facilities)** | Schedule suggest, no-show risk, WO drafts, proof review hints | Mark cleaning done without biometric+proof |
| **Vision** | Photo slot check, optional 3D quality hint, CCTV summarize | Replace biometric identity |

```text
User (dashboard or client app)
    → Chat / voice / “Ask AI” panel
        → Backend agent (RBAC + tools)
            → AI Gateway (API key)
            → Tools: search units, listings, escrow summary, alerts, cleaning jobs
            → UI actions: pin widgets, draft listing, open WO, send reminder
```

---

## 2. AI products to build

### A. Owner / PM **Command Dashboard AI**
- **Risk cockpit:** AI ranks today’s issues (arrears, vacancies days, tank critical, cleaning no-shows).  
- **NL query:** “Which units are vacant > 14 days without listing?” → tool query → table + suggest publish.  
- **Widget control:** User says “show only water + rent arrears” → AI sets dashboard layout prefs.  
- **Briefs:** Daily/weekly narrative for owner (Dzongkha/English later).  
- **Actions with confirm:** “Remind unit 3B about rent” → drafts message → human send (or auto if policy allows).

### B. Prospect / Resident **Client AI**
- **Listing concierge:** Answer amenities, deposit, photos; suggest units from filters.  
- **Apply coach:** Missing fields, document checklist.  
- **Security pay helper:** Explain escrow steps (“pay security → get secured”).  
- **Resident assistant:** Balance due, receipt find, water tank status, raise ticket in chat.  
- **Language:** Prefer bilingual prompts when ready.

### C. Cleaning **Workforce AI**
- Suggest schedules from vacancy / turnover.  
- Flag likely no-show from history.  
- Review proof pack (vision: “person at door?”) — **advisory only**; biometric still required.  
- Draft vendor messages.

### D. Building water **Ops AI**
- Explain tank alerts; suggest pump vs supply.  
- Owner push copy generation.  
- (Later) corridor / Pamtsho root-cause when municipal phase starts.

### E. Listing media AI
- Check **standard photo slots** completeness (“missing kitchen photo”).  
- Caption / description draft from photos.  
- Optional: quality score before publish.

---

## 3. Agent architecture

```text
┌─────────────────────────────────────────────┐
│  AI Gateway  (AI_GATEWAY_API_KEY)           │
│  models: fast for chat, stronger for plans  │
└──────────────────┬──────────────────────────┘
                   │
         ┌─────────▼─────────┐
         │  Orchestrator     │  (Next.js server)
         │  - auth + RLS ctx │
         │  - tool registry  │
         │  - audit log      │
         └─────────┬─────────┘
                   │ tools
     ┌─────────────┼─────────────┬──────────────┐
     ▼             ▼             ▼              ▼
  listings     escrow       cleaning        water
  vacancies    invoices     jobs/proofs     sensors
  media        payouts      workers         alerts
```

### Tools (examples — all RBAC-scoped)

| Tool | Effect |
|---|---|
| `list_vacancies` | Query unit statuses |
| `create_listing_draft` | Draft listing + photo slot checklist |
| `summarize_escrow` | Expected vs collected for owner |
| `list_overdue_invoices` | Arrears board |
| `list_cleaning_jobs` | Today’s jobs / no-shows |
| `get_tank_status` | Building water |
| `draft_work_order` | Create WO draft |
| `set_dashboard_focus` | Persist UI preference |
| `send_reminder` | Queued notification (policy-gated) |

**Confirm step:** Money-moving or status-closing tools require explicit user confirm unless org enables autopilot for that tool.

---

## 4. Dashboard control model

1. **Default dashboard** = rules-based KPIs (always work offline from AI).  
2. **AI layer** overlays: ranked cards, explanations, suggested actions.  
3. **Voice/chat bar** on every Owner/PM screen: “Ask anything about my buildings.”  
4. **Autopilot modes** (per org toggle):
   - Off — suggest only  
   - Assist — auto-draft, human approve  
   - Auto — allowed safe actions (reminders, listing captions) without money changes  

---

## 5. Client control model

| Client | AI surface |
|---|---|
| Prospect web/app | Listing chat + apply help |
| Resident app | Home assistant (rent, water, tickets) |
| Cleaning worker app | Job instructions only (no owner money data) |

Guardrails: strip PII; never expose other tenants’ data; never expose owner bank details to prospects.

---

## 6. Models & keys

| Piece | Choice |
|---|---|
| SDK | Vercel AI SDK |
| Router | AI Gateway — one key, many models |
| Env | `AI_GATEWAY_API_KEY` server-only |
| Fast path | Small/cheap model for chat & captions |
| Deep path | Stronger model for briefs & multi-tool plans |
| Vision | Gateway vision model for photo slots / CCTV stills |

Setup: Vercel AI Gateway → create key → `.env.local` + Vercel env. Never `EXPO_PUBLIC_*`.

---

## 7. Safety matrix

| Domain | Source of truth | AI role |
|---|---|---|
| Escrow paid / secured | Payment webhook + ledger | Explain, remind, draft |
| Cleaning done | Biometric + proof | Suggest, review |
| Water P0 | Sensor rules | Explain, notify copy |
| Listing published | PM publish action | Draft, slot check |
| Dashboard numbers | SQL aggregates | Rank, narrate |

If AI Gateway is down: dashboards and payments still work; chat shows “AI offline.”

---

## 8. MVP AI (ship with building product)

1. Owner/PM chat with tools (vacancies, arrears, tank, cleaning)  
2. Dashboard morning brief  
3. Listing description + photo-slot checker  
4. Resident FAQ bot (rent + water)  
5. Water alert explain + cleaning no-show draft message  

### Phase 2 AI
- Full dashboard layout autopilot  
- Vision on cleaning CCTV  
- Multilingual Dzongkha  
- Pamtsho WTP plant copilot (later municipal)

---

## 9. Data / cost

- Log `ai_usage(org_id, feature, tokens)`  
- Cache briefs 15–60 min  
- Gateway budget caps  
- Minimize PII in prompts  

---

## 10. One-line decision

**Full AI via AI Gateway controls dashboards and clients through an RBAC tool-calling agent; escrow, biometric, and sensor rules remain the system of record.**
