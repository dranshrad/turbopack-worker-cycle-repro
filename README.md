# Turbopack hangs on a Web Worker whose module graph imports its own spawner

`next build` (Next.js 16.2.10, Turbopack) never returns — 0 % CPU, no error — when a client
component imports a module that (a) spawns a worker with `new Worker(new URL('./worker.js',
import.meta.url), { type: 'module' })` and (b) is itself imported by that worker's module graph.

- `lib/spawn.js` spawns `lib/worker.js`
- `lib/worker.js` imports `lib/work.js`
- `lib/work.js` imports `lib/spawn.js`  ← the cycle across the worker boundary
- `app/page.js` ('use client') imports `lib/work.js`

Reproduce: `npm install && npm run build` (hangs) vs `npm run build:webpack` (finishes).
Real-world instance: `@cornerstonejs/tools` 5.11.3 — `workers/computeWorker.js` →
`utilities/segmentation/index.js` → `utilities/registerComputeWorker.js` → spawns `computeWorker.js`.
