# Graph Paper Racing Design Notes

This document is a compact reference for future agents working on [`race.html`](/Users/mkruk/graphpapergames/race.html).

## Purpose

`race.html` is a self-contained browser game inspired by graph-paper vector racing. Two cars alternate turns on a grid. Each move changes velocity by `-1`, `0`, or `+1` on both axes, then advances by the resulting velocity vector.

## Structure

- UI, rendering, game rules, track generation, and AI all live in one file.
- The board is a `<canvas>` drawn from grid coordinates.
- Sidebar controls configure track type, random generation settings, players, and restart/undo.
- Game state is kept in the global `state` object and prior states are copied into `history` for undo.

## Core State

- `track`: road cells, start line, finish line, starting positions, and progress helpers.
- `state.turn`: index of the active player.
- `state.winner`: winning player index or `null`.
- `state.players[]`: per-player position, velocity, path history, crash status, finish status, move count, lap progress, and AI depth.
- `aiCache`: memoized search scores for the current track and state slice.

## Movement Rules

- A move applies acceleration first, then computes the next position from velocity.
- Ending off-road is always illegal, including near the finish on an open track.
- A move may not end on the opponent's cell.
- A move also may not pass through the opponent's occupied cell anywhere along the segment.
- When `strictPath` is enabled, sampled points along the segment must stay on the road.
- If a player has no legal moves on their turn, they crash.

## Finish Logic

- Circuit tracks use the same line for start and finish.
- Circuit wins require crossing that line after enough forward lap progress has accumulated.
- Open tracks have separate start and finish lines.
- On open tracks, a win only counts on a fully legal move whose segment crosses the finish line while staying on the road.

## Track Generation

- Preset tracks: `oval`, `bottleneck`, `snake`.
- Random tracks support `open` and `circuit` shapes.
- Random styles currently include `smooth`, `hairpin`, `clover`, `chicane`, and `polygon`.
- `withProgress()` computes `distToFinish` for open-track AI heuristics.

## AI

- AI depth comes from the player dropdown (`ai1` through `ai10`).
- `chooseAI()` ranks legal root moves, then deepens with `searchAI()`.
- `searchAI()` is a beam search over legal moves only.
- Open-track evaluation uses `distToFinish`; circuit evaluation uses `lapProgress`.
- Simulated win detection should always match real move resolution. If finish behavior changes, update both `legal()` / `doMove()` and AI helpers together.

## Rendering

- `drawGrid()`, `drawRoad()`, `drawPaths()`, and `drawCandidates()` repaint the full board.
- The winner banner is an HTML overlay, not part of the canvas.
- Player trails and car markers are drawn directly on the canvas.

## Change Hazards

- `legal()` is the main rules choke point. Small changes there can affect UI, human play, AI, and undo behavior.
- `samples()` is reused for road validation, opponent-through-path checks, and finish stopping behavior.
- Finish-line behavior is easy to desync between real moves and AI search if helper functions diverge.
- The file is intentionally standalone, so changes usually need to stay local rather than introducing module infrastructure.
