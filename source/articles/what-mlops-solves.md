# What MLOps solves

**Covers:** MLOps, CPUs

## A model that works on your laptop is not a product {#overview}
A data scientist trains a model in a notebook, sees a good score and feels finished. For the people who must rely on it, nothing has started yet. The model has to be given to real users, keep working when data changes, and be fixed when it breaks. Those tasks are different from building the model, and they are the subject of this course.

## MLOps {#mlops}
MLOps, machine learning operations, is the set of practices and tools that keep a model working reliably after it leaves the notebook. It borrows from DevOps, which does the same for ordinary software, and adds what is special to models: they depend on data as well as code, they degrade silently as the world changes, and their results can differ between two executions that look identical.

The practices fall into a few groups. Versioning means recording exactly which data, code and settings produced a model. Automation means building, testing and releasing it without manual steps. Monitoring means watching it once it is live. And governance means making sure that someone reviewed it before it affected people.

## CPUs {#cpus}
CPUs, central processing units, are the general-purpose processors that every computer has. They do many kinds of work, one after another, very well. For machine learning, they matter for two reasons. Small models can be trained and used on a CPU without special hardware, which is cheap and simple. And large models usually need other chips, but even then a CPU controls the work and prepares the data. Knowing what a CPU is helps you judge when specialized hardware is worth its cost.

## A worked example {#example}
A bank's team builds a model that predicts which customers will miss a payment. In the notebook, it reaches 90% accuracy on last year's data.

What happens without any MLOps:

1. **Hand-over.** The notebook is emailed to an engineer, who spends two weeks rewriting it. The rewrite behaves slightly differently, and nobody notices.
2. **Silent decay.** Six months later, customer behavior has changed. Accuracy has dropped to 78%, but no one is measuring, so the bank keeps relying on it.
3. **A complaint.** A customer asks why the model flagged them. Nobody can say which data or settings produced that version of the model.

Now with MLOps practices:

1. The training steps are written as code, so the hand-over is a button, not a rewrite.
2. A monitor compares live results with outcomes every week and raises an alert when the score falls under 85%.
3. Every model version records its data, code and settings, so the question from the customer has an answer in minutes.

Count the cost of the gap. If the 12-point fall in accuracy (90 to 78) affects 100,000 customers a month, and each wrong decision costs the bank 20 dollars on average, the extra cost can be estimated. Suppose accuracy 90% means 10,000 wrong decisions a month, and 78% means 22,000. The extra 12,000 wrong decisions cost 12,000 × 20 = 240,000 dollars per month. A monitor that catches the fall in the first month would save most of it.

The hardware question appears early. Training this small model, a tree-based one on a table of 2 million rows, takes about 10 minutes on a modern CPU. No special chips are needed, and the team saves money by not renting any. A model with billions of parameters would be a different case.

| Without MLOps | With MLOps |
|---|---|
| Manual hand-over | Automated build and release |
| No idea when the model gets worse | Alerts on a falling score |
| Cannot reproduce an old result | Every version records its inputs |
| Hardware chosen by habit | Hardware chosen by need |

The rest of the course follows the order in which a model matures: first the loop it lives in, then tracking, packaging, release and watching.

## A common mistake {#mistake}
Thinking that MLOps is only about tools. The tools are a means. The aim is a process in which a model can be built, released, watched and fixed by the whole team. Another mistake is to start with the most powerful hardware before measuring what the model needs.

## Key points {#key-points}
- MLOps keeps models working reliably after the notebook stage.
- A model depends on data as well as code, and it degrades silently.
- The main groups of practice are versioning, automation, monitoring and governance.
- A CPU is enough for many small models.
