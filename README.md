## How to run

1. Upload `Ad_Optimization_Agent_Colab.ipynb` to Google Colab.
2. Add your API key to Colab **Secrets** as `OPENAI_API_KEY` and enable notebook access. If your secret has a different name, edit the name in the setup cell.
3. Select **Runtime > Run all** and upload `ad_performance_mock.csv` (or your own CSV) when prompted.
4. Review the allocation table, decision log, baseline comparison, and chart. The notebook saves `ad_agent_decision_log.csv` and `ad_agent_evaluation.csv` in the Colab runtime.

The CSV needs one row per day for each channel (Search, Social, Display) and these columns: `date,channel,spend,impressions,clicks,conversions`. Set `USE_LLM = False` for an offline reference run without an API key.

## Assumptions

- The first three days use equal budgets. Later decisions use only the previous three completed days.
- The daily total defaults to $300. Each channel keeps at least 10% of the total and changes by no more than 20% from its previous budget. A proposed transfer is capped at 10% of the daily total. The API can approve or hold a proposed shift, but cannot set an arbitrary budget.
- Dates are consecutive, spend is positive, and `conversions <= clicks <= impressions`. The assignment dataset has 14–30 days.
- Historical results assume each channel's impressions, clicks, and conversions scale linearly with spend on a given day. They are modeled estimates, not measured causal lift.

## Results snapshot

The reproducible offline reference run (`USE_LLM = False`) on the 21-day mock dataset passed the budget guardrails.

| Metric | Agent | Equal split |
| --- | ---: | ---: |
| Total spend | $6,300.00 | $6,300.00 |
| Estimated conversions | 519.78 | 412.00 |
| Estimated CPA | $12.12 | $15.29 |

The modeled conversion difference was **26.2%**. A live API run may hold proposed shifts and produce different results.
