# Hue Stack

An original one-finger color-sorting puzzle with a "drip" pressure twist. Built as a single self-contained `public/index.html` (HTML + CSS + JS, no external libraries, no network requests), designed for browser game portals (Poki, CrazyGames, itch.io) and future submission to YouTube Playables.

Play it locally by opening `public/index.html` in any modern browser, or visit the deployed URL: **https://hue-stack.u10111012.workers.dev**

The game lives in `public/` so that `wrangler deploy` (Cloudflare) only ever uploads that one file — it never publishes this repo's `.git` history or other project files.

## How to play

Tap a tube to lift its top same-color group, then tap a destination tube to move it there (destination must be empty or topped with the same color, with enough free space). Fill a tube with 4 matching blocks to clear it and score. Every few moves a random new block "drips" into an open tube, forcing you to keep sorting — the game ends when every tube is full and no move is possible.

## Tuning: the `CONFIG` object

All game-balance numbers live in one `CONFIG` object at the top of the script in `public/index.html`. Values were tuned using a standalone Node.js simulation (200 random-play + semi-smart-bot runs) so that the early game is forgiving and a typical run lasts about one to two minutes:

| Key | Value | Purpose |
|---|---|---|
| `TUBE_CAPACITY` | 4 | Blocks per tube before it's "full" |
| `START_FILLED_TUBES` / `START_EMPTY_TUBES` | 4 / 2 | Initial board: 6 tubes total, 2 free for maneuvering |
| `START_COLOR_COUNT` | 4 | Colors present at game start |
| `COLOR_STAGE2_COUNT` / `COLOR_STAGE2_SCORE` | 5 / 250 | A 5th color enters the drip pool once score ≥ 250 |
| `COLOR_STAGE3_COUNT` / `COLOR_STAGE3_SCORE` | 6 / 900 | A 6th color enters once score ≥ 900 |
| `DRIP_INTERVAL_START` / `DRIP_INTERVAL_MIN` | 6 / 2 | Moves between drips; shrinks as score rises |
| `DRIP_DECREASE_SCORE_STEP` | 150 | Every 150 score, the drip interval tightens by 1 (floored at the minimum) |
| `SCORE_PER_BLOCK` | 25 | Points per block cleared (100 per completed tube before combo) |
| `COMBO_WINDOW_MOVES` | 3 | Completions within 3 moves of each other chain the combo multiplier |
| `UNDO_LIMIT` / `SHUFFLE_LIMIT` | 2 / 1 | Free uses per run (stubbed as rewarded-ad-gated) |

To make the game easier, raise `START_EMPTY_TUBES`, raise `DRIP_INTERVAL_START/MIN`, or raise the color-unlock thresholds. To make it harder, do the reverse, or lower `DRIP_DECREASE_SCORE_STEP`.

### Self-test results

- **Unit tests** (legal-move validation, multi-block moves, tube completion detection, drip-never-targets-a-full-tube, game-over detection): 16/16 passing, run with a standalone Node script mirroring the in-file logic exactly.
- **200-run random/bot simulation** with the shipped `CONFIG`: average run length ≈ 115–125 moves, only ~1% of runs end before move 20 (i.e. the early game is forgiving as intended), and no run hit the simulation's safety cap — the game reliably reaches a genuine game-over state rather than running forever.
- **Viewports checked**: 360×640 / 375×812 (mobile, tubes wrap to two rows), 768×1024 (tablet, all 6 tubes fit one row). No console errors in any of the above.

### Known limitations

- Layout and interaction were verified with the Chromium-based preview browser and its accessibility/DOM inspection, not on physical iOS/Android hardware — real-device touch latency and Safari-specific quirks are untested.
- The "smart" bot used for balancing is a simple greedy heuristic (prefers moves that complete a tube), not a human — real players will likely last longer on average since they can plan several moves ahead.
- `Platform.showRewardedAd()` / `showInterstitial()` are stubs that resolve immediately; no real ad SDK is wired in yet.

## Future improvement ideas

1. Daily seeded challenge mode (same board for everyone each day, with a shareable score).
2. A second original mechanic layered on top of drip, e.g. a "freeze" block that must be tapped twice before it can move.
3. Wire `Platform` to the real YouTube Playables SDK (`firstFrameReady`, `gameReady`, `loadData`/`saveData`, rewarded ads) once accepted into the program.

## Deployment

Deployed to Cloudflare Workers (static assets) at **https://hue-stack.u10111012.workers.dev**, serving only the `public/` directory — no build step required. To redeploy after a change:

```bash
npx wrangler deploy
```

To instead auto-deploy on every push, connect this GitHub repo in the Cloudflare dashboard (Workers & Pages → Create → connect to Git), with build output directory set to `public`.
