# Quality gates, drift and retraining triggers

**Covers:** quality gate, data validation, CD, data drift, cron, champion/challenger

## Automatic checks before a change goes further {#overview}
If you have written software in a team, you know the habit: every change is tested before it goes anywhere. Machine learning needs the same habit, with two additions. You also check the data, and you check the quality of the model. And because the world keeps changing, you need a rule that says when to train again. Let's go through the pieces in the order a change meets them.

## CD {#cd}
Start with the end of the line. CD, continuous delivery, means that every change is prepared automatically so that it could be released at any time. Once the checks pass, the package is built and waiting, and the release becomes a decision instead of a project. There is a further step, continuous deployment, which releases without anyone deciding.

## Quality gate {#quality-gate}
Before a change reaches that point, it passes a quality gate. A gate is an automatic check in the pipeline that can block a merge or a promotion when something misses a required bar. In machine learning, gates cover data, such as "no column is more than 5% empty", and the model, such as "the score on the test set is at least 0.85 and not more than two points worse than the model in use".

## Data validation {#data-validation}
The first gate usually looks at the data. Data validation checks incoming or training data against expected rules automatically: its structure, its value ranges and its share of missing values. Why bother? Because bad data that reaches training quietly produces a worse model, and nobody gets an error message.

## Data drift {#data-drift}
Even good data ages. Data drift is a shift in the statistical shape of a model's input data over time. Imagine a loan model trained on applicants whose average age was 35. If it begins to see applicants who average 52, it works on inputs it barely knew. The model code did not change, but its quality may fall.

## Cron {#cron}
How do you start retraining on a timetable? Most tools borrow the syntax of cron, the classic scheduler. It has five fields, for minute, hour, day of month, month and day of week, and it starts a job at fixed times. For example, `0 3 * * 1` means 03:00 every Monday.

## Champion/challenger {#champion-challenger}
When a new model has been trained, how do you decide whether to use it? Compare it with the model now in use. In the champion/challenger pattern, the new model, the challenger, is tested against the current one, the champion, on the same data. Only if the challenger is better by a chosen margin does it take over. This prevents a common accident: replacing a good model with a worse one only because it is newer.

## A worked example {#example}
Let's follow a churn model that is retrained every Monday at 03:00, by the schedule `0 3 * * 1`. The new model is a challenger.

1. **Data validation.** The new data has 120,000 rows. The gate checks that all expected columns exist, that `age` is between 18 and 100, and that no column is more than 5% missing. One column has 9% missing, so the gate fails, the pipeline stops and an alert goes out. The bad model is never trained.

2. After the data is fixed, the pipeline continues. The challenger scores 0.88 on the fixed test set, and the champion scores 0.87 on the same set.

3. **Quality gate.** The rule says the challenger must beat the champion by at least 0.005 and score at least 0.85. We check: 0.88 − 0.87 = 0.01, which is at least 0.005, and 0.88 is at least 0.85. It passes.

4. **CD.** The package is built and put where it can be released. A person approves, and the release happens.

Now suppose the challenger had scored 0.872. The gain of 0.002 would fail the gate. It is within the noise of the test, and a new model always brings its own risk.

Drift can start retraining outside the schedule. A weekly check compares the age of this week's applicants with the training set. The training average is 35 and this week's is 36. That is a small change, so nothing happens. Three months later it is 49, the check raises an alarm, and retraining starts at once instead of waiting for Monday.

| Trigger | Starts retraining when |
|---|---|
| Schedule (cron) | The clock says so, e.g. every Monday at 03:00 |
| Drift alarm | Inputs moved far from the training data |
| Quality drop | Live results fall under a chosen bar |

Let's compare costs. Weekly training of a 2-hour job is 52 × 2 = 104 hours of computing a year. If the data rarely changes, most of those runs reproduce the same model, and a drift trigger trains only when it matters. If the data changes fast, though, a schedule gives a predictable minimum.

## A common mistake {#mistake}
A common slip is leaving gate thresholds untouched for years. A gate fixed at 0.85 long ago may be too low or too high today, so review them from time to time.

Another slip is retraining and releasing automatically with no champion comparison. A single bad week of data can then replace a good model.

## Key points {#key-points}
- A quality gate blocks a change that fails a chosen bar. Data validation is a gate on the data.
- Data drift means the inputs have shifted away from the training data.
- Cron schedules retraining, and drift alarms can trigger it earlier.
- A challenger replaces the champion only if it is better by a chosen margin.
