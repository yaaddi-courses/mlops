# Hyperparameters and Weights & Biases sweeps

**Covers:** hyperparameter, Weights & Biases, sweep

## Choosing the settings that training cannot choose {#overview}
During training, a model learns its own internal numbers. But some settings must be fixed before training begins, and the model cannot learn them. Finding good values for these is a large part of the effort, and the good news is that you can automate it. This article covers what those settings are and how a tool searches them for you.

## Hyperparameter {#hyperparameter}
A hyperparameter is a setting you choose before training, as opposed to a value the model learns by itself. Two common ones are the learning rate, which controls the size of each step of learning, and the batch size, which is how many examples are used in each step. Others include the depth of a tree or the number of layers in a network. A poor choice can make training fail or give a mediocre model.

## Weights & Biases {#weights-and-biases}
Weights & Biases, W&B for short, is a hosted experiment-tracking tool similar to MLflow. "Hosted" means the data is kept on the company's servers, so there is nothing for you to install and operate, and sharing results with a team is easy. It is best known for two extras: live dashboards that show curves while a run is still going, and sweeps.

## Sweep {#sweep}
Trying a learning rate, then another, then another by hand gets tedious. A sweep is W&B's automated search over hyperparameters. You describe the ranges to explore and a method, and the tool launches many runs for you. Each combination becomes a run, and the dashboard shows which settings worked best.

Three methods are common. Grid search tries every combination. Random search samples combinations at random. Bayesian search uses the results so far to choose promising next ones.

## A worked example {#example}
Let's tune two hyperparameters. The learning rate can be 0.001, 0.01 or 0.1. The batch size can be 16, 32 or 64.

**By hand.** You try a few combinations that you remember to try and write the scores in a note. After 4 tries, you pick the best of those. You may have missed a better combination, and you cannot say how many you skipped.

**Grid sweep.** The tool tries all combinations: 3 × 3 = 9 runs. The configuration is a short file:

```yaml
method: grid
metric:
  name: val_accuracy
  goal: maximize
parameters:
  learning_rate:
    values: [0.001, 0.01, 0.1]
  batch_size:
    values: [16, 32, 64]
```

The tool starts 9 runs, one after another or in parallel on several machines, and collects the scores:

| Learning rate | Batch 16 | Batch 32 | Batch 64 |
|---|---|---|---|
| 0.001 | 0.86 | 0.87 | 0.85 |
| 0.01 | 0.90 | 0.92 | 0.91 |
| 0.1 | 0.70 | 0.75 | 0.78 |

The best result is 0.92, at learning rate 0.01 and batch size 32. The table also teaches you something: the learning rate matters much more than the batch size. Across rates at batch 32 the range is 0.92 − 0.75 = 0.17, but across batch sizes at rate 0.01 it is only 0.92 − 0.90 = 0.02.

**The cost of a bigger grid.** With 5 learning rates, 5 batch sizes and 4 tree depths, a grid needs 5 × 5 × 4 = 100 runs. If each takes 10 minutes, that is 1,000 minutes, almost 17 hours. A random sweep of 20 runs often finds a nearly as good result in 200 minutes, which is 3 hours and 20 minutes. That is why random search is a common default when the space is large.

A hosted tool also means that information about your runs leaves your machine. For private data, check what is uploaded. Many teams log only numbers and settings, not the data itself.

| | MLflow | Weights & Biases |
|---|---|---|
| Hosting | Your own server, or local | Hosted by the company, with a self-hosted option |
| Special strength | Registry and packaging | Sweeps and dashboards |
| Cost | Free software | Free tier, paid plans for teams |

## A common mistake {#mistake}
A common slip is tuning on the same data you use for the final report. The chosen settings fit that data, and the score is too hopeful. Keep a separate test set that the search never sees.

Another slip is a grid that is too fine, which burns hours of computing for a gain of a few thousandths.

## Key points {#key-points}
- A hyperparameter is chosen before training. The model does not learn it.
- A sweep tries many combinations automatically.
- Random search is often better value than a full grid when there are many settings.
- Tune on validation data, and keep a separate test set.
