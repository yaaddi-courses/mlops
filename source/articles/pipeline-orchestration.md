# Pipeline orchestration: DAGs, Kubeflow, Airflow and Prefect

**Covers:** DAG, Kubeflow Pipelines, component, K8s, Airflow, Prefect, YAML, SQL

## From a pile of scripts to a pipeline {#overview}
A machine learning workflow has several steps: fetch the data, clean it, build features, train, evaluate and publish. At first, someone starts five scripts by hand, in the right order. That works until the person is ill, a script fails halfway, or the order changes. A pipeline tool takes over the order, the retries and the schedule. Let's meet the vocabulary, starting with the shape of a workflow.

## DAG {#dag}
Most pipeline tools describe a workflow as a DAG, a directed acyclic graph. It sounds heavy, but the idea is small: steps connected by arrows that show what depends on what, with no loops. "Directed" means each arrow has a direction. "Acyclic" means you can never follow the arrows back to where you started. Because there are no loops, the tool can always find an order in which every step comes after the ones it needs, and it can start independent steps at the same time.

## SQL {#sql}
SQL, Structured Query Language, is the standard language for querying and changing data in relational databases. Many pipeline steps are SQL, such as selecting the training rows or computing a summary. Several tools can take a SQL file as one step of a DAG.

## YAML {#yaml}
Pipelines are often described in YAML, a human-readable text format for configuration based on indentation. The files list the steps, their inputs and their settings. Indentation carries meaning, so a misplaced space can change what the file says.

## K8s {#k8s}
K8s is the short name of Kubernetes: the letter K, then eight letters, then s. Kubernetes is the system that manages containers across many machines. Some pipeline tools are built on it, which brings scaling and isolation, and also brings the cost of learning it.

## Kubeflow Pipelines {#kubeflow-pipelines}
Kubeflow Pipelines is an orchestration tool built for machine learning on Kubernetes. Each step of the pipeline executes inside its own container, so it has its own software and cannot disturb the other steps. You write the pipeline in Python, and it is compiled into a YAML description.

## Component {#component}
In Kubeflow Pipelines, the building block is the component: one packaged, reusable step, such as "preprocess" or "train", bundled with its own code and software requirements. A component declares its inputs and outputs, so you can reuse it in other pipelines.

## Airflow {#airflow}
Apache Airflow is a widely used open-source tool for scheduling and running workflows defined as DAGs. It is general-purpose, not built for machine learning, but data teams use it a lot. Its web page shows every execution, which steps succeeded, and the logs of those that failed.

## Prefect {#prefect}
Prefect is an open-source workflow orchestrator in the same family as Airflow, built around writing flows as plain Python functions. A flow can branch and change shape while it executes, which a fixed DAG finds harder.

## A worked example {#example}
Let's take a pipeline with five steps: `fetch`, `clean`, `features`, `train`, `evaluate`.

```text
fetch -> clean -> features -> train -> evaluate
```

This is a simple chain, and a chain is also a DAG. Now add a second data source with its own cleaning:

```text
fetch_a -> clean_a ---\
                       +-> features -> train -> evaluate
fetch_b -> clean_b ---/
```

The two branches have no arrow between them, so the tool can execute them in parallel. If each fetch-and-clean takes 20 minutes, doing them one after the other takes 40 and doing them together takes 20. Add features (10 minutes), training (30) and evaluation (5), and the total goes from 40 + 45 = 85 minutes to 20 + 45 = 65 minutes. You save 20 minutes every time.

Why no loops? An arrow from `evaluate` back to `fetch_a` would be a cycle, and the tool could not decide where to start. Retraining happens by executing the whole DAG again, on a schedule.

Failure handling shows the value. Suppose `train` fails at 3 a.m. With scripts started by hand, someone restarts everything from `fetch` and loses 65 minutes. With an orchestrator, the earlier steps are marked done, and only `train` is retried, costing 30 minutes at most, with no human awake.

The tools differ in style. In Airflow, you define the DAG in Python. In Kubeflow, each step is a component in a container, so the training step can use a different software version from the cleaning step without conflict. In Prefect, a normal function marked as a flow is enough.

| Tool | Best for | Trade-off |
|---|---|---|
| Airflow | Scheduled data and ML workflows | Fixed DAG, heavier setup |
| Prefect | Flexible, Python-first flows | Smaller community than Airflow |
| Kubeflow Pipelines | ML on Kubernetes, isolated steps | Needs a Kubernetes cluster |

## A common mistake {#mistake}
A common slip is choosing the heaviest tool for a three-step job. A scheduled script may be enough until the steps multiply.

Another slip is steps that depend on hidden state, such as a file left in a folder, so a second attempt gives a different result.

## Key points {#key-points}
- A DAG has steps, arrows for dependencies, and no loops. Independent branches can go in parallel.
- Orchestrators handle order, retries, schedules and logs.
- Kubeflow Pipelines packages each step as a container component. Airflow and Prefect are general tools.
- Retry only the failed step, not the whole pipeline.
