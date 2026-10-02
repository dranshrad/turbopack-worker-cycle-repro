### Describe the bug

`packages/tools/src/workers/computeWorker.ts` imports from `../utilities/segmentation` (and `getSegmentLargestBidirectional` directly). That index re-exports `getStatistics`, `getSegmentLargestBidirectional`, `computeMetabolicStats`, which import `../registerComputeWorker` — the module that does `new Worker(new URL('../workers/computeWorker.js', import.meta.url), { type: 'module' })`. The worker's own module graph therefore contains its spawner.

Webpack tolerates this. Turbopack (Next.js 16.2.10) never finishes a build that includes `@cornerstonejs/tools` 5.11.3: `next build` sits at 0 % CPU with no error. Bisected: `@cornerstonejs/core` alone builds in 43 s; core + tools hangs; tools with `registerComputeWorker.js` stubbed to a no-op builds.

Minimal standalone reproduction (five files, no Cornerstone): <PASTE the repro repository URL>.

### Steps to reproduce

1. Next.js 16.2.10 app, a client component that imports `{ init, addTool, StackScrollTool } from '@cornerstonejs/tools'`.
2. `next build` (Turbopack) → hangs.
3. Replace `node_modules/@cornerstonejs/tools/dist/esm/utilities/registerComputeWorker.js`'s worker factory with a no-op → builds in ~12 s.

### Suggested fix

Have `computeWorker.ts` import the leaf utilities it needs directly (not the `segmentation` index), or move `registerComputeWorker` out of the graph the worker imports, so the worker bundle no longer contains its own spawner.

### Version

@cornerstonejs/tools 5.11.3 (also core 5.11.3), Next.js 16.2.10, Node 26.5.0, macOS 27 arm64.
