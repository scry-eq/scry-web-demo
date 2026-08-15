# scry-web-demo

A standalone WebSocket server that pretends to be a Scry daemon, so
`scry-web` can be exercised without a real EQ packet capture. Speaks real
`seq.v1` protobuf — `scry-web` connects unchanged. See
[`README.md`](README.md) for the query params that control zone/spawn
simulation.

## Stack

- Bun + TypeScript, no framework.
- `@bufbuild/protobuf` + buf-generated bindings from `proto/` (a git
  submodule → `scry-proto`, same schema every other Scry component uses).

## Structure

- `src/server.ts` — WebSocket server + per-session simulation loop
- `src/geometry.ts` — `maps/*.txt` parser (matches the daemon's
  `loadSOEMap`) + zone scanner
- `src/sim.ts` — mob & player movement step
- `src/mobs.ts` — NPC name pool + `combat.pbstream` extraction
- `src/filters.ts` — in-memory FilterRule engine
- `src/seed.ts` — static panel seed data (categories, group, items, …)
- `src/zoneNames.ts` — generated short→long zone name map
- `src/smoke.ts`, `src/smoke-filters.ts` — self-contained clients that
  verify the server without needing `scry-web` running
- `maps/` — vendored zone geometry files (curated ~40-zone set)

## Commands

- `git submodule update --init` — one-time, pulls `scry-proto` into `proto/`
- `bun install`
- `bun run gen` — regenerate `src/gen/` from `proto/`
- `bun run start` — listen on `ws://localhost:9091`
- `bun run dev` — same, with `bun --watch`
- `bun run typecheck`

## Conventions

- **The wire contract is real, not a simplified stand-in** — every message
  must be valid `seq.v1` protobuf a real `scry-web` client already speaks.
  Never invent a demo-only shape.
- **The filter engine mirrors scry-cpp's `FilterItem` pattern syntax
  exactly** (`filter.cpp:92`: `<regex>[;<minlevel>[-<maxlevel>]]`), so the
  filter panel behaves identically against the demo and a real daemon.
- **Zone geometry parsing matches the daemon's `loadSOEMap` rules**
  (`mapcore.cpp:940`) — a map that renders correctly here should render
  correctly against a real daemon, and vice versa.

## Documentation

- [`README.md`](README.md) — running the server, the `?m=`/`&spawncount=`/
  `&cycle=` query params, and the full file layout.
