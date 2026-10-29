# What MLOps solves

**Covers:** MLOps, CPUs

## A model that works on your laptop is not a product {#overview}
Picture a data scientist who trains a model in a notebook, sees a good score and feels finished. For the people who must rely on that model, nothing has started yet. It has to be given to real users, keep working when the data changes, and be fixed when it breaks. Those tasks differ from building the model, and they are the subject of this course.

## MLOps {#mlops}
MLOps, machine learning operations, is the collection of practices and tools that keep a model working reliably after it leaves the notebook. It borrows from DevOps, which does the same for ordinary software, and adds what is special about models. They depend on data as well as code, they degrade silently as the world changes, and two executions that look identical can give different results.

The practices fall into four groups. Versioning records exactly which data, code and settings produced a model. Automation builds, tests and releases it without manual steps. Monitoring watches it once it is live. Governance makes sure someone reviewed it before it affected people.

## CPUs {#cpus}
There is one piece of hardware worth meeting now. CPUs, central processing units, are the general-purpose processors that every computer has. They do many kinds of work, one after another, very well. For machine learning they matter for two reasons. Small models can be trained and used on a CPU without special hardware, which is cheap and easy. Large models usually need other chips, but even then a CPU controls the work and prepares the data. Knowing what a CPU is helps you judge when specialized hardware is worth its cost.

## A worked example {#example}
Let's follow a bank's team that builds a model to predict which customers will miss a payment. In the notebook, it reaches 90% accuracy on last year's data.

Here is what happens without any MLOps:

1. **Hand-over.** The notebook is emailed to an engineer, who spends two weeks rewriting it. The rewrite behaves slightly differently, and nobody notices.
2. **Silent decay.** Six months later, customer behavior has changed. Accuracy has dropped to 78%, but nobody is measuring, so the bank keeps relying on the model.
3. **A complaint.** A customer asks why the model flagged them. Nobody can say which data or settings produced that version.

Now the same story with MLOps practices:

1. The training steps are written as code, so the hand-over is a button instead of a rewrite.
2. A monitor compares live results with outcomes every week and raises an alert when the score falls under 85%.
3. Every model version records its data, code and settings, so the customer's question has an answer in minutes.

Let's count the cost of the gap. Say the fall in accuracy from 90% to 78% affects 100,000 customers a month, and each wrong decision costs the bank 20 dollars on average. At 90% accuracy, 10,000 decisions a month are wrong. At 78%, 22,000 are wrong. The extra 12,000 wrong decisions cost 12,000 × 20 = 240,000 dollars per month. A monitor that catches the fall in the first month saves most of it.

The hardware question appears early too. Training this small model, a tree-based one on a table of 2 million rows, takes about 10 minutes on a modern CPU. No special chips are needed, and the team saves money by not renting any. A model with billions of parameters would be a different case.

| Without MLOps | With MLOps |
|---|---|
| Manual hand-over | Automated build and release |
| No idea when the model gets worse | Alerts on a falling score |
| Cannot reproduce an old result | Every version records its inputs |
| Hardware chosen by habit | Hardware chosen by need |

The rest of the course follows the order in which a model matures: first the loop it lives in, then tracking, packaging, release and watching.

## A common mistake {#mistake}
A common slip is thinking that MLOps is only about tools. The tools are a means. The aim is a process in which the whole team can build, release, watch and fix a model.

Another slip is starting with the most powerful hardware before measuring what the model needs.

## Key points {#key-points}
- MLOps keeps models working reliably after the notebook stage.
- A model depends on data as well as code, and it degrades silently.
- The main groups of practice are versioning, automation, monitoring and governance.
- A CPU is enough for many small models.
