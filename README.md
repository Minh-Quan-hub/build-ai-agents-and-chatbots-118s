## How to run

1. Open `ad_optimization_agent_colab(2).ipynb` in Google Colab.
2. Add your API key to Colab **Secrets** as `OPENAI_API_KEY` and enable notebook access. If your secret has another name, change it in the setup cell.
3. Select **Runtime > Run all** and upload `ad_performance_mock.csv` when prompted. You can use your own CSV with the columns `date,channel,spend,impressions,clicks,conversions`.
4. Review the daily budget table, budget shifts with short explanations, evaluation table, and chart. The notebook saves `ad_agent_decision_log.csv` and `ad_agent_evaluation.csv` in the Colab runtime.

To run without an API key, set `USE_LLM = False` in the setup cell. The budget decisions remain the same; the notebook uses a brief default explanation.

## Assumptions

- The CSV has one row per channel per day for Search, Social, and Display, with positive spend and `conversions <= clicks <= impressions`.
- The first three days use an equal split. Each later decision uses only the previous three completed days.
- The daily budget is $300. Each channel keeps at least 10% ($30), changes by no more than 20% of its previous budget per day, and transfers no more than 10% of the daily total in one decision.
- The budget tool chooses the allocation. The API provides a short explanation for each shift; it does not change the allocation.
- Evaluation estimates outcomes by assuming conversions, clicks, and impressions scale linearly with channel spend. The comparison is a modeled estimate, not measured lift.

## Results snapshot

On the included 21-day mock dataset, the agent makes 12 budget shifts and keeps the daily total at $300.

| Metric | Agent | Equal split |
| --- | ---: | ---: |
| Total spend | $6,300.00 | $6,300.00 |
| Modeled conversions | 519.78 | 412.00 |
| Modeled CPA | $12.12 | $15.29 |

The modeled conversion difference is **26.2%**. API explanations may vary between runs, but they do not affect the budget or these modeled totals.
