# HerdWise competition edition — livestock decision support

A deployable six-agent farm-management application for a hackathon. It combines animal, health, nutrition, breeding, inventory, production and financial records into a human-reviewed action plan.

## New in this edition
- Decision lab with per-animal and aggregate next-seven-day forecasts, a recent-average baseline, indicative ranges, temporal holdouts and coverage gates.
- Two editable what-if scenarios with visible milk-response assumptions, saleable fraction, prices and action costs.
- Priority-first budget allocation with editable quotes, unknown-cost handling and urgent funding-gap flags.
- Evidence-linked individual watchlist for health observations, withholding, yield decline and breeding follow-up.
- Server-persisted frozen forecast decisions, live actual scoring, daily error, action follow-up and JSON exports.
- Reviewed voice queries with English, Hindi and Tamil language selection and typed fallback.
- A 90-day synthetic dataset plus date regeneration, historical replay and Windows instructions in **DEMO_WALKTHROUGH.md**.

## Quick demo
1. Open the private deployed app and select **Load demo farm** on an empty workspace.
2. Ask **Milk production has decreased this week** and click **Investigate farm**.
3. Review six specialist reports and the collaboration trace.
4. Open **Action plan**, review evidence, and mark an action **In progress** or **Done**.
5. Export the plan or a farm JSON backup. Add or edit records and investigate again.

Demo data is synthetic. Four cows produce 560 L in the previous seven days and 448 L in the current seven days: a 20% decline. Current recorded milk revenue is INR 16,464, expenses INR 6,500, recorded net INR 9,964. A feed purchase estimate is INR 5,824. These are scenario calculations, not forecasts.

## What is implemented
- Six separately scoped agents: Health, Nutrition, Breeding, Finance, Inventory and Farm Manager.
- Parallel first-wave specialist analysis, dependency messages, finance budget review, manager reconciliation and urgent-first priorities.
- Server-persisted farm records, investigations and task progress in Cloudflare D1.
- Validated create, edit, delete and JSON import; linked animal tags; duplicate daily production rejection.
- Milk trends, recorded cash metrics, stock coverage and expiry alerts, withdrawal reminders, calving and pregnancy checks.
- Owner, due date, priority, estimated cost, evidence IDs and action status.
- Collaboration trace, missing-data disclosures, Markdown plan export, JSON backup, print and history.
- Responsive desktop/mobile interface and an in-app guide.

## AI modes
**Default: Rules-based multi-agent.** No API key is needed. Separate specialist functions use explicit checks; they analyze all domains for every query. This mode does not understand arbitrary natural-language intent. It does not claim a diagnosis or understand arbitrary query intent. The separate Decision lab uses simple statistical production forecasts.

**Optional: Rules + language model collaboration.** Five specialist model calls run in parallel over domain-specific records and rules. Their notes are passed to a sixth Farm Manager model call. The rules remain authoritative for safety and structured action tracking. Model suggestions are displayed as unverified interpretation, never executed automatically. A partial or total provider failure retains the rules-based plan.

Configure server-only environment variables:
- `FARM_AI_KEY`: provider API secret.
- `FARM_AI_URL`: HTTPS OpenAI-compatible chat completions endpoint, for example `https://api.openai.com/v1/chat/completions` for that provider.
- `FARM_AI_MODEL`: a model supported by your provider.

No credential goes in the browser or repository. Use the hosting environment settings or Wrangler secrets. This optional path is implemented but has not been exercised against a paid live provider. Each investigation uses up to six model calls. Specialist prompts include at most 150 domain records, and outputs are limited to 700 tokens per call. Prompt contents include farm observations and the farmer query: obtain appropriate consent before sending real farm data to a provider.

## Stack and source map
React 19, TypeScript, Vinext/Vite, Cloudflare Workers, Cloudflare D1, Drizzle migrations, Zod validation and Lucide icons.

