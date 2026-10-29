# The ML lifecycle: a loop, not a line

**Covers:** ML lifecycle, ML

## Models are never finished {#overview}
Think about ordinary software. A program can stay the same for years and keep working. A model cannot, because it is a picture of the data it learned from, and the world moves on. So the work does not end when the model is released. It goes round a loop for as long as the model is in use, and this article draws that loop.

## ML {#ml}
Let's start with the term itself. ML, machine learning, is a family of methods in which a computer learns patterns from examples instead of following rules written by hand. A spam filter, for instance, is shown thousands of messages marked "spam" or "not spam" and finds out for itself which features matter. What comes out of this learning is the model: a file of numbers and a procedure that turns new input into a prediction.

## ML lifecycle {#ml-lifecycle}
The ML lifecycle is the loop a model goes through for as long as it is in use. It has five stages.

1. **Train.** Fit the model to examples.
2. **Validate.** Test it on data it has not seen, to find out whether it really learned or only memorized.
3. **Deploy.** Make it available to users or other programs.
4. **Monitor.** Watch how it behaves on live data and compare it with real outcomes.
5. **Retrain.** When monitoring shows it has become stale, return to step 1 with fresh data.

This loop is the central picture of the field. Every later topic is a way to make one of these steps faster, safer or easier to repeat.

## A worked example {#example}
Let's say a shop uses a model to predict next week's demand for each product, so it orders the right amount.

**Round 1.**
- Train on two years of sales data.
- Validate: on the last three months, held back from training, the average error is 8%.
- Deploy: the ordering system calls the model every Monday.
- Monitor: each week, compare the prediction with the actual sales. Weeks 1 to 12 show an error of 8% to 9%.

**A change in the world.** In week 20, a competitor opens next door and a product's sales fall by a third. The model has never seen this, so it keeps predicting the old level. By week 24 the error is 19%.

**Round 2.** The monitor's alert at 15% error, in week 22, sends the team back to the start.
- Retrain on the data including the last 22 weeks.
- Validate: the error on a fresh hold-out period is 9%.
- Deploy the new version.
- Keep monitoring.

Let's count what the loop gave. Overstocking at 19% error against 9% is about ten points of unnecessary stock. If the shop holds 500,000 dollars of goods and a tenth of that is excess, the surplus is 50,000 dollars. Without the loop, nobody would have found the problem until the shelves overflowed.

How long does each stage take? In this example, training takes 2 hours, validation 30 minutes, and deployment 10 minutes with automation but 3 days by hand. Monitoring never stops. The slow steps, usually the manual ones, decide how fast the loop turns. A loop that takes a month to turn means a month of poor predictions each time the world changes.

| Stage | Question it answers |
|---|---|
| Train | Can the model learn from this data? |
| Validate | Does it work on new data? |
| Deploy | Can users reach it? |
| Monitor | Is it still working? |
| Retrain | Is it time to start again? |

Notice that the loop feeds on its own output. Monitoring produces the evidence that triggers the next round of training, and the new model is monitored in the same way.

## A common mistake {#mistake}
A common slip is treating deployment as the last step and sending the team to other work. Without monitoring and retraining, the model decays.

Another slip is validating on the same data used for training. That only measures memory, not learning.

## Key points {#key-points}
- Machine learning learns patterns from examples, and the result is a model.
- The lifecycle is a loop: train, validate, deploy, monitor, retrain.
- Monitoring tells you when to go round again.
- The slowest manual step decides how fast the loop can turn.
