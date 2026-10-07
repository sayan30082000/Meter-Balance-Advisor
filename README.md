# Meter Balance Advisor

A prepaid electricity meter advisor that runs entirely in the browser. One HTML file, no build step, no dependencies, no network calls — open `index.html` and it works offline.

**Live demo:** hosted on Netlify (link added after the first deploy)

## What it does

The page carries daily meter readings and recharge events for 25 households (`PUB-01` … `PUB-25`) and reconstructs each one's balance day by day under a Bangladeshi-style monthly cumulative slab tariff.

- **Editable tariff.** Six cumulative slabs, demand charge, meter rent and VAT are all inputs. Change any figure and every balance, bill, chart and comparison recomputes instantly across all 25 households. A calibration line shows total computed cost against the households' actual recharges, so you can tell whether an assumed tariff is realistic.
- **Reconstructed balance.** Each day's units are billed at the slab the month's running total has reached; demand charge and meter rent are deducted on the first recharge of each month; VAT rides on top. A canvas chart draws the history, marks every recharge, and projects forward at the usual daily use until the balance hits zero.
- **Two questions.** When does the balance run out at this daily use? And how much must be recharged today to last until a target date — broken into base-slab energy, higher-slab uplift, fixed charges and VAT.
- **Habit comparison.** Same consumption, same opening balance, only the recharge timing differs: topping up whenever the balance dips below a threshold, versus topping up on the 1st. Cost is what the meter *consumes* — energy + VAT + the fixed charges that actually fire — never what is deposited.
- **Day-by-day ledger** with CSV and JSON export, light/dark themes, arrow-key navigation between households, and deep links (`index.html#PUB-07`).

## Method

The billing engine is a faithful client-side port of the original Python implementation (`meter.py`), cross-checked to the paisa across 475 assertions with zero mismatches. Energy is charged marginally across cumulative monthly slabs — the counter resets on the 1st of each month.

Cost is defined as energy + VAT + applicable monthly fixed charges, not the amount deposited. Two recharge habits costing exactly the same is a legitimate result: it means both triggered a first-recharge fixed charge in every month, so the only lever — how many months fire that charge — never moved.

The tariff is an explicit, editable assumption. The dataset ships without one.

## Structure

```
index.html    the whole application: styles, engine, data, UI
```

Inside it, in order: the CSS, the pure billing engine (`window.P10` — `rebuild`, `project`, `item1`, `item3`, `item4`), the dataset as an inline `application/json` block, and the rendering layer.

## Running it

```bash
git clone https://github.com/sayan30082000/Meter-Balance-Advisor.git
cd Meter-Balance-Advisor
open index.html          # or: python3 -m http.server 8000
```

## Using your own data

Replace the JSON inside `<script id="rawdata">`. Each case takes:

| field | meaning |
| --- | --- |
| `id` | household label shown in the selector |
| `opening` | balance in BDT on the first day |
| `start` | first date, `YYYY-MM-DD` |
| `units` | daily consumption, one entry per day from `start` |
| `rec` | recharge events, `[date, amount]` |
| `today` | the day the reconstruction stops and the projection begins |
| `usual` | default daily units for the forecast |
| `target` | default target date for the top-up question |
| `cmp` | habit-comparison window: `months`, `opening`, `low_thr`, `low_amt`, `mon_amt` |
