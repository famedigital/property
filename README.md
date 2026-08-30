# Property Management + Water Sensors

Multi-platform property (buildings, units, flats) management with water-level sensor monitoring.

## Status

Plan only — see:
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — tech stack, DB, deploy
- [`docs/SYSTEMS_AND_OPERATIONS.md`](docs/SYSTEMS_AND_OPERATIONS.md) — organogram, rent-to-owner, anti-skimming
- [`docs/THIMPHU_LORAWAN_PIPELINE.md`](docs/THIMPHU_LORAWAN_PIPELINE.md) — source→tank LoRaWAN + CCTV

## Planned stack (summary)

- **Web:** Next.js · **Mobile:** Expo or Flutter · **DB:** PostgreSQL + RLS RBAC · **Rent:** pay-to-owner · **Sensors:** LoRaWAN → ChirpStack → MQTT · **CCTV:** 4G/fiber (not LoRa) · **Deploy:** Supabase + Vercel + Hetzner (AWS optional)
