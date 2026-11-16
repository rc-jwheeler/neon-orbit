# Neon Orbit (prototype v1)

A quiet, neon, space-themed transit builder. Draw glowing conduits between geometric stations; little capsule ships carry each cargo pod to a station of the **same shape**. Let one station get too crowded for too long and the run ends.

**Play:** https://rc-jwheeler.github.io/neon-orbit/  ·  pick a map with `?seed=123` (e.g. https://rc-jwheeler.github.io/neon-orbit/?seed=42)

Single file (`index.html`), Canvas 2D, no build step, no backend.

![Gameplay](preview.png)

## Controls (touch or mouse)

| Action | How |
|---|---|
| New line | Drag from a station across other stations, release. 5 lines to start, +1 every 40 deliveries (up to 7). |
| Extend a line | Drag the small round **knob** at either end of a line onto more stations. Drag back over a station to un-add it. |
| Delete / redraw | **Long-press** a line (or tap its ⚡% chip at the top), then **✕ Delete line**. Redraws are free. |
| Boost a conduit | **Tap a line** (or ⚡ then tap a line). Uses one charge: +50% energy. |
| Stabilize a phase station | Tap **❄**, then tap a station with a ring timer: its shape is frozen for 10 s. |
| Undo last line edit | **↶** button or **Ctrl/Cmd+Z** |
| Pause / speed | **❚❚** (or Space) · speed button cycles **Chill (0.5×) / 1× / 2× / 3×**. Chill is the default; your choice is remembered. |

The simulation always pauses while you are drawing or have a line selected (dashed amber frame + "EDITING · PAUSED").

## Rules

- **Stations** appear on a seeded map (circle, triangle, square; diamond later): every 30 s at first, ramping gently (1 s faster per minute) to every 18 s. Each holds 8 waiting pods; above that a red ring fills and after 25 s over capacity the run ends (it drains back down when the crowd clears).
- **Pods** route over the whole network (Dijkstra over stations × lines, with a transfer penalty). They board only a ship heading toward their next hop and change lines at transfer stations. Routes are recomputed instantly on any edit, phase change or new station.
- **Energy conduits:** every line shows its energy as glow brightness, a bright core that fills the line in proportion to energy, and a % on its start knob and HUD chip. Energy drains while its ship moves (more on longer lines) and a little per pod moved, and slowly recharges (+0.3%/s) whenever the ship carries 2 pods or fewer. Below 50% the ship slows (down to 30% speed) and the line shows red dashes. Boost charges: start with 4, +1 every 4 deliveries and every 30 s (max 8).
- **Phase stations** flip between two shapes on a ring timer (a small ghost shows the next shape). Pods heading there flash red in the last 3 s; on the flip, pods re-path immediately (and pods already waiting there that now match are delivered). Stabilizers: start with 2, +1 every 12 deliveries (max 4).
- Score = pods delivered.

## Notes

- **Deterministic:** seeded PRNG (mulberry32) and a fixed 60 Hz timestep with an accumulator. The same seed + same edits at the same sim tick give the same run. Station layout is generated in unit space, so topology and timing match across screens; absolute positions stretch to the screen's aspect.
- `window.__dbg` exposes a small test hook (`ff(seconds)`, `step(n)`, `addLine([ids])`, `route(stationId, shape)`, screen coordinates of stations/knobs).
- Balance is still a guess, tuned once from the first playtest (v1.1 pacing pass):

  | | v1 | v1.1 |
  |---|---|---|
  | Station spawn | every 15 s | 30 s → 18 s (−1 s/min) |
  | Pod spawn per station | 6–12 s × max(.45, 1 − t/400) | 10–18 s × max(.55, 1 − t/900) |
  | Energy drain (moving) | .003 + .006·len/500 per s | .0015 + .003·len/500 per s |
  | Drain per pod moved | .004 | .002 |
  | Passive regen | none | +.003/s when ship carries ≤ 2 pods |
  | Boosts | start 3, +1 / 6 deliveries, +1 / 45 s, max 6 | start 4, +1 / 4 deliveries, +1 / 30 s, max 8 |
  | Station capacity / overcrowd timer | 6 / 14 s | 8 / 25 s |
  | Ship speed | 78 | 72 |
  | Lines | 5 | 5, +1 per 40 deliveries, max 7 |
  | Default speed | 1× | Chill (0.5×) |

  A scripted test player (greedy extend-nearest network, boosts lines under 35%) on 10 seeds lasted a median of ~137 s of sim time in v1 and ~310 s in v1.1 (about 10 min of real time at Chill).
- Out of scope for v1 on purpose: orbits, wormholes, extra ships/carriages, loops, scoring meta.
