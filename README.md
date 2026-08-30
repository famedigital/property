# Property Management + Water Sensors

Multi-platform property (buildings, units, flats) management with water-level sensor monitoring.

## Status

Plan only — see:
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — tech stack, DB, deploy
- [`docs/SYSTEMS_AND_OPERATIONS.md`](docs/SYSTEMS_AND_OPERATIONS.md) — organogram, rent-to-owner, anti-skimming
- [`docs/THIMPHU_LORAWAN_PIPELINE.md`](docs/THIMPHU_LORAWAN_PIPELINE.md) — source→tank LoRaWAN + CCTV
- [`docs/WATER_SOURCE_ALERTS.md`](docs/WATER_SOURCE_ALERTS.md) — source water issue push/SMS alerts
- [`docs/CITY_WATER_MONITORING_COMPARISON.md`](docs/CITY_WATER_MONITORING_COMPARISON.md) — Seoul/Tokyo vs Thimphu
- [`docs/AI_AND_API_KEYS.md`](docs/AI_AND_API_KEYS.md) — AI features + `AI_GATEWAY_API_KEY`

## Planned stack (summary)

- **Web:** Next.js · **Mobile:** Expo or Flutter · **DB:** PostgreSQL + RLS RBAC · **Rent:** pay-to-owner · **Sensors:** LoRaWAN → ChirpStack → MQTT · **CCTV:** 4G/fiber · **AI:** Vercel AI Gateway (API key server-side) · **Deploy:** Supabase + Vercel + Hetzner (AWS optional)
