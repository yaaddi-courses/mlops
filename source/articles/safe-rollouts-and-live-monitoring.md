# Safe rollouts and watching a live model

**Covers:** canary deployment, shadow deployment, rollback, output distribution monitoring, confidence score, ground truth

## Release carefully, then keep watching {#overview}
A model can pass every test and still fail on live traffic, because real data is stranger than test data. Two release methods limit the damage of a surprise. Monitoring then tells you whether the model keeps working over time, in ways that ordinary software monitoring cannot see. Let's take the release methods first.

## Canary deployment {#canary-deployment}
In a canary deployment, you send the new model version to only a small slice of live traffic first, say 5%, while the rest keeps using the old, proven version. If the canary looks healthy, the share grows step by step. If it does not, the damage stayed within that small slice. The name comes from the canary in the coal mine.

## Shadow deployment {#shadow-deployment}
A shadow deployment goes one step more careful. The new model receives real, live traffic in parallel with production, but its output is thrown away instead of being shown to anyone. It sees real inputs, and you compare its answers with the live model's, with no risk to users. The cost is that you operate two models at once.

## Rollback {#rollback}
If a release goes wrong, you need a way back. Rollback means returning traffic to the last known-good version of a model, preferably automatically, when live measurements cross a bad threshold, such as an error rate above 2%. A fast, automatic return is far better than waiting for a person to notice.

## Output distribution monitoring {#output-distribution-monitoring}
Once the model is live, what can you watch? Output distribution monitoring follows the overall shape of what a model predicts over time: the split between classes, or the range of numbers it returns. It needs no outside information, so it works immediately. If a fraud model suddenly flags 8% of payments when it used to flag 1%, something has changed, either in the world or in the model's inputs.

## Confidence score {#confidence-score}
A confidence score is the probability the model attaches to its own prediction, which shows how sure it is. A rising share of low-confidence predictions is often an early warning, because the model is meeting inputs unlike those it learned from.

## Ground truth {#ground-truth}
Ground truth is the actual, correct outcome for a prediction once it is known: whether a loan was repaid, or whether a message was really spam. You compare predictions against it to measure real accuracy. The difficulty is delay, since a loan outcome may take a year to appear.

## A worked example {#example}
Let's follow a bank that releases a new fraud model, version 2, to replace version 1.

**Step 1: shadow.** For a week, version 2 scores every payment, and the results are only recorded. You compare it with version 1 on 1 million payments.
- Version 1 flagged 10,000 (1.0%).
- Version 2 flagged 14,000 (1.4%).
- They agree on 9,200 of the flags. Version 2 flags 4,800 extra and misses 800 that version 1 caught.

That is a signal for the analysts. The 4,800 extra flags might be real fraud or false alarms, and a sample review decides.

**Step 2: canary.** After the review, version 2 receives 5% of traffic. The team watches the error rate and the flag rate. After a day all is stable, so the share goes 5%, 25%, 50%, 100% over a week.

**Step 3: rollback rule.** If the share of low-confidence predictions (below 0.6) rises from the usual 4% to above 10%, or the service errors exceed 2%, traffic returns to version 1 automatically.

Now the monitoring. Three months after the release, this is what the team sees:

| Signal | Needs ground truth? | Delay | Reading |
|---|---|---|---|
| Output distribution (flag rate) | No | Minutes | Flag rate rose from 1.4% to 3.0% |
| Confidence scores | No | Minutes | Low-confidence share rose from 4% to 12% |
| Accuracy | Yes | Weeks | Not yet known |

The first two signals arrive at once, and they warn that something is wrong. Fraud may have changed, or an upstream system may have broken. Accuracy can be measured only when investigators confirm the fraud cases, which takes about six weeks. So the early signals stand in until the ground truth arrives.

Let's count the cost of the shadow week. Running two models at once doubles the serving cost for that week. If serving normally costs 3,000 dollars a week, the extra is 3,000 dollars. That is cheap insurance against releasing a model that flags 40% more payments without anyone knowing why.

## A common mistake {#mistake}
A common slip is relying on accuracy alone when the ground truth arrives weeks late. By then, the damage is done, so watch the signals that need no outcome.

Another slip is a 5% canary without a clear stop rule. The team sees a problem and cannot agree whether to act.

## Key points {#key-points}
- A canary sends a small share of traffic to the new version. A shadow runs it silently beside the live one.
- Rollback returns to the last good version, ideally automatically.
- Output distributions and confidence scores warn at once. Ground truth confirms later.
- Define stop rules before the release.
