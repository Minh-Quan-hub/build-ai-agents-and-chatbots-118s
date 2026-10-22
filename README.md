# Ad Optimization Agent - README

# Run in Google Colab
1. Upload Ad_Optimization_Agent_Colab.ipynb and add an API key in Secrets as OPENAI_API_KEY (or OPENAI_EDU_API_KEY). Enable notebook access to that secret. The key is read at runtime and is never written into the notebook.
2. For the included 21-day example, leave DATA_SOURCE = "mock". To provide your own file, set DATA_SOURCE = "upload" and choose one CSV when prompted, or set DATA_SOURCE = "path" and CSV_PATH to an uploaded file.
3. Choose Runtime > Run all. The setup installs LangGraph and the OpenAI SDK if needed. The default USE_LLM = True sends aggregate performance summaries to gpt-6-luna for approve/hold reviews. Set it to False for a reproducible offline reference run.
4. Inspect the allocation table, per-channel reasons, comparison, and chart. The notebook writes ad_agent_decision_log.csv and ad_agent_evaluation.csv in the Colab runtime.
Your CSV needs date,channel,spend,impressions,clicks,conversions, with one row per day for each of Search, Social, and Display. Dates must be consecutive; spend positive; counts whole and nonnegative; conversions <= clicks <= impressions. Use 14-30 days for the assignment.

# Assumptions
- The first three days use equal budget. Later decisions use only the previous three completed days.
- The fixed daily budget defaults to $300. The candidate transfer is capped at 10% of total spend, each channel changes by at most 20% from yesterday, and each keeps at least 10% of total spend. The API can approve the candidate or hold; it cannot set arbitrary budgets.
- Historical counterfactual results assume within-day impressions, clicks, and conversions scale linearly with spend. Auction effects and diminishing returns are omitted, so the comparison is illustrative rather than causal.
- API calls need an active OpenAI project with access to the configured model. The notebook caps calls at 30, outputs at 250 tokens per review, and uses short timeouts and limited retries. A real account-level spend limit is configured outside the notebook.

# Results snapshot
The reproducible offline reference run (USE_LLM = False) on the mock CSV passed its budget and zero-click checks. At equal total spend of $6,300, the modeled agent allocation yielded 519.78 conversions and $12.12 CPA, versus 412.00 conversions and $15.29 CPA for equal split (about 26.2% modeled conversion difference). A live API run may hold some candidates and produce different results; it has not been claimed here as tested with a real key.
