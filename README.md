# Property Management + Water Sensors

Multi-platform property (buildings, units, flats) management with water-level sensor monitoring.

## Status

Architecture plan only — see [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## Planned stack (summary)

- **Web:** Next.js · **Mobile:** Expo (iOS/Android) · **DB:** PostgreSQL + RLS RBAC · **Telemetry:** Timescale/partitioned Postgres · **Devices:** MQTT · **Deploy:** Supabase + Vercel + Hetzner/MQTT (AWS optional)
