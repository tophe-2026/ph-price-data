# ph-price-data

Shared price file for **mitsubishipromos.ph/fuel-tracker.html** and **bydpromos.ph/ev-charging-cost.html**.
Both pages and their share images read `prices.json` from this repo, so updating this one file updates both sites.

- Page URL for the data: `https://raw.githubusercontent.com/tophe-2026/ph-price-data/main/prices.json`
- Changes show on the sites within ~5 minutes (GitHub's cache).
- This repo must stay **public** (it only contains public price data).

---

## How to update (rules for the scheduled updater and for manual edits)

Philippine time (Asia/Manila). Only change what actually changed. Never invent numbers: every figure must come from a source found during the run. If a number can't be confirmed, leave the old value and say so in the commit message.

### Fuel (`fuel`), Mitsubishi page

Pump prices change every **Tuesday 6 AM**. Oil firms announce on Monday; industry estimates appear Thursday–Friday.

| When | What to update |
|---|---|
| **Friday / Saturday** (estimate out) | `next`: date of the coming Tuesday (e.g. `"Tuesday, Oct 6"`), `status: "estimate"`, ranges `[low, high]` per liter for diesel, gasoline, kerosene. **Negative = rollback, positive = hike**, e.g. rollback of ₱7–8 → `[-7.00, -8.00]`. Update the last `history` item for that week with the midpoint and `"projected": true` (add a new item if that week isn't there). |
| **Monday evening** (final announced) | `next.status: "final"`, ranges = the DOE/major-oil-firm figure (same number twice if single, e.g. `[1.20, 1.20]`). Update that week's `history` item to the final DOE figure and remove `projected`. |
| **Tuesday** (new week in effect) | `week` (e.g. `"Oct 6 – 12, 2026"`), `averages.diesel/gasoline.price` = Metro Manila average ₱/L (GasWatch PH or DOE), `change` = this week's adjustment (+hike / −rollback), `cheapest` top 3 brands per fuel with ₱/L. Set `next` to the following Tuesday with `status: "estimate"` and ranges `[0, 0]` until estimates come out. Keep `history` to the **last 5 items** (drop the oldest). |

Always set `lastUpdated` (e.g. `"October 6, 2026"`) and `lastUpdatedISO` (`"2026-10-06"`) to the day of the update, and replace `sources` with the URLs used.

Good sources: DOE oil price monitoring, gaswatchph.com (Metro Manila averages + cheapest brands), Rappler / Philstar / GMA / Inquirer / Manila Bulletin fuel price articles.

### Electricity (`electricity`), BYD page

Meralco announces the new monthly rate around the **second week of each month**.

- `home.rate`: Meralco overall rate for a typical 200 kWh household, ₱/kWh, 4 decimals (e.g. `14.7424`).
- `home.change`: difference vs last month (e.g. `-0.0409`).
- `month` (e.g. `"October 2026"`), `lastUpdated`, `lastUpdatedISO`, `sources`.
- `publicAC` / `publicDC`: only change when the DOE publishes new national averages (update `asOf` too).
- `networks`: only change when a network's published rate changes. Keep the home row's `rate` text in sync with `home.rate` (e.g. `"₱15.10"`).

### Checks before committing

1. `prices.json` must be valid JSON (run `python3 -m json.tool prices.json`).
2. Prices must be plausible: diesel/gasoline between ₱40 and ₱200 per liter; weekly change within ±₱20; Meralco between ₱8 and ₱25 per kWh.
3. Commit message: what changed, e.g. `Fuel: week Oct 6–12, diesel ₱98.31 (+1.20), gasoline ₱92.69 (+0.90)`.
