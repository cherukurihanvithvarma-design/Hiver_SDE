# AmericanAir Support Agent — Hiver SDE Intern Take-Home

A small AI customer-support agent for @AmericanAir built on the Customer Support on Twitter dataset. It classifies customer intent, drafts a reply grounded in how AA historically resolved similar issues, and decides auto-handle vs escalate with a stated reason.

Headline result: <POST_ACC>% intent accuracy on a frozen 86-row test set, against a 25.58% majority baseline and a 30.23% keyword baseline.

Read this next: REPORT.md — including the mandatory "what is misleading about my headline number" section — and DECISION_LOG.md.

## Process Sequence

1. Open `notebooks/agent.ipynb` in Google Colab (File → Upload notebook).
2. Runtime → Run all. Paste your API key when the first cell prompts for it.
3. Total runtime ~8 min, ~200 LLM calls against google/gemini-2.5-flash.

### Expected output from the evaluation cells:

- PRE LLM accuracy: 41.86%
- POST LLM accuracy: 43.02%
- Majority baseline: 25.58%
- Keyword baseline: 30.23%

The golden set and the frozen test split ship with the repo (data/golden_set.csv, data/test_set.csv), so you do not need to download twcs.csv to reproduce the headline number — the notebook loads the frozen split directly. twcs.csv (Kaggle, ~500MB) is only needed to rebuild the golden set from scratch or to regenerate the retrieval corpus for reply generation; those cells are marked optional and skipped by default.

If you would rather not run anything, every number in this README is reproduced row-by-row in data/classifier_results.csv and data/judged_replies.csv.

## Repo layout

```text
Hiver_SDE/
├── README.md
├── REPORT.md
├── DECISION_LOG.md
├── notebooks/
│   └── agent.ipynb
└── data/
    ├── golden_set_180_full.csv
    ├── test_set.csv
    ├── classifier_results.csv
    ├── post_classifier_results.csv
    ├── human_agreement_check.csv
    └── judged_replies.csv
```

## Data

- Source: twcs.csv (~2.8M tweets, dozens of brands). Filtered to inbound tweets mentioning @AmericanAir that have a linked brand reply → ~1,737 customer/reply pairs. Malformed rows skipped on load.
- Golden set: 181 pairs, labelled by me with intent (7 classes) and escalation (auto/escalate). Sampled randomly within the brand, so it preserves the real class imbalance. 100 rows were labelled from scratch in two batches of 50; the remaining 81 were labelled by reviewing and correcting AI-suggested labels row by row (see DECISION_LOG.md #13). Ambiguous rows carry a note explaining the call.
- Test split: 86 rows, frozen before any prompt improvement and never modified.

## Intents

| intent | definition |
|---|---|
| praise | Compliments, thanks, positive feedback — no action needed |
| flight_disruption | Delays, cancellations, diversions, mechanical issues, tarmac/deplaning |
| baggage_issue | Lost/damaged/delayed bags, baggage fees, carry-on disputes |
| staff_complaint | Rude, unhelpful or poor treatment by employees |
| refund_compensation | Explicit requests for refunds, vouchers, miles, reimbursement |
| seating_booking | Seat assignment, upgrades, standby, booking/reservation changes |
| general_other | Vague venting, app/Wi-Fi/policy questions, loyalty questions, unclear |

## Evaluation

- Intent: accuracy + per-class precision/recall/F1 on the frozen 86 rows, against two baselines.
- Replies: LLM-as-judge, 1–5 rubric (1 = doesn't address the issue, 5 = specific actionable resolution), mean 4.07/5 over 30 replies.
- Judge validity: judge vs my own ratings on 12 replies — Pearson r ≈ 0.70, p = 0.011, 83% within 1 point. Small and non-blind; treated as weak evidence, see REPORT.md §5.
- Escalation: 63.3% accuracy (19/30) against hand-labelled auto/escalate.

## Known limitations

Stated plainly rather than buried: the test set is 86 rows (±~10pp confidence interval); refund_compensation and seating_booking have 1 and 5 test examples; accuracy measures agreement with my labels, and some model "errors" look more correct than my gold label; the PRE→POST prompt change was written after reading PRE errors, so the split is not fully clean; escalation is derived from predicted intent, so it inherits classification error. Full discussion in REPORT.md.

## Credits

- Dataset: Customer Support on Twitter (Kaggle, thoughtvector/customer-support-on-twitter).
- Model: google/gemini-2.5-flash via OpenRouter, using the openai Python client.
- AI assistance: Claude & Chatgpt was used for code scaffolding, prompt drafting, label pre-suggestion on the 80-row add-on batch, and drafting the report and decision log. All labels were reviewed by me; all design decisions and the analysis in REPORT.md are mine.