- `app/page.tsx`: dashboard, records, investigations, plans, history and guide.
- `app/globals.css`: responsive theme and print styles.
- `lib/model.ts`: typed entities and input schemas.
- `lib/fields.ts`: all seven form definitions.
- `lib/agents.ts`: deterministic specialists, collaboration and metrics.
- `lib/ai.ts`: optional model specialist calls and manager synthesis.
- `lib/storage.ts`: prepared D1 queries and origin checks.
- `lib/demo.ts` and `demo-farm.json`: reproducible synthetic example.
- `app/api/farm/route.ts`: CRUD, import and demo data.
- `app/api/investigate/route.ts`: orchestration and saved investigations.
- `app/api/tasks/route.ts`: action status updates with conflict detection.
- `lib/decision.ts`: forecasting, scenarios, budget allocation, watchlist and outcome scoring.
- `components/decision-lab.tsx`: decision workflow and forecast chart.
- `components/voice-query.tsx`: browser speech recognition with transcript review.
- `app/api/decisions/route.ts`: validated, server-derived frozen decision snapshots.
- `scripts/setup-db.mjs`: idempotent local database setup for fresh installs and upgrades.
- `db/schema.ts` and `drizzle/`: database schema and migrations.
- `tests/`: meaningful calculation and built-worker integration tests.

## Local setup
Requires Node >=22.13 and pnpm matching the `packageManager` in package.json. Use `corepack enable` if your Node distribution includes Corepack, then `pnpm install --frozen-lockfile`.

```sh
pnpm install --frozen-lockfile
pnpm build
npm run db:setup
pnpm start
```

The production-like local server uses Wrangler; its console prints the URL. This local deployment has no independent application authentication. Do not expose it to the internet without access protection. To run the dev server use `pnpm dev`; ensure its D1 state has the schema. Schema changes use `pnpm db:generate`, followed by application of newly generated migrations.

## Standalone Cloudflare deployment
The existing private hosted app is already published through Sites. For your own Cloudflare account:
1. Install dependencies and build as above.
2. Authenticate with `pnpm exec wrangler login`.
3. Create a D1 database using `pnpm exec wrangler d1 create herdwise-db`.
4. Copy `dist/server/wrangler.json` to `dist/server/wrangler.deploy.json`. Keep it beside index.js so paths remain correct. Set a unique Worker name and replace the placeholder D1 database_name and database_id with your actual values. Remove the local-only CONNECTORS service if present. The source/build contain no external workspace connector requirements.
5. Apply both SQL migrations (0000 then 0001): `pnpm exec wrangler d1 execute herdwise-db --remote --config dist/server/wrangler.deploy.json --file drizzle/0000_sharp_hobgoblin.sql`.
6. Optional AI: `pnpm exec wrangler secret put FARM_AI_KEY --config dist/server/wrangler.deploy.json`; set `FARM_AI_URL` and `FARM_AI_MODEL` in the deployment config's vars or as secrets.
7. Deploy using `pnpm exec wrangler deploy --config dist/server/wrangler.deploy.json`.
8. Protect the hostname with Cloudflare Access before entering real farm data. This version is one private farm workspace; it has no built-in multi-tenant identity or role separation.

Rebuilding regenerates dist. Recreate the deployment config after every build, or automate copying the correct binding values. Back up D1 and JSON exports. Never deploy the placeholder database ID. Apply `drizzle/0001_decision_runs.sql` to the remote database as well before using Decision lab. Standalone deployment steps are supplied; deployment to a separate user-owned Cloudflare account was not executed.

## Checks
```sh
node node_modules/typescript/bin/tsc --noEmit
npm test
pnpm build
node tests/api.test.mjs
```

The API test uses the built Worker and isolated ephemeral D1 through Miniflare. It checks saved state, the demo conflict, investigation, task status persistence, record updates, animal linkage, invalid milk volumes and origin rejection. No farm data is deleted by this test.

