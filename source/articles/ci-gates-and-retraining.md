# Quality gates, drift and retraining triggers

**Covers:** quality gate, data validation, CD, data drift, cron, champion/challenger

## Automatic checks before a change goes further {#overview}
Software teams run tests on every change. Machine learning needs the same, plus checks on the data and on the model's quality. And since the world changes, there must be a rule for when to train again.

## CD {#cd}
CD, continuous delivery, is the practice of automatically preparing every change so that it can be released at any time. After the checks pass, the package is built and ready, and the release is a decision, not a project. A further step, continuous deployment, releases without a human decision.

## Quality gate {#quality-gate}
A quality gate is a check built into the automatic pipeline that can block a merge or a promotion when something does not meet a set bar. In machine learning it includes checks on data, such as "no column is more than 5% empty", and on the model, such as "the score on the test set is at least 0.85 and not more than two points worse than the model in use".

## Data validation {#data-validation}
Data validation is the practice of automatically checking incoming or training data against expected rules: its structure, value ranges and share of missing values. It stops bad data before it reaches training, where it would silently produce a worse model.

## Data drift {#data-drift}
Data drift is a shift in the statistical shape of a model's input data over time. Imagine a loan model trained on applicants whose average age was 35. If it starts to see applicants averaging 52, it works on inputs it barely knew. The model code did not change, but its quality may fall.

## Cron {#cron}
Cron is the classic scheduling syntax, with five fields for minute, hour, day of month, month and day of week, used to start a job at fixed times. `0 3 * * 1` means 03:00 every Monday. Many pipeline tools reuse the syntax for their schedules.

## Champion/challenger {#champion-challenger}
Champion/challenger is a pattern in which a newly trained model, the challenger, is compared with the model now in use, the champion, on the same data. Only if the challenger is better by a set margin does it replace the champion. It prevents the common accident of replacing a good model with a worse one just because it is newer.

## A worked example {#example}
A churn model is retrained every Monday at 03:00, by the schedule `0 3 * * 1`. The new model is a challenger. The steps:

1. **Data validation.** The new data has 120,000 rows. The gate checks: all expected columns exist, `age` is between 18 and 100, and no column is more than 5% missing. One column has 9% missing. The gate fails, the run stops, and an alert is sent. The bad model is never trained.

2. After the data is fixed, the run goes on. The challenger scores 0.88 on the fixed test set. The champion scores 0.87 on the same set.

3. **Quality gate.** The rule: the challenger must beat the champion by at least 0.005 and score at least 0.85. 0.88 − 0.87 = 0.01, which is at least 0.005, and 0.88 is at least 0.85. It passes.

4. **CD.** The package is built and placed where it can be released. A person approves, and the release happens.

If the challenger had scored 0.872, a gain of 0.002, the gate would stop it. The gain is within the noise of the test, and a new model brings its own risk.

Drift can also start a retraining outside the schedule. A weekly check compares the age of this week's applicants with the training set. Training average: 35. This week: 36. A small change, no action. Three months later: 49. The check raises an alarm, and a retraining is triggered at once, instead of waiting for the next Monday.

| Trigger | Starts retraining when |
|---|---|
| Schedule (cron) | The clock says so, e.g. every Monday at 03:00 |
| Drift alarm | Inputs moved far from the training data |
| Quality drop | Live results fall under a set bar |

Compare the cost of a fixed schedule: weekly training of a 2-hour job is 104 hours of computing a year. If the data rarely changes, most of those runs reproduce the same model. A trigger based on drift trains only when it matters. If the data changes fast, though, a schedule gives a predictable minimum.

## A common mistake {#mistake}
Gates with thresholds that nobody revisits. A gate set at 0.85 two years ago may be too low or too high today. Review them. Another mistake is to retrain automatically and release automatically with no champion comparison, so a bad week of data can replace a good model.

## Key points {#key-points}
- A quality gate blocks a change that fails a set bar. Data validation is a gate on the data.
- Data drift means the inputs have shifted from the training data.
- Cron schedules retraining. Drift alarms can trigger it earlier.
- A challenger replaces the champion only if it is better by a set margin.
