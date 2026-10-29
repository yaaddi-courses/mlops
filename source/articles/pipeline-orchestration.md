# Pipeline orchestration: DAGs, Kubeflow, Airflow and Prefect

**Covers:** DAG, Kubeflow Pipelines, component, K8s, Airflow, Prefect, YAML, SQL

## From a pile of scripts to a pipeline {#overview}
A machine learning workflow has several steps: fetch the data, clean it, build features, train, evaluate and publish. At first, someone runs five scripts by hand, in the right order. That works until the person is ill, a script fails halfway, or the order changes. A pipeline tool takes over the order, the retries and the schedule.

## DAG {#dag}
A DAG, a directed acyclic graph, is the shape most pipeline tools use to describe a workflow: steps connected by arrows that show what depends on what, with no loops. "Directed" means each arrow has a direction. "Acyclic" means you can never follow the arrows back to where you started. Because there are no loops, the tool can always find an order in which every step runs after the ones it needs, and it can run independent steps at the same time.

## SQL {#sql}
SQL, Structured Query Language, is the standard language for querying and changing data in relational databases. Many pipeline steps are SQL: selecting the training rows or computing a summary. Several tools can run a SQL file as one step of a DAG.

## YAML {#yaml}
YAML is a human-readable text format for configuration, based on indentation. Pipelines are often described in YAML files that list steps, their inputs and their settings. Indentation carries meaning, so a misplaced space can change what the file says.

## K8s {#k8s}
K8s is the short name of Kubernetes: the letter K, then eight letters, then s. Kubernetes is the system that runs containers across many machines. Some pipeline tools are built on it, which brings scaling and isolation, and also brings the cost of learning it.

## Kubeflow Pipelines {#kubeflow-pipelines}
Kubeflow Pipelines is an orchestration tool built for machine learning that runs on Kubernetes. Each step of the pipeline executes inside its own container, so it has its own software and cannot disturb the other steps. You write the pipeline in Python, and it is compiled into a YAML description.

## Component {#component}
In Kubeflow Pipelines, a component is one packaged, reusable step, such as "preprocess" or "train", bundled with its own code and its software requirements. A component declares its inputs and outputs, so that it can be reused in other pipelines. Components, the plural, are the building blocks.

## Airflow {#airflow}
Apache Airflow is a widely used open-source tool for scheduling and running workflows defined as DAGs. It is general-purpose and not built for machine learning, but it is common in data teams. It has a web page showing every execution, which steps succeeded, and logs for those that failed.

## Prefect {#prefect}
Prefect is an open-source workflow orchestrator, like Airflow, but built around writing flows as plain Python functions. A flow can branch and change shape while it executes, which a fixed DAG finds harder.

## A worked example {#example}
A pipeline with five steps: `fetch`, `clean`, `features`, `train`, `evaluate`.

```text
fetch -> clean -> features -> train -> evaluate
```

This is a simple chain, which is also a DAG. Now add a second data source, with its own cleaning:

```text
fetch_a -> clean_a ---\
                       +-> features -> train -> evaluate
fetch_b -> clean_b ---/
```

The two branches have no arrow between them, so the tool can run them in parallel. If each fetch-and-clean takes 20 minutes, sequential execution takes 40, parallel takes 20. Together with features (10 minutes), training (30) and evaluation (5), the total goes from 40 + 45 = 85 minutes to 20 + 45 = 65 minutes, a saving of 20 minutes each time.

Why no loops? An arrow from `evaluate` back to `fetch_a` would be a cycle, and the tool could not decide where to start. Retraining is handled by running the whole DAG again, on a schedule.

Failure handling shows the value. Suppose `train` fails at 3 a.m. With scripts run by hand, someone restarts everything from `fetch`, losing 65 minutes. With an orchestrator, the earlier steps are marked done, and only `train` is retried, costing 30 minutes at most, with no human awake.

In Airflow, the DAG is defined in Python. In Kubeflow, each step is a component in a container, so the training step may use a different software version from the cleaning step without conflict. In Prefect, a normal function marked as a flow is enough.

| Tool | Best for | Trade-off |
|---|---|---|
| Airflow | Scheduled data and ML workflows | Fixed DAG, heavier set-up |
| Prefect | Flexible, Python-first flows | Smaller community than Airflow |
| Kubeflow Pipelines | ML on Kubernetes, isolated steps | Needs a Kubernetes cluster |

## A common mistake {#mistake}
Choosing the heaviest tool for a three-step job. A scheduled script may be enough until the steps multiply. Another mistake is steps that depend on hidden state, such as a file left in a folder, so that a rerun gives a different result.

## Key points {#key-points}
- A DAG has steps, arrows for dependencies, and no loops. Independent branches run in parallel.
- Orchestrators handle order, retries, schedules and logs.
- Kubeflow Pipelines packages each step as a container component. Airflow and Prefect are general tools.
- Retry only the failed step, not the whole pipeline.