## API
- `GET /api/farm`: records and latest 50 investigations.
- `POST /api/farm`: `{kind,data}`, `{operation:"update",id,kind,data}`, `{operation:"demo"}`, or `{operation:"import",records:[{kind,data}]}`.
- `DELETE /api/farm`: `{id}`; animal deletion is blocked while linked observations exist.
- `POST /api/investigate`: `{query}` (5–2000 characters).
- `GET /api/decisions`: latest 100 saved decision snapshots.
- `POST /api/decisions`: `{name,asOf,scenario,budget,quotes,planId,notes}`; forecast is always recomputed server-side.
- `PATCH /api/tasks`: `{planId,actionId,status}`; status is Open, In progress or Done.

## Important practical limits
This is a tested hackathon MVP, not a clinically validated production livestock system. Rule thresholds target dairy cattle; registering other species does not validate thresholds for them. It does not calculate complete nutritional requirements, diagnose disease, predict conception, dispense drugs, place orders, ingest live sensors, notify by SMS, or optimize farm profit. Financial totals depend on entered records; no cash balance, debts or accrual accounting is inferred. Missing production dates are not treated as zero. Coverage counts recorded dates; it does not prove every animal was recorded. Health histories and notes remain in saved findings even if a source record is later removed; this is not a legally compliant immutable audit trail. Imports append records, assign fresh IDs and reject tag/production duplicates; they do not restore investigation history.

Forecasts require one recorded daily total for every included lactating animal over seven consecutive dates. Missing animals are excluded and farm totals are labeled partial. Forecasts use at most 180 contiguous daily observations and ignore dates after the chosen reference date. Damped-trend daily predictions are bounded between zero and 130% of the recent daily average. With at least three earlier weekly holdouts, the model with lower weekly absolute error is selected; otherwise the average baseline is used. Reported historical error also uses those selection holdouts, so it can be optimistic. Indicative ranges sum per-animal daily spreads from historical residual RMSE (or recent standard deviation), with a 5% floor. They are not calibrated probability intervals. Only real-farm testing can establish practical forecast accuracy.

Scenario response assumptions are user-entered and start at zero; the illustrative preset is not a learned estimate. Budget allocation is priority-first greedy fitting of known costs, not a profit optimizer. Urgent quote/funding gaps remain visible even when lower-priority actions fit. Scenario extra costs and action-plan budgets are independent planning views; avoid counting the same costs twice. Saved budget snapshots exclude already completed actions. Recorded outcome totals require seven complete dates for the saved set of animal tags, regardless of subsequent life-stage changes. Scores update after record edits; saved forecasts do not. Historical replay is a development evaluation, not proof that interventions caused a yield change.

Voice input depends on browser recognition support, microphone permission and localhost/HTTPS. Hindi/Tamil transcription is supported through browser language selection; rules-based investigations still run the same all-domain checks and do not understand arbitrary translated questions. Audio may be processed by the browser's speech provider. The application never auto-submits a voice query.

For farm rollout add farmer authentication and roles, tenant isolation, rate limits, database uniqueness for concurrent ingestion, transactional stock movements, per-animal coverage checks, record provenance, species-specific expert-approved thresholds and field validation. The current UI date reporting uses UTC for daily periods. Budget for model usage and data retention. See the project report for requirements coverage and evaluation guidance.

## Clinical reference context
These sources support general screening context, not validation of this application's rules:
- University of Minnesota Extension: https://extension.umn.edu/agriculture/animals-and-livestock/dairy/mastitis-screening
- Penn State Extension: https://extension.psu.edu/dry-cow-heat-stress-abatement
- Penn State Extension: https://extension.psu.edu/heat-stress-abatement-techniques-for-dairy-cattle

Rule policy: THI >=68, dairy-cattle body temperature >=39.5 °C, intake/offered <90%, stock cover <= supplier lead +2 days, 14-day calving window, 30-day pending-pregnancy reminder and production decline >=10% are screening/planning heuristics. Only the THI context is tied directly to the cited extension guidance. All thresholds need farm-specific expert review.
