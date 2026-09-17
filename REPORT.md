# AmericanAir Support Agent — Report

## 1. What is this project?

-I built a small AI agent that reads a customer tweet sent to American Airlines and does three things:

- Figures out what the customer wants (their 'intent') — for example, a delayed flight, a lost bag, or just saying thank you.
- Writes a reply based on how American Airlines has actually replied to similar messages in the past, not a generic made-up answer.
- Decides whether the reply can be sent automatically, or whether a human needs to step in — and says why.

## 2. What counts as 'good' here, and what I chose not to build

-For an airline, the worst mistake isn't getting the category slightly wrong. It's auto-sending a reply to someone who needed a human — for example, someone reporting harassment or an unresolved refund. So the real measure of success is not just accuracy, it's: does the system correctly recognize when a human needs to step in?

Things I deliberately did not build, and why:

- No fine-tuning a custom model — the available labelled set contains 180 examples, including 100 manually labelled examples and an additional 80 examples with AI-suggested labels. This is too small and inconsistently validated to justify fine-tuning a custom model safely.
- No fancy search/embedding system to find similar past messages — I used a simpler method (match by category) that is easier to explain and check by hand.
- No handling of full back-and-forth conversations — I only look at the customer's first message.
- No confidence scores or 'how sure am I' thresholds — with only 86 test examples, any cutoff I picked would just be fitting noise.

## 3. Results

### ---3.1 Intent classification vs. simple baselines

-A 'baseline' is a deliberately dumb comparison point, to prove the AI is actually adding value and not just getting lucky.

| Approach | Accuracy (on 86 test examples) |
|---|---:|
| Majority baseline (always guess the most common category) | 25.58% |
| Keyword baseline (simple hand-written if/else rules) | 30.23% |
| AI classifier, first version (PRE) | 41.86% |
| AI classifier, improved version (POST) | 43.02% |

-Only one thing changed between the first version (PRE) and the improved version (POST): the instructions I gave the AI (the 'prompt'). Same AI model, same 7 categories, same 86 test messages, same scoring method. The POST result increased from 41.86% to 43.02%, a gain of 1.16 percentage points, which is small relative to the uncertainty of such a small test set. That's on purpose — if I'd changed several things at once, I wouldn't know which change actually helped.

### ---3.2 How good are the categories individually?

**PRE version**

| Category | Precision | Recall | How many test examples |
|---|---:|---:|---:|
| Baggage issue | 0.25 | 0.14 | 7 |
| Flight disruption | 0.39 | 0.47 | 19 |
| General / other | 0.50 | 0.50 | 22 |
| Praise / thank-you | 0.57 | 0.63 | 19 |
| Refund / compensation | 0.00 | 0.00 | 1 ← too few to judge |
| Seating / booking | 0.00 | 0.00 | 5 ← too few to judge |
| Staff complaint | 0.30 | 0.23 | 13 |

**POST version**

| Category | Precision | Recall | How many test examples |
|---|---:|---:|---:|
| Baggage issue | 0.40 | 0.29 | 7 |
| Flight disruption | 0.45 | 0.53 | 19 |
| General / other | 0.55 | 0.50 | 22 |
| Praise / thank-you | 0.55 | 0.63 | 19 |
| Refund / compensation | 0.00 | 0.00 | 1 ← too few to judge |
| Seating / booking | 0.17 | 0.20 | 5 ← too few to judge |
| Staff complaint | 0.22 | 0.15 | 13 |

### ---3.3 How good are the generated replies?

