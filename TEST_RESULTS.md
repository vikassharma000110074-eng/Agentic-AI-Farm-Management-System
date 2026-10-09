# Competition edition verification — 9 October 2026

- Production build: passed using Node 24.19 and the locked pnpm dependencies.
- TypeScript no-emit check: passed.
- ESLint for new decision and voice modules: passed.
- Existing specialist-agent calculation tests: passed.
- Forecast/scenario/budget/watchlist/outcome tests: passed. Includes incomplete days, duplicate/future records, zero yield, held-out weeks, temporal leakage checks, unknown/zero quotes, urgent budget gaps and complete-only outcome scoring.
- React server render checks: passed for populated and empty Decision lab, accessible chart, all feature panels and the speech-recognition fallback.
- Built Worker + isolated D1 integration tests: passed for farm records, investigations, action status, saved decisions, server-generated snapshots, immutable forecast history after source edits, validation and origin checks. The full 379-record demo imports through the validated API; a duplicate import is rejected.
- Local database setup: passed for the original tables and the added decision table.

Limits: interactive browser layout/hydration and microphone/provider recognition were not verified in this environment. The cloud browser could not reach the localhost preview. Synthetic historical tests do not establish real-farm forecast accuracy. The optional external language-model provider was not exercised. Windows instructions use cross-platform Node scripts, but were not run on an actual Windows device.

Reproduce from the herdwise folder:

```sh
pnpm install --frozen-lockfile
node node_modules/typescript/bin/tsc --noEmit
npm test
node node_modules/eslint/bin/eslint.js components/decision-lab.tsx components/voice-query.tsx lib/decision.ts app/api/decisions/route.ts
pnpm build
node tests/api.test.mjs
npm run db:setup
```

The integration tests use an isolated ephemeral database; they do not delete your farm records. No new dependency packages were added.

---

Original edition verification follows for historical context:

# HerdWise verification results

Executed 9 October 2026 against this source and the built Cloudflare Worker.

| Check | Result |
| --- | --- |
| TypeScript noEmit | Passed |
| Production Vinext Worker build | Passed |
| Six-agent calculation and policy suite | Passed |
| Complete demo weekly change | Passed: 560 L to 448 L, -20% |
| Finance totals and stock estimate | Passed: INR 16,464 revenue, INR 6,500 expenses, INR 9,964 net, INR 5,824 purchase |
| Missing dates, zero previous output, future observations | Passed: no fabricated comparable trend |
| D1 built-worker API integration | Passed |
| Record creation and demo duplicate rejection | Passed |
| Persistent investigation and task status | Passed |
| Record edit stored in D1 | Passed |
| Unknown animal and sold-over-produced rejection | Passed |
| Cross-origin mutation rejection | Passed |
| Live paid-provider model calls | Not performed; optional integration requires credentials |
| Browser visual/interaction QA | Not performed; supported browser-testing capability unavailable |
| Standalone deployment to user's Cloudflare account | Not performed; complete instructions included |
| Field or clinical validation | Not performed; expert review required |

The API suite intentionally sends invalid data; associated validation log messages are expected. Its database is ephemeral and isolated from the deployed farm. Tests do not demonstrate improved productivity, diagnostic accuracy or real-world financial outcomes.
