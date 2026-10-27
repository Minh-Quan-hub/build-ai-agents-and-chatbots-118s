# Inventory Replenishment Agent

This Colab project adapts the [AAI2025 inventory replenishment starter notebook](https://github.com/myrah/AAI2025/blob/dev/aai/InventoryReplenishmentAgent.ipynb) into a daily, three-SKU simulation using real retail transaction quantities. The agent forecasts demand, checks inventory and open purchase orders, and explains each order or wait decision. The notebook also compares its enhanced policy with the original EWMA policy on the same data.

## Files

- `inventory_replenishment_agent_colab.ipynb` — runnable notebook, daily agent log, policy comparison, and summary.
- `sales.csv` — 90 calendar days of demand for three SKUs (`date,sku,qty_sold`).
- `inventory.csv` — illustrative starting stock (`sku,opening_stock`).
- `params.csv` — illustrative cost and replenishment inputs (`sku,unit_cost,holding_cost_per_day,stockout_cost,lead_time_days,min_order_qty,service_level`).

## Data and assumptions

The sales come from Daqing Chen's [UCI Online Retail dataset](https://archive.ics.uci.edu/dataset/352/online+retail), 2015, DOI [10.24432/C5BW33](https://doi.org/10.24432/C5BW33), licensed CC BY 4.0. This project sums positive, non-cancelled transaction quantities for product codes `22720`, `22666`, and `22960` from January 1 through March 31, 2011. A day without a recorded transaction is filled with zero, including closed trading days.

UCI supplies sales transactions, **not** historical starting inventory, lead times, procurement costs, stockout costs, or unmet customer demand. `inventory.csv` and `params.csv` are editable example assumptions in USD. The simulation treats recorded sales as the demand that would have occurred even when its modeled stock reaches zero. Its stockouts and fill rates are simulated outcomes, not verified historical results.

## How to run in Google Colab

1. Open `inventory_replenishment_agent_colab.ipynb` in Google Colab.
2. Upload `sales.csv`, `inventory.csv`, and `params.csv` to Colab's **Files** sidebar.
3. Select **Runtime → Run all**. No API key or extra installation is required in a standard Colab Python runtime.
4. Read the 270 daily log lines and the per-SKU and overall summary. The notebook also saves `agent_log.csv` and `agent_summary.csv` for download. Set `PRINT_DAILY_LOG = False` if you only want the summary.

You can adjust the starting inventory and policy values in the two assumptions CSVs and rerun. The notebook aggregates duplicate `date,sku` rows and fills missing calendar dates with zero.

## How the agent works

Each morning, purchase orders due that day arrive. The agent then acts using **only demand observed before that day**:

1. Update a per-SKU exponential weighted moving average (EWMA) after each day's demand. Its smoothing factor is `0.30`.
2. Forecast demand over `lead_time_days + 1` days; the extra day represents the daily review interval.
3. Estimate forecast uncertainty from up to 28 past one-day errors. Once enough history exists, also examine the quantile of rolling lead-time-plus-review-period errors. Use the larger buffer to cover bursts and correlated errors.
4. Set the risk quantile to at least `service_level`. For the enhanced agent, a cost-based floor is `stockout_cost / (stockout_cost + holding_cost_per_day × protection_days)`, capped at `0.999`. This simple cost heuristic raises the buffer when shortages are much more expensive than short-term holding.
5. Compare inventory position (`on_hand + outstanding PO units`) with forecast demand plus safety stock. Order enough to close the gap, at least `min_order_qty`; otherwise wait. Orders arrive at the **start** of the date `order_date + lead_time_days`.
6. Reveal that day's demand, fulfill what is available, record lost units, charge holding cost on closing stock, and update the forecast for the next day.

Fill rate is `fulfilled units / requested units`. Operating cost is `holding cost + stockout cost + fixed PO cost`; the fixed PO cost is `0`. Purchase spend uses `unit_cost` but is reported separately because a purchase may still be on hand or in transit at the end of the simulation. This is a simple planning policy, not a guaranteed service-level or globally optimal inventory solution.

## My contributions to this version

- Replaced the starter's synthetic single-item demand with three real, daily SKU series and separate editable inventory and policy CSVs.
- Implemented a per-SKU EWMA forecast, historical error-based safety stock, cost-aware risk adjustment, and minimum-order rule.
- Tracked open purchase orders and their exact arrival dates, lost sales, fill rate, closing inventory, outstanding end-of-horizon POs, and the three operating-cost components.
- Added a daily decision rationale and a same-input comparison against the original EWMA policy; checked order timing, minimum quantities, cost totals, and that future sales do not affect earlier decisions.

## Results on the supplied assumptions

| Policy | Fill rate | Lost units | Stockout SKU-days | POs | Holding cost | Stockout cost | Operating cost |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Original EWMA policy | 94.5% | 361 | 10 | 65 | $643.20 | $2,610.00 | $3,253.20 |
| Enhanced agent | 96.6% | 222 | 6 | 63 | $834.13 | $1,678.00 | $2,512.13 |

The enhanced rule holds more stock but lowers the modeled total cost by **$741.07** and improves fill rate by about **2.1 percentage points** on this 90-day sample. These are in-sample scenario results with illustrative costs. Test other time periods and assumptions before treating the policy as a business recommendation. Daily observed sales may itself be censored by real stockouts that the dataset does not identify.
