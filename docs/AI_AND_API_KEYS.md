# AI Layer — Where AI Runs + API Keys

**Goal:** Use AI across property ops, rent, and **Thimphu source→tank water monitoring** — with **one managed API key path**, never keys in git.

---

## 1. Recommended AI stack

| Piece | Choice | Why |
|---|---|---|
| SDK | **Vercel AI SDK** (`ai` package) | Streaming, tools, typed, works in Next.js |
| Key routing | **Vercel AI Gateway** | **One key** → many models (OpenAI, Anthropic, etc.) |
| Local auth | `AI_GATEWAY_API_KEY` in `.env.local` | Simple for laptop / CI |
| Prod on Vercel | OIDC `VERCEL_OIDC_TOKEN` (or same gateway key) | Less key sprawl |
| Fallback (non-Vercel host) | Same gateway key, or provider keys via BYOK | Hetzner API can still call Gateway over HTTPS |

**Do not** put OpenAI/Anthropic keys in the mobile app. All AI calls go through **your backend** (Next.js server / edge function).

```text
Mobile / Web UI
    → your API (authenticated user)
        → AI Gateway (AI_GATEWAY_API_KEY or OIDC)
            → model (openai/…, anthropic/…, …)
```

---

## 2. Where AI is used in *this* product

| Feature | AI job | Inputs | Output |
|---|---|---|---|
| **Source water alert explain** | Turn sensor codes into human message (Dzongkha/English later) | `SRC_FLOW_ZERO`, readings, stage | Push/SMS copy + severity suggest |
| **Root-cause assist** | “Source vs leak vs building pump?” | Last N readings along path + open alerts | Ranked cause + recommended action |
| **Alert triage** | Dedupe / priority when many nodes alarm | Alert burst | Group into one incident |
| **Work-order draft** | Auto ticket text from alert | Alert + asset + CCTV link | Draft WO for facilities |
| **Owner daily brief** | Morning summary | Overnight alerts, tank %, rent arrears | Short brief in app |
| **Resident FAQ bot** | “Why is water low?” | Public alert stage + FAQ policy | Safe answer (no internal money data) |
| **Rent anomaly** | Flag odd patterns | Ledger (paid/expected) | “Unit 12 always marked cash by manager” risk hint |
| **CCTV assist (later)** | Describe clip / detect person at intake | Snapshot URL | Note on alert (optional vision model) |
| **Ops copilot (admin)** | Ask “which buildings fed by Chamgang are critical?” | Tools → DB (RBAC-scoped) | Answer with citations |

**MVP AI (ship first):** alert explain + root-cause assist + work-order draft + owner brief.  
**Later:** resident bot, vision, full copilot with tools.

---

## 3. API keys — how to set up (no secrets in repo)

### A) Create gateway key
1. Vercel Dashboard → **AI Gateway** → **API Keys** → Create  
2. Copy key once  

### B) Local
```bash
# .env.local  (gitignored)
AI_GATEWAY_API_KEY=gw_xxxxxxxx

# Optional direct providers (only if not using Gateway)
# OPENAI_API_KEY=
# ANTHROPIC_API_KEY=
```

### C) Vercel project
- Settings → Environment Variables  
- Add `AI_GATEWAY_API_KEY` for Production / Preview / Development  
- Or rely on OIDC on deployed Vercel and keep gateway key for local only  

### D) Committed template only
```bash
# .env.example  (safe to commit)
AI_GATEWAY_API_KEY=
# DATABASE_URL=
# MQTT_URL=
# PAYMENTS_SECRET_KEY=
```

### E) Rules
- Never commit `.env.local`  
- Never put keys in Expo/`EXPO_PUBLIC_*`  
- Rotate key if leaked  
- Set Gateway **budget caps** in Vercel  
- Log `request_id` / token usage per org for cost control  

---

## 4. Code pattern (Next.js server)

```ts
// apps/web — server only
import { generateText } from 'ai';

export async function explainSourceAlert(input: {
  code: string;
  stage: string;
  value: number;
  sourceName: string;
}) {
  const { text } = await generateText({
    model: 'anthropic/claude-sonnet-4.5', // via AI Gateway string
    system: `You write short water-supply alerts for building owners in Thimphu.
Be factual. Name the stage (SOURCE/MAIN/ZONE/BUILDING). Max 2 sentences.`,
    prompt: JSON.stringify(input),
  });
  return text;
}
```

Auth to Gateway: AI SDK reads `AI_GATEWAY_API_KEY` automatically (or OIDC on Vercel).

**Tool-calling ops copilot** (later): AI SDK `tools` that query Postgres **only after** setting RLS `org_id` / user role — AI never bypasses RBAC.

---

## 5. Architecture fit

```text
LoRaWAN → readings → alert rules (deterministic first)
                         │
                         ├─ always: create alert + notify (rules)
                         └─ then: AI enriches message / suggests cause
                                      │
                                      └─ AI Gateway (API key)
```

**Important:** Core safety alerts must work **even if AI / API key is down**.  
Rules engine = source of truth for P0/P1 fire. AI = wording + advice + drafts.

```text
if (aiGatewayOk) enrichAlert()
else sendTemplateAlert()   // never block water P0 on LLM
```

---

## 6. Cost control

| Control | Practice |
|---|---|
| Model tier | Cheap/fast for SMS copy; stronger model for root-cause |
| Cache | Same `SRC_FLOW_ZERO` template cache 15 min |
| Batch | Owner brief once/day, not per reading |
| Cap | Gateway budget + per-org monthly AI quota |
| Meter | Store `ai_usage(org_id, feature, tokens, cost)` |

---

## 7. Security / compliance

- AI sees **minimized** payloads (no full resident PII in prompts unless needed)  
- Rent tools: owner/accounts only  
- Prompt injection: treat sensor/user text as untrusted data  
- Audit: who ran copilot, what tools fired  
- Bhutan / data: prefer hosting region you already chose; don’t send CCTV frames to AI unless policy allows  

---

## 8. Env checklist for this monorepo

| Variable | Where | Purpose |
|---|---|---|
| `AI_GATEWAY_API_KEY` | Vercel + `.env.local` | All LLM calls |
| `DATABASE_URL` | Server | Postgres |
| `MQTT` / ChirpStack creds | Bridge service | Sensors |
| Payment secrets | Server only | Rent-to-owner |
| Expo push keys | Server | Notifications |
| `EXPO_PUBLIC_API_URL` | Mobile | Public API base only |

---

## 9. Rollout

1. Add AI Gateway key to env  
2. Wire `explainSourceAlert` on P0/P1 create  
3. Owner morning brief job  
4. Work-order draft button  
5. Ops copilot with RBAC tools  
6. Optional vision on source CCTV stills  

---

## 10. One-line decision

**Use Vercel AI SDK + AI Gateway with `AI_GATEWAY_API_KEY` (server-side only).**  
AI explains and assists; **LoRaWAN rules still fire water alerts if the API key or model fails.**
