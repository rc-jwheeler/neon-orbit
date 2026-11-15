# Neon Orbit (prototype v1)

A quiet, neon, space-themed transit builder. Draw glowing conduits between geometric stations; little capsule ships carry each cargo pod to a station of the **same shape**. Let one station get too crowded for too long and the run ends.

**Play:** https://rc-jwheeler.github.io/neon-orbit/  ·  pick a map with `?seed=123` (e.g. https://rc-jwheeler.github.io/neon-orbit/?seed=42)

Single file (`index.html`), Canvas 2D, no build step, no backend.

![Gameplay](preview.png)

## Controls (touch or mouse)

| Action | How |
|---|---|
| New line | Drag from a station across other stations, release. Up to 5 lines. |
| Extend a line | Drag the small round **knob** at either end of a line onto more stations. Drag back over a station to un-add it. |
| Delete / redraw | **Long-press** a line (or tap its ⚡% chip at the top), then **✕ Delete line**. Redraws are free. |
| Boost a conduit | **Tap a line** (or ⚡ then tap a line). Uses one charge: +50% energy. |
| Stabilize a phase station | Tap **❄**, then tap a station with a ring timer: its shape is frozen for 10 s. |
| Undo last line edit | **↶** button or **Ctrl/Cmd+Z** |
| Pause / speed | **❚❚** (or Space) · **1×/2×/3×** |

The simulation always pauses while you are drawing or have a line selected (dashed amber frame + "EDITING · PAUSED").

## Rules

- **Stations** appear every 15 s on a seeded map (circle, triangle, square; diamond later). Each holds 6 waiting pods; above that a red ring fills and after ~14 s over capacity the run ends.
- **Pods** route over the whole network (Dijkstra over stations × lines, with a transfer penalty). They board only a ship heading toward their next hop and change lines at transfer stations. Routes are recomputed instantly on any edit, phase change or new station.
- **Energy conduits:** every line shows its energy as glow brightness, a bright core that fills the line in proportion to energy, and a % on its start knob and HUD chip. Energy drains while its ship moves (more on longer lines) and a little per pod moved. Below 50% the ship slows (down to 30% speed) and the line shows red dashes. Boost charges: start with 3, +1 every 6 deliveries and every 45 s (max 6).
- **Phase stations** flip between two shapes on a ring timer (a small ghost shows the next shape). Pods heading there flash red in the last 3 s; on the flip, pods re-path immediately (and pods already waiting there that now match are delivered). Stabilizers: start with 2, +1 every 12 deliveries (max 4).
- Score = pods delivered.

## Notes

- **Deterministic:** seeded PRNG (mulberry32) and a fixed 60 Hz timestep with an accumulator. The same seed + same edits at the same sim tick give the same run. Station layout is generated in unit space, so topology and timing match across screens; absolute positions stretch to the screen's aspect.
- `window.__dbg` exposes a small test hook (`ff(seconds)`, `step(n)`, `addLine([ids])`, `route(stationId, shape)`, screen coordinates of stations/knobs).
- Balance (spawn rates, drain, capacity, timers) is a first guess.
- Out of scope for v1 on purpose: orbits, wormholes, extra ships/carriages, loops, scoring meta.
