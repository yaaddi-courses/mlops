# The model registry, lifecycle stages and governance

**Covers:** model registry, model registries, lifecycle stage, model governance

## A saved file is not enough {#overview}
After training, a model is just a file. Putting it in a shared folder with a name like `model_final_v2_REAL.pkl` works until the second person touches it. A team needs to know which version is live, which is being tested, who approved it and how to go back. A registry answers these questions.

## Model registry {#model-registry}
A model registry is a central catalog of trained model versions, each with details attached and a lifecycle stage, so that a team can find, promote a model or return to an older one by name instead of by hunting for files. A registered model has a name, such as `churn-predictor`, and numbered versions: 1, 2, 3. Each version links to the run that produced it, so you can see its settings, data and scores.

## Model registries {#model-registries}
Model registries, the plural, exist in several products: the one in MLflow, those of the big cloud providers, and others. They differ in detail but share the same idea: one place that is the source of truth for which model is in use.

## Lifecycle stage {#lifecycle-stage}
A lifecycle stage is a label on a model version in the registry. The common ones are Staging, where the version is being tested, Production, where it serves real traffic, and Archived, where it is retired but kept for the record. Moving a version from one stage to another is an explicit, recorded act. Newer tools also offer aliases, such as "champion", which can point at any version you choose.

## Model governance {#model-governance}
Model governance is the set of practices that make sure a model is reviewed and understood before it affects people: documentation, approval steps and checks for fairness. It answers the question "who decided that this model was fit for use, and on what evidence?". In regulated fields such as lending or health, it is a legal need as well as good practice.

## A worked example {#example}
A team has a registered model `churn-predictor`.

| Version | Stage | Note |
|---|---|---|
| 1 | Archived | First model, replaced in March |
| 2 | Production | Serving customers now |
| 3 | Staging | New candidate, being tested |

Version 3 finishes training. The sequence:

1. Register it: it appears as version 3, linked to its run.
2. Set its stage to Staging. A test job reads the model by the name and stage, without knowing the file's location.
3. The tests pass, and a reviewer signs off. The team moves version 3 to Production, and version 2 to Archived.
4. The serving code asks the registry for "the model called churn-predictor in Production", so it picks up version 3 at its next refresh. No file was copied and no one edited the serving code.

A week later, version 3 misbehaves. The fix takes one step: move version 2 back to Production. Going back takes seconds, and a record shows who did it and when.

Compare this with the shared folder. Files `model_v2.pkl`, `model_v2_new.pkl`, `model_final.pkl`. Which is live? Who knows. Suppose finding the right file after an incident takes 45 minutes, and the incident costs 2,000 dollars per hour in lost sales. The shared folder adds 45/60 × 2,000 = 1,500 dollars to every incident. A registry brings that cost close to zero.

Governance adds the human side. Before the move to Production, the registry can require:

- a link to the model's documentation,
- the results of tests,
- a named approver.

For a low-risk model such as a movie recommender, one approver may suffice. For a model that decides loans, the checks are stricter and may include a second reviewer and a fairness report. The registry keeps the record of all of this next to the version.

| Question | Where the answer lives |
|---|---|
| Which version is live? | Stage Production in the registry |
| What data and settings produced it? | The linked run |
| Who approved it and when? | The governance record |
| How do we go back? | Move the old version back |

## A common mistake {#mistake}
Using the registry only as a list, and still copying files by hand into production. Make the serving code read from the registry, so the stage is the real switch. Another mistake is skipping governance for "small" models that later grow in importance.

## Key points {#key-points}
- A registry gives each model a name, numbered versions and a stage.
- Stages such as Staging, Production and Archived make promotion and returning to an old version explicit.
- Governance records who approved a model and on what evidence.
- Serving code should read the registry, not a file path.
