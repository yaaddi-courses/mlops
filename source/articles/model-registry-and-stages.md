# The model registry, lifecycle stages and governance

**Covers:** model registry, model registries, lifecycle stage, model governance

## A saved file is not enough {#overview}
After training, a model is only a file. Putting it in a shared folder under a name like `model_final_v2_REAL.pkl` works until the second person touches it. A team needs to know which version is live, which one is being tested, who approved it and how to go back. A registry answers these questions, and this article shows how.

## Model registry {#model-registry}
A model registry is a central catalog of trained model versions. Each version has details attached and a lifecycle stage, so a team can find a model, promote it or return to an older one by name, without hunting for files. A registered model has a name, such as `churn-predictor`, and numbered versions: 1, 2, 3. Each version links to the run that produced it, so you can see its settings, data and scores.

## Model registries {#model-registries}
Registries, in the plural, come in several products: the one in MLflow, those of the big cloud providers and others. They differ in detail but share one idea, a single place that is the source of truth for which model is in use.

## Lifecycle stage {#lifecycle-stage}
Each version in the registry carries a label called its lifecycle stage. The usual ones are Staging, where the version is being tested, Production, where it serves real traffic, and Archived, where it is retired but kept for the record. Moving a version between stages is an explicit, recorded act. Newer tools also offer aliases, such as "champion", which can point at any version you choose.

## Model governance {#model-governance}
Who decided that a model was fit for use, and on what evidence? Model governance is the set of practices that answers that question. It makes sure a model is reviewed and understood before it affects people, through documentation, approval steps and checks for fairness. In regulated fields such as lending or health, it is a legal need as well as good practice.

## A worked example {#example}
Let's look at a team with a registered model `churn-predictor`.

| Version | Stage | Note |
|---|---|---|
| 1 | Archived | First model, replaced in March |
| 2 | Production | Serving customers now |
| 3 | Staging | New candidate, being tested |

Version 3 has finished training. The sequence goes like this:

1. Register it: it appears as version 3, linked to its run.
2. Move it to Staging. A test job reads the model by name and stage, without knowing the file's location.
3. The tests pass, and a reviewer signs off. The team moves version 3 to Production, and version 2 to Archived.
4. The serving code asks the registry for "the model called churn-predictor in Production", so it picks up version 3 at its next refresh. No file was copied and nobody edited the serving code.

A week later, version 3 misbehaves. The fix takes one step: move version 2 back to Production. Going back takes seconds, and a record shows who did it and when.

Compare that with the shared folder, where you find `model_v2.pkl`, `model_v2_new.pkl` and `model_final.pkl`. Which is live? Nobody can say. Suppose finding the right file after an incident takes 45 minutes, and the incident costs 2,000 dollars per hour in lost sales. The shared folder adds 45/60 × 2,000 = 1,500 dollars to every incident. A registry brings that cost close to zero.

Governance adds the human side. Before the move to Production, the registry can require:

- a link to the model's documentation,
- the results of tests,
- a named approver.

For a low-risk model such as a movie recommender, one approver may be enough. For a model that decides loans, the checks are stricter and may include a second reviewer and a fairness report. The registry keeps the record of all this next to the version.

| Question | Where the answer lives |
|---|---|
| Which version is live? | Stage Production in the registry |
| What data and settings produced it? | The linked run |
| Who approved it and when? | The governance record |
| How do we go back? | Move the old version back |

## A common mistake {#mistake}
A common slip is using the registry only as a list while still copying files by hand into production. Make the serving code read from the registry, so the stage is the real switch.

Another slip is skipping governance for "small" models that later grow in importance.

## Key points {#key-points}
- A registry gives each model a name, numbered versions and a stage.
- Stages such as Staging, Production and Archived make promotion and returning to an old version explicit.
- Governance records who approved a model and on what evidence.
- Serving code should read the registry, not a file path.
