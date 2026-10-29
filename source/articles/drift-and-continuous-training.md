# Concept drift, PSI and continuous training

**Covers:** concept drift, PSI, continuous training

## Two ways that the world moves {#overview}
A model can become wrong for two different reasons. The inputs may change, which is data drift. Or the connection between inputs and the right answer may change, which is concept drift. The remedy is the same in both cases: train again on recent data. Doing that on a regular basis has a name.

## Concept drift {#concept-drift}
Concept drift is a shift in the relationship between inputs and the correct output, even when the inputs themselves look the same. A classic example: buying patterns that meant "likely to purchase" before a holiday season, or before a change in the economy, no longer mean it afterwards. The model's rules are old, and the world's rules are new.

Concept drift is harder to see than data drift, because the inputs look normal. Often only the ground truth, once it arrives, reveals it.

## PSI {#psi}
PSI, the population stability index, is a score of how much a feature's distribution has drifted from a reference. You split the feature into buckets, compare the share of values in each bucket now with the share in the training data, and add up a measure of the differences. Common rules of thumb: below 0.1 means stable, 0.1 to 0.25 is a moderate change worth watching, and above 0.25 is a large change.

## Continuous training {#continuous-training}
Continuous training, CT, is the practice of automatically retraining a model on a regular basis, so that it does not quietly go stale as the world and the data change. It is triggered by a schedule, a drift alarm or a drop in measured quality. It needs the pipeline, tests and release steps to be automatic, or each retraining becomes a project.

## A worked example {#example}
A feature, "customer age", is split into three buckets. Shares of customers:

| Bucket | Training data | This month | Difference | Term |
|---|---|---|---|---|
| Under 30 | 40% | 25% | −15 points | (0.25 − 0.40) × ln(0.25 / 0.40) |
| 30 to 50 | 40% | 40% | 0 | 0 |
| Over 50 | 20% | 35% | +15 points | (0.35 − 0.20) × ln(0.35 / 0.20) |

Calculate each term (natural logarithms):

- Under 30: (−0.15) × ln(0.625) = (−0.15) × (−0.470) = 0.0705.
- 30 to 50: 0.
- Over 50: (0.15) × ln(1.75) = 0.15 × 0.5596 = 0.0839.

PSI = 0.0705 + 0 + 0.0839 = 0.154. That is between 0.1 and 0.25: a moderate change. Worth watching, and perhaps worth retraining soon, but not an emergency. If next month the shares were 15%, 40% and 45%, the terms would be (0.15 − 0.40) × ln(0.375) = (−0.25) × (−0.981) = 0.245, and (0.45 − 0.20) × ln(2.25) = 0.25 × 0.811 = 0.203. PSI = 0.448. That is large, and triggers retraining.

PSI only compares the inputs, so it detects data drift. It says nothing about concept drift. For that, you need the outcomes.

A concept drift case: a model predicts whether a customer will buy, using age, visits and basket size. In the training year, customers with many visits bought 30% of the time. After a price rise, those same customers buy only 15% of the time. The inputs, many visits, look just the same, so PSI stays at 0.03. But the share of correct predictions falls from 85% to 71% once purchase outcomes are collected. Only that comparison reveals the change.

A continuous training setup might read:

1. Every night, compute PSI for the main features.
2. Every week, compute accuracy on the outcomes that have arrived.
3. Retrain when PSI is above 0.25, or accuracy has fallen by more than 3 points, or 30 days have passed since the last training.
4. The new model goes through the same tests and release steps as any other.

| Signal | Detects | Needs ground truth? |
|---|---|---|
| PSI on inputs | Data drift | No |
| Accuracy on outcomes | Concept drift and more | Yes |
| Calendar | Anything slow | No |

Training too often has a cost: computing, and the risk of releasing noise as if it were progress. Training too rarely lets quality slide. The best interval is found by watching how fast the accuracy falls in your own setting.

## A common mistake {#mistake}
Relying on input drift scores alone and missing concept drift. Compare predictions with outcomes whenever they become available. Another mistake is automating retraining but not the tests, so a poor model can go live automatically.

## Key points {#key-points}
- Data drift changes the inputs. Concept drift changes the link between inputs and answers.
- PSI scores input drift: below 0.1 stable, above 0.25 large.
- Concept drift shows up only against real outcomes.
- Continuous training retrains automatically on a schedule or a trigger, with the same tests every time.
