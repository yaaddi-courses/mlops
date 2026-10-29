# Concept drift, PSI and continuous training

**Covers:** concept drift, PSI, continuous training

## Two ways that the world moves {#overview}
A model can go wrong for two different reasons. The inputs may change, which is data drift. Or the link between the inputs and the right answer may change, which is concept drift. The remedy is the same either way: train again on recent data. In this article you will learn how to measure the first kind, why the second kind is harder to catch, and how to automate the retraining.

## Concept drift {#concept-drift}
Let's start with the harder one. Concept drift means the relationship between the inputs and the correct output has shifted, even though the inputs themselves look the same. Here is a classic case. Buying patterns that meant "likely to purchase" before a holiday season, or before a change in the economy, may mean something else afterwards. The model's rules are old, and the world's rules are new.

This one is hard to see, because the inputs look normal. Often only the real outcomes, once they arrive, reveal it.

## PSI {#psi}
For the easier kind, the changing inputs, you can use a score. PSI, the population stability index, tells you how far a feature's distribution has moved from a reference. You split the feature into buckets, compare the share of values in each bucket now with the share in the training data and add up a measure of the differences. The usual rules of thumb: below 0.1 is stable, 0.1 to 0.25 is a moderate change worth watching, and above 0.25 is a large change.

## Continuous training {#continuous-training}
Once you can detect change, you can respond to it automatically. Continuous training, or CT, means retraining a model on a regular basis, so it does not quietly go stale as the world changes. The trigger can be a schedule, a drift alarm or a drop in measured quality. It only works if the pipeline, the tests and the release steps are automatic, because otherwise each retraining becomes a project.

## A worked example {#example}
Let's compute PSI by hand. A feature, "customer age", is split into three buckets. The shares of customers are:

| Bucket | Training data | This month | Difference | Term |
|---|---|---|---|---|
| Under 30 | 40% | 25% | −15 points | (0.25 − 0.40) × ln(0.25 / 0.40) |
| 30 to 50 | 40% | 40% | 0 | 0 |
| Over 50 | 20% | 35% | +15 points | (0.35 − 0.20) × ln(0.35 / 0.20) |

We calculate each term with natural logarithms:

- Under 30: (−0.15) × ln(0.625) = (−0.15) × (−0.470) = 0.0705.
- 30 to 50: 0.
- Over 50: 0.15 × ln(1.75) = 0.15 × 0.5596 = 0.0839.

PSI = 0.0705 + 0 + 0.0839 = 0.154. That sits between 0.1 and 0.25, a moderate change. It is worth watching, and perhaps worth retraining soon, but it is not an emergency.

Suppose next month the shares are 15%, 40% and 45%. The terms are (0.15 − 0.40) × ln(0.375) = (−0.25) × (−0.981) = 0.245, and (0.45 − 0.20) × ln(2.25) = 0.25 × 0.811 = 0.203. PSI = 0.448. That is large, and it triggers retraining.

Notice what PSI looks at: only the inputs. It detects data drift and says nothing about concept drift. For that, you need the outcomes.

Here is a concept drift case. A model predicts whether a customer will buy, using age, visits and basket size. In the training year, customers with many visits bought 30% of the time. After a price rise, those same customers buy only 15% of the time. The inputs, many visits, look the same, so PSI stays at 0.03. But once purchase outcomes are collected, the share of correct predictions falls from 85% to 71%. Only that comparison reveals the change.

A continuous training setup might read like this:

1. Every night, compute PSI for the main features.
2. Every week, compute accuracy on the outcomes that have arrived.
3. Retrain when PSI is above 0.25, or accuracy has fallen by more than 3 points, or 30 days have passed since the last training.
4. The new model goes through the same tests and release steps as any other.

| Signal | Detects | Needs real outcomes? |
|---|---|---|
| PSI on inputs | Data drift | No |
| Accuracy on outcomes | Concept drift and more | Yes |
| Calendar | Anything slow | No |

Training has a cost either way. Too often, and you spend computing and risk releasing noise as if it were progress. Too rarely, and quality slides. You find the right interval by watching how fast accuracy falls in your own setting.

## A common mistake {#mistake}
A common slip is relying on input drift scores alone and missing concept drift. Compare predictions with outcomes whenever they become available.

Another slip is automating the retraining but not the tests, so a poor model can go live without anyone seeing it.

## Key points {#key-points}
- Data drift changes the inputs. Concept drift changes the link between inputs and answers.
- PSI scores input drift: below 0.1 is stable, above 0.25 is large.
- Concept drift shows up only against real outcomes.
- Continuous training retrains automatically on a schedule or a trigger, with the same tests every time.
