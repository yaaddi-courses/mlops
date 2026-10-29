# Experiment tracking with MLflow

**Covers:** run, UI, AUC, MLflow, MLflow Tracking, log_metric(), GitHub

## When a spreadsheet stops being enough {#overview}
In the first week of a project, you try three settings and write the scores in a spreadsheet. That works. By the third month you have 400 attempts, five people, several datasets, and the spreadsheet is wrong. Experiment tracking replaces it with a system that records each attempt for you. Let's see what it records and how you read it.

## Run {#run}
In tracking, the unit is the run. A run is one recorded attempt at training a model: a single execution with its own settings, results and output files. Ten tries with different settings are ten runs. Each has a unique identifier, a start and end time and everything logged to it. You compare runs to find the best one.

## MLflow {#mlflow}
MLflow is a widely used open-source platform for managing the machine learning process. It has several parts: Tracking, which logs runs, a catalog for finished models and a packaging format for them. You can install it with one command and use it on your own machine, or point a whole team at one shared server.

## MLflow Tracking {#mlflow-tracking}
Tracking is the part you will use first. It logs each training attempt as a run, with its parameters, metrics and output files, and gives you a place to browse and compare them. Parameters are the settings you chose. Metrics are the numbers that measure how well the attempt did. Output files include the trained model and plots.

## log_metric() {#log-metric}
How does a number get into a run? With `log_metric()`. This MLflow function records one numeric result, such as accuracy, for the current run. You call it with a name and a value, and you can call it repeatedly with a step number to record how a metric changes during training. Its partner `log_param()` records a setting.

## AUC {#auc}
The metric in our example is AUC, the area under the curve. It is a score from 0 to 1 for how well a classifier separates two classes. A value of 0.5 means the model does no better than guessing, and 1.0 means perfect separation. People like to log it because it does not depend on choosing a cut-off.

## UI {#ui}
Logged numbers are only useful if you can look at them. The UI, the user interface, is the set of screens and controls that people use to work with software. MLflow's UI is a web page where you list runs, sort them by a metric, select several and see their settings and curves side by side. It is where most people first see the value of tracking.

## GitHub {#github}
GitHub is the most widespread site for storing Git repositories online, with tools that let teams review each other's code. It matters here because MLflow can record the exact code version of each run. A link from a run to a commit on GitHub closes the loop: from a result you can reach the code that produced it.

## A worked example {#example}
Let's train a model three times with different settings:

```text
import mlflow

for depth in (3, 5, 8):
    with mlflow.start_run():
        mlflow.log_param("max_depth", depth)
        auc = 0.80 + 0.03 * min(depth, 5) - 0.02 * max(depth - 5, 0)
        mlflow.log_metric("auc", auc)
```

This code only simulates a score, but the logging calls are the real ones. Each pass of the loop creates one run, so there are three runs with these values:

| Run | max_depth | AUC |
|---|---|---|
| 1 | 3 | 0.89 |
| 2 | 5 | 0.95 |
| 3 | 8 | 0.89 |

Let's check the formula. Depth 3 gives 0.80 + 0.09 = 0.89. Depth 5 gives 0.80 + 0.15 = 0.95. Depth 8 gives 0.80 + 0.15 − 0.06 = 0.89. The story is that a medium depth is best, and a deeper model gets worse.

Start the UI with `mlflow ui` and open the page in a browser. You see three rows. Sort by AUC and run 2 is on top. Open it, and you see its parameter and metric, its time and its code version.

Now count the benefit. With 400 attempts, finding "the best run with depth 5" in a spreadsheet takes minutes and can be wrong. In the UI, a filter such as `params.max_depth = "5"` gives the answer in a second, and each row links to its model file and code.

Tracking pays off the day someone asks, "which settings gave the model that is in use?" With tracking, the answer is a link. Without it, the answer is "I think it was run 17, but I changed the notebook after that".

Log more than the minimum: the data version, the random seed, the library versions and the training time. Each one is cheap to record, and each answers a question that would otherwise cost hours.

## A common mistake {#mistake}
A common slip is logging only the final score. When a result changes later, you cannot tell why, so log the settings and the inputs too.

Another slip is keeping the tracking server on one laptop, so nobody else can see the results.

## Key points {#key-points}
- A run is one recorded attempt. Tracking stores its settings, metrics and files.
- `log_metric()` records numbers, and `log_param()` records settings.
- The UI lets you sort and compare runs.
- Link each run to its code version.
