# Testing ML systems, model cards and approvals

**Covers:** data validation test, invariance test, directional test, model card, approval workflow, fairness check

## Beyond unit tests {#overview}
Ordinary tests check that code does what it should. A model is not just code, and its behavior cannot be written out in advance. So testing a model means checking the data, checking how the model behaves on chosen examples, and checking that people reviewed it before it reached real users.

## Data validation test {#data-validation-test}
A data validation test is a check on the data itself, and not on your code's logic. For example: the `age` column never holds a negative number, a required column is present, and no more than 5% of a column is empty. It runs whenever new data arrives, and it is the cheapest test to write and often the most valuable.

## Invariance test {#invariance-test}
An invariance test changes something the model should not care about and expects the output to stay the same. Swap an applicant's name, or change the order of two harmless words in a sentence, and the prediction should not move. A change shows the model uses something it should not.

## Directional test {#directional-test}
A directional test checks that a prediction moves in the expected direction when an input changes in a meaningful way. For example, a bigger floor area should not lower a house's predicted price, other things equal, and a higher income should not lower the chance of loan approval.

## Model card {#model-card}
A model card is a short, standard document that goes with a model. It covers the intended use, the training data, the known limitations and the measured performance, often for different groups. A reader should learn within minutes what the model is for, and what it must not be used for.

## Approval workflow {#approval-workflow}
An approval workflow is a required human sign-off before a model is promoted to production, usually kept for higher-stakes models, such as one that decides loans or medical priority. It names who must approve, what evidence they see, and records the decision.

## Fairness check {#fairness-check}
A fairness check compares a model's performance across different groups of people, for example whether a hiring model approves résumés at similar rates across genders or age ranges. It shows differences that an overall average hides.

## A worked example {#example}
A bank has a loan model. Its checks:

**Data validation.** A new data file arrives. Rules: `age` between 18 and 100; `income` not negative; `loan_amount` present. Test result: 3 rows have `age` of −1 or 250. The test fails, and the file is rejected before training.

**Invariance.** Take 1,000 applications and change only the applicant's first name. Count how many predictions change. Expected: 0. Result: 14 of 1,000 change (1.4%). The model reacts to names, possibly through a proxy for gender or origin. This is a problem to fix, even at 1.4%.

**Directional.** Raise each applicant's income by 10% and keep everything else. The approval score should not fall. Result: it rose for 972 of 1,000 and fell for 28. Investigate the 28.

**Fairness.** Approval rates for two age groups with the same repayment record: 62% for the group aged 25 to 40, and 51% for those over 60. The gap is 11 points. A gap needs an explanation, or a fix.

**Model card.** The card states: purpose (pre-screening of personal loans), training data (applications 2022 to 2025), limits (not tested for business loans), performance (accuracy 87% overall; the group results above) and an owner to contact.

**Approval.** Because the model affects people's access to credit, the workflow needs two sign-offs: a data scientist and a risk officer. Each reviews the tests and the card. The decision, the names and the date are recorded in the registry. With the 1.4% name effect and the 11-point gap open, the officers refuse to approve. The model returns for work.

Count the cost of skipping. A failing fairness check found before release costs a few days of work. Found after release by a regulator, it can mean fines, a forced withdrawal of the model and harm to the people affected.

| Check | Catches | Needs the real answers? |
|---|---|---|
| Data validation test | Bad inputs | No |
| Invariance test | Use of irrelevant details | No |
| Directional test | Behavior that makes no sense | No |
| Fairness check | Unequal results by group | Often yes |

## A common mistake {#mistake}
Testing only the average score. A model can reach 90% overall and fail badly on a small group. Another mistake is writing the model card once and never updating it when the model is retrained.

## Key points {#key-points}
- Test the data, the behavior and the process, not only the code.
- Invariance tests expect no change, and directional tests expect a known direction.
- A model card says what the model is for and where it fails.
- Higher-stakes models need a recorded human approval.
