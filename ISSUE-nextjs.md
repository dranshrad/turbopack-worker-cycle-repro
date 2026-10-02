### Link to the code that reproduces this issue

<PASTE the repro repository URL — contents of docs/viewer/repro/turbopack-worker-cycle>

### To Reproduce

1. `npm install`
2. `npm run build` (Turbopack, the default) → prints "Creating an optimized production build …" and never returns (0 % CPU, no error; killed after 180 s).
3. `npm run build:webpack` → finishes in ~5 s.
4. Edit `lib/work.js` so it no longer imports `lib/spawn.js` (break the cycle) → `npm run build` finishes in ~1 s.

### Current vs. Expected behavior

The app has a worker spawned with `new Worker(new URL('./worker.js', import.meta.url), { type: 'module' })` in `lib/spawn.js`. The worker (`lib/worker.js`) imports `lib/work.js`, which imports `lib/spawn.js` — so the worker's module graph contains the module that spawns it. A client component imports `lib/work.js`.

Current: `next build` with Turbopack hangs silently.
Expected: either a build, or an error naming the cycle across the worker boundary.

Real-world instance: `@cornerstonejs/tools` 5.11.3 (`workers/computeWorker.js` → `utilities/segmentation/index.js` → `utilities/registerComputeWorker.js`, which spawns `computeWorker.js`) — any Next 16.2 app importing that package cannot be built with Turbopack without patching the package.

### Provide environment information

See `next-info.txt` in the repository (macOS 27 arm64, Node 26.5.0, Next 16.2.10).

### Which area(s) are affected?

Turbopack

### Which stage(s) are affected?

next build (local), Vercel (Deployments)

### Additional context

A panic log is not written; the process simply never completes. Webpack handles the same graph.
