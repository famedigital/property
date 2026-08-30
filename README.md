# Property Management + Water Sensors

Multi-platform property (buildings, units, flats) management with water-level sensor monitoring.

## Status

Plan only — see:
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — tech stack, DB, sensors, deploy
- [`docs/SYSTEMS_AND_OPERATIONS.md`](docs/SYSTEMS_AND_OPERATIONS.md) — organogram, rent-to-owner money flow, ops

## Planned stack (summary)

- **Web:** Next.js · **Mobile:** Expo or Flutter · **DB:** PostgreSQL + RLS RBAC · **Rent:** pay-to-owner (Connect-style) · **Telemetry:** Timescale/partitioned Postgres · **Devices:** MQTT · **Deploy:** Supabase + Vercel + Hetzner/MQTT (AWS optional)
