# Safe rollouts and watching a live model

**Covers:** canary deployment, shadow deployment, rollback, output distribution monitoring, confidence score, ground truth

## Release carefully, then keep watching {#overview}
A model can pass every test and still fail on live traffic, because real data is stranger than test data. Two release methods reduce the damage of a surprise. Monitoring then tells you whether the model keeps working over time, in ways that ordinary software monitoring cannot see.

## Canary deployment {#canary-deployment}
A canary deployment sends a new model version to only a small slice of live traffic first, say 5%, while the rest keeps using the old, proven version. If the canary looks healthy, the share grows step by step. If not, the damage was limited to that small slice. The name comes from the canary in the coal mine.

## Shadow deployment {#shadow-deployment}
A shadow deployment runs a new model on real, live traffic in parallel with production, but throws its output away instead of showing it to anyone. The new model sees real inputs, and you compare its answers with the live model's, with zero risk to users. The cost is that you run two models at once.

## Rollback {#rollback}
Rollback is reverting traffic to the last known-good version of a model, preferably automatically, when live measurements cross a bad threshold, such as an error rate above 2%. A fast, automatic return is far better than waiting for a person to notice.

## Output distribution monitoring {#output-distribution-monitoring}
Output distribution monitoring means watching the overall shape of what a model predicts over time: the split between classes, or the range of numbers it returns. It needs no outside information, so it works immediately. If a fraud model suddenly flags 8% of payments when it used to flag 1%, something has changed, either in the world or in the model's inputs.

## Confidence score {#confidence-score}
A confidence score is the probability that a model attaches to its own prediction, showing how sure it is. A rising share of low-confidence predictions is often an early warning, because the model is meeting inputs unlike those it learned from.

## Ground truth {#ground-truth}
Ground truth is the actual, correct outcome for a prediction once it is known: whether a loan was repaid, or whether a message was really spam. It is what you compare predictions against to measure real accuracy. The difficulty is delay: a loan outcome may take a year to appear.

## A worked example {#example}
A bank releases a new fraud model, version 2, to replace version 1.

**Step 1: shadow.** For a week, version 2 scores every payment, and the results are only recorded. Compare with version 1 on 1 million payments.
- Version 1 flagged 10,000 (1.0%).
- Version 2 flagged 14,000 (1.4%).
- They agree on 9,200 of the flags. Version 2 flags 4,800 extra and misses 800 that version 1 caught.

This is a signal for the analysts: 4,800 extra flags might be real fraud or false alarms. A sample review decides.

**Step 2: canary.** After the review, version 2 receives 5% of traffic. The team watches an error rate and the flag rate. After a day, all is stable, so the share goes 5%, 25%, 50%, 100% over a week.

**Step 3: rollback rule.** If the share of low-confidence predictions (below 0.6) rises from the usual 4% above 10%, or the service errors exceed 2%, traffic returns to version 1 automatically.

Now the monitoring. Three months after the release:

| Signal | Needs ground truth? | Delay | Reading |
|---|---|---|---|
| Output distribution (flag rate) | No | Minutes | Flag rate rose from 1.4% to 3.0% |
| Confidence scores | No | Minutes | Low-confidence share rose from 4% to 12% |
| Accuracy | Yes | Weeks | Not yet known |

The first two signals, available at once, warn that something is wrong. Fraud may have changed, or an upstream system may have broken. Accuracy can be measured only when investigators confirm the fraud cases, which takes about six weeks. So the early signals act as a stand-in until the ground truth arrives.

Cost of the shadow week: running two models at once doubles the serving cost for that week. If serving normally costs 3,000 dollars a week, the extra is 3,000 dollars. It is cheap insurance against releasing a model that flags 40% more payments without knowing why.

## A common mistake {#mistake}
Relying on accuracy alone, when the ground truth arrives weeks late. By then, the damage is done. Watch the signals that need no outcome. Another mistake is a canary of 5% without a clear stop rule, so a problem is seen but nobody is sure whether to act.

## Key points {#key-points}
- A canary sends a small share of traffic to the new version. A shadow runs it silently beside the live one.
- Rollback returns to the last good version, ideally automatically.
- Output distributions and confidence scores warn at once. Ground truth confirms later.
- Define stop rules before the release.
