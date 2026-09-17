# Decision Log

1. I chose @AmericanAir from approximately dozens of brands. Airline support has the broadest spread of issue types in the data set (disruption, baggage, staff, refunds, seating, plus lots of praise) so a 7-intent taxonomy is in fact exercised. Tech-support brands skew heavily towards either one or two intents. Trade-off: airline replies are usually templated unusually, which flatters the reply-generation evaluation.

2. I built the taxonomy from the data, and not reused Banking77 or a generic support ontology. I read a few hundred inbound tweets first, then named the clusters I kept seeing. The assignment allows using Banking77 for intent work, which describes banking transactions, not airline grievances, and the borrowed taxonomy would have hidden the interesting problem of where the boundaries actually fall.

3. Seven intents, not fifteen. More classes would cause each class's support to fall below anything measurable on a golden set this size. I prefer a rough taxonomy I can evaluate over a fine one who I can only describe.

4. I kept general_other as a single bucket, knowing it's a dumping ground. Splitting it mid-project would have invalidated the already-labelled batches and broken the PRE/POST comparison. I documented it as failure mode 5 and as week-two work instead.

5. Single-label, no multi-label. Many messages include two issues (delay and rude agent). Multi-label would be more faithful but makes accuracy, baselines and the confusion matrix harder to read for a short project. The cost is reflected in failure modes 2 and 3.

6. I sampled randomly within the brand rather than balancing classes. A balanced golden set would have made the per-class numbers look better and the headline number meaningless the production distribution is what the agent will face. The price is that refund_compensation has 1 test example.

7. I froze an 86-row test set before any improvement work, and never touched it. No regeneration, no re-labelling, no dropping hard rows. This is the only reason the PRE→POST comparison means anything.

8. I changed exactly one variable between PRE and POST: the system prompt. Same model, intents, rows, labels and scoring. Tempting alternatives (bigger model, retrieval-augmented classification, more few-shot examples) were held back so the delta is attributable.

9. I dropped the auto-selected few-shot examples in the POST prompt and wrote clean synthetic ones instead. The PRE prompt picked its examples with golden[golden["intent"]==intent].head(2), which fed it some of the worst rows in the set, a refund_compensation example that is really a baggage case and another that is really praise. Teaching the model my annotation noise was worse than teaching it nothing.

10. I did not relabel the three bad refund_compensation rows, even though it would have raised accuracy. Fixing labels after seeing model errors is how you launder a bad number into a good one. They are documented as annotation-quality issues instead.

11. I revised 12 human reply ratings after re-reading them and disclosed it. The revision happened because I had been scoring "acknowledges politely" and "actually resolves" the same way, not because I was chasing agreement with the judge. But it does mean the judge-human correlation (r ≈ 0.70, n = 12) is not blind, so I report it as weak evidence rather than validation.

12. Two baselines: majority-class and keyword rules, both deliberately weak and said so. A trivial and a simple baseline satisfy the brief, but beating them is a floor check, not proof of value. Naming a TF-IDF + logistic regression baseline as the missing harder bar was better than quietly implying 25.58% is the real competition.

13. I used AI to pre-suggest labels for the 80 additonal rows, then spot-checked the resulting labels rather than treat them as independent hand labels. The requirement is 150–250 examples hand-labelled by me, so I do not present all 180 as independently hand-labelled. The original 100 rows were labelled from scratch; the additional 80 were accelerated with AI suggestions and reviewed, with ambiguous rows getting extra attention. This is disclosed as a limitation rather than hidden.

14. I made escalation a rule over the predicted intent, not a second LLM call. It is explainable ("escalated because staff_complaint"), free, and importantly it makes escalation errors traceable to classification errors. The honest consequence is that 63.3% escalation accuracy is largely inherited classifier error, which a standalone escalation model would decouple.

15. I grounded reply generation on retrieved historical AA replies rather than letting the model write freely. The model otherwise invents policy (compensation amounts, rebooking promises). Retrieval keeps it inside AA's real behaviour mostly "acknowledge, then ask for a record locator via DM" ,which is less impressive-looking and much safer.

16. I reported the escalation number in the report body even though it is the weakest result. It is the metric closest to real product risk. Leaving it in a notebook cell while headlining classification accuracy would have been the most misleading thing I could have done.
