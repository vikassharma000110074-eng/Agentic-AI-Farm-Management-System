# HerdWise competition edition — five-minute demo

## Run on Windows (PowerShell)
Extract this ZIP, open the **herdwise** folder in VS Code and open its terminal. `dir package.json` should find the file. Node.js 22.13 or newer is required.

Run each command separately and wait for it to finish:

```powershell
npm install -g pnpm@11.25.0
pnpm install --frozen-lockfile
pnpm build
npm run db:setup
pnpm start
```

Open the URL printed by Wrangler; keep the terminal running. On subsequent launches, run `pnpm start`. If you edit source, build again first.

**Updating an existing local farm:** copy your old `herdwise/.wrangler/state` folder into the new `herdwise/.wrangler/state` before running `npm run db:setup`. Keep a copy of the old project and export a farm JSON backup first. The setup command adds missing tables and retains existing records. Do not import the backup into an already populated database: imports append records. A JSON backup contains records, not saved investigations or decisions; copying the local state retains all three. Never copy a secret environment file into a shared ZIP.

## Demo dataset
Use **Farm records → Import JSON** and select `competition-demo-90-days.json` from this folder. It contains 379 synthetic records, including four HW2-tagged cows and 90 consecutive production dates, with a recent decline, recorded health and milk-withholding concerns, water interruption, feed refusal, calving, pregnancy confirmation and stock alerts.

Import it once. HW2 tags avoid the old C001–C004 demo, but if both datasets are present totals combine. Use a fresh database for the cleanest presentation. The supplied dataset ends on 9 October 2026. Regenerate it for your presentation date with `npm run demo:generate`, then import the regenerated file into a fresh workspace. Regeneration writes a JSON file; it does not change your database.

## Presentation sequence
1. **Investigation:** ask “Milk production has decreased this week. What should we investigate first?” Run the investigation. Show six agents, evidence and collaboration trace. Default mode is rules-based; optional server AI settings remain available.
2. **Decision lab:** show per-animal and combined forecasts, the average baseline, indicative range and held-out historical error. Four animals should be ready after importing the 90-day dataset. Explain that these are synthetic development results, not field validation.
3. **What-if simulator:** click **Use illustrative demo assumptions**. Compare scenario A (intake, water and cooling) with B (cooling only). Adjust prices, saleable fraction and extra cost. Read the assumptions aloud. The gain is illustrative, not an intervention-effect estimate learned from the records.
4. **Budget:** enter ₹3,000, review urgent funding gaps and enter veterinary quotes. Raise the budget to ₹8,000 and show which actions fit. Unknown quotes never count as free actions. Clinical and milk-safety actions still need human review.
5. **Watchlist:** show animal-level evidence for reduced yield, health, withholding and upcoming calving.
6. **Outcome tracking:** set the reference date to **2026-10-02** for the supplied file (or seven days before the regenerated dataset's end date). Add decision notes and save scenario A. This is a **historical replay**: the model uses records through the chosen date, and the later seven recorded days populate actual results. Show daily error and the complete week total. This demonstrates prediction followed by scoring without inventing future actuals.
7. Mark an action **In progress** or **Done** in Action plan. Return to Decision lab to show the linked follow-up status, refresh the page to demonstrate saved history, and export the decision JSON with its forecast, assumptions, budget, quotes and observed outcome.
8. **Voice:** use a supported Chrome/Edge browser on localhost/HTTPS. Select English, Hindi or Tamil, click **Speak a question**, review the transcript, then click **Use reviewed question**. Investigation only runs when you click its normal button. Microphone permission and browser/provider support are required. Speech recognition may send audio to the browser's service; typing remains available.

## What to tell judges
“HerdWise connects farm observations to a reviewed action plan, compares transparent planning scenarios, checks budget feasibility and measures saved forecasts against later records.”

The additional standout capabilities are a data-readiness gate, temporal backtesting, historical replay, evidence-linked animal signals, frozen server-persisted decisions and an exportable decision record. Production forecasting is a simple baseline/damped-trend model, not a trained clinical or causal AI system. Scenario contribution is extra sale revenue less entered action cost; it is not full farm profit. No automatic purchases, clinical actions or external messages occur.