-I had the AI grade 30 of its own generated replies on a 1–5 scale (1 = doesn't help at all, 5 = solves the problem). Average score: 4.07 out of 5.

To check whether the AI's grading can be trusted, I also graded 12 of those replies myself by hand and compared. The two sets of scores agreed reasonably well (correlation ≈ 0.70, and they were within 1 point of each other 83% of the time). That's a decent sign, but 12 examples is a small sample — I'm treating this as 'some evidence', not proof.

### ---3.4 How good is the auto-vs-escalate decision?

-Out of 30 messages checked, the system made the right auto/escalate call 63.3% of the time (19 out of 30). This is the weakest number in the whole project, and arguably the most important one — a wrong escalation decision is the costliest kind of mistake this system can make.

### ---3.5 Supplementary check across the 181-example labelled set

-Separately from the frozen 86-example test, I also ran the classifier across the full 180-example labelled set. The additional 80 examples were initially labelled using AI-suggested labels rather than being independently hand-labelled from scratch, so this 180-example result should not be treated as a fully human-validated accuracy estimate. It is included only as a supplementary sanity check, not as evidence comparable to the frozen test-set result.

## 4. Top 5 things that go wrong, with real examples

-Messages that only make sense with earlier context. Example: a customer just writes 'HANSVM' (a booking code) or a two-word reply like 'Well played.' Neither the AI nor I can classify these correctly without seeing the message they're replying to. This is a data-collection problem, not an AI problem(simply – Lack of data provided).

-Complaining about staff during a flight delay gets misclassified as just 'Flight disruption'. Example: "Get your act together @AmericanAir. Been sitting on the ground for 2 hrs." The delay is the situation, but the real complaint is about the staff's response to it , “the AI (and sometimes I) can't always tell which one is the main issue”.

-A compliment buried inside a complaint. Example: someone thanks a specific gate agent by name inside a message I labelled 'general/other' — arguably the AI's 'praise' guess is more right than my own label. Some of this is genuinely ambiguous language, not a model mistake.

-Two categories (refund requests, and seating/booking) barely appear in the test data. Only 1 and 5 test examples respectively. I can't honestly say how well the AI handles these — there just isn't enough data yet to know.

-The catch-all 'general/other' category is too broad. It mixes together three very different things — vague venting, policy questions, and messages that don't fit anywhere else — which makes it both the biggest category and the one most often confused with others.

## 5. What is misleading about my headline number

This section is something the assignment specifically asks for, so I'm being direct about it:

- The 43.02% accuracy number measures agreement with my manually assigned labels on the frozen 86-example test set, not objective truth. When I re-read the mistakes, some of the AI's 'wrong' answers looked more reasonable than my own label — see failure mode 3 above.
- 86 test examples is a small sample. The real accuracy could plausibly be up to about 10 percentage points higher or lower than what's reported, just from sampling luck.
- Two categories (refund, seating/booking) have almost no test data, so their scores are close to meaningless on their own.
- The two 'dumb' baselines are intentionally weak. Beating them is a minimum bar, not proof the system is genuinely useful. A stronger baseline (a simple trained model, not just hand-written rules) would be a fairer comparison, and I didn't build one.
- I wrote the improved (POST) prompt after looking at where the first version made mistakes. That means the test set isn't perfectly 'clean' — there's some indirect influence from having seen the errors first.
- The improvement from PRE to POST is only 1.16 percentage points (41.86% → 43.02%). With only 86 frozen test examples, this should be interpreted as a small directional change rather than strong evidence that the revised prompt substantially improved classification.
- The reply-quality score (4.07/5) is generated by another AI grading this AI, checked against only 12 human ratings by me. It's supportive evidence, not proof of quality.
- The auto/escalate accuracy (63.3%) is the number most likely to get skipped over because it isn't the 'headline' — but for a real support tool, it's arguably the most important number in this whole report.
- The additional 81 rows used for the 180-example labelled set were AI-assisted and only spot-checked, so the full 181 should not be described as independently hand-labelled.

## 6. What I'd do with one more week

- Include the previous message in the conversation, not just the customer's latest tweet, to fix failure mode 1.
- Re-label a batch of examples a second time, a week later, and measure how often I agree with my own past self. That gives an honest ceiling for the accuracy number.
- Split the overloaded 'general/other' category into more specific sub-categories, then re-label with the new categories.
- Deliberately find and label more refund and seating/booking examples, since right now there are too few to say anything meaningful about them.
- Build a proper trained baseline (not just hand-written keyword rules), so the AI has to beat something harder than a dummy guess.
- Treat the escalate/auto decision as its own problem to evaluate carefully, since getting it wrong is the costliest kind of mistake this system can make.
