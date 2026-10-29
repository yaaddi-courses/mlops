# Feature stores and Feast

**Covers:** feature store, offline store, online store, training-serving skew, Feast, FeatureView, DRY

## One definition of every input {#overview}
A model does not read raw data. It reads features: numbers calculated from the data, such as "the number of purchases in the last 30 days". You need the same feature twice. In training, you compute it over millions of historical rows. In live use, you compute it for one customer in a few milliseconds. When two teams write that calculation twice, trouble follows, and this article shows why and how a feature store prevents it.

## DRY {#dry}
Let's start with a principle. DRY stands for "don't repeat yourself": every fact should live in exactly one place. If the rule for "purchases in the last 30 days" exists in two programs, they will drift apart sooner or later, and one will be updated without the other. A feature store applies this principle to features.

## Feature store {#feature-store}
A feature store is a shared system that computes and stores the input features models use. It has two halves: an offline store for training and an online store for live predictions. Teams define each feature once, and every model gets it from there.

## Offline store {#offline-store}
The offline store is the half built for training. It holds large batches of historical feature values and is tuned for scanning a lot of data at once. It can answer "what was the value of this feature for each customer on each day last year?", which is exactly what a training set needs.

## Online store {#online-store}
The online store is the half built for live predictions. It holds only the latest value of each feature and is tuned to answer one lookup in a few milliseconds, such as "the features for customer 1234, now".

## Training-serving skew {#training-serving-skew}
Now the problem the store exists to prevent. Training-serving skew is a mismatch between the features a model saw in training and the ones it sees in production. A common cause is two separate pieces of code that compute the same feature in slightly different ways. The model then receives inputs unlike those it learned from, and its quality falls, often without any error message.

## Feast {#feast}
Feast is a popular open-source feature store, real software that implements the offline and online split. Teams define features once in code, and Feast serves them both ways. It does not do heavy transformations itself. It organizes, stores and serves features computed from your data sources.

## FeatureView {#featureview}
In Feast, related features that come from the same data source are grouped into a FeatureView, so they can be defined, versioned and reused as one unit. It names the source, the key that identifies each row (such as the customer), the features and how long a value stays valid.

## A worked example {#example}
Imagine a bank with a model that predicts loan default. One feature is `avg_balance_30d`, the average account balance over the last 30 days.

**Without a feature store.**
- The training team computes it in a database query over the history, using calendar days.
- The serving team computes it in the application, using the last 30 transactions instead of 30 days.
- For an active customer with 90 transactions a month, the two numbers differ a lot. Say the training value is 4,200 and the serving value is 5,100. The model learned from one definition and is being fed another. That is skew. Accuracy falls by a few points, and nobody sees why.

**With Feast.**

```text
customer_stats = FeatureView(
    name="customer_stats",
    entities=[customer],
    ttl=timedelta(days=1),
    schema=[Field(name="avg_balance_30d", dtype=Float32)],
    source=transactions_source,
)
```

The definition is written once. For training, Feast reads the offline store and returns the value as it was on each historical date, which avoids leaking information from the future. For a live request, Feast reads the online store and returns the latest value for customer 1234. The same definition feeds both.

Let's count the effort. Two teams with two implementations need a bug fix applied in two places. With a store, there is one definition and one fix.

A second benefit is reuse. If five models need `avg_balance_30d`, they all use the one definition, computed once. Without a store, five teams each compute it, which takes five times the computing and gives five slightly different numbers.

| | Offline store | Online store |
|---|---|---|
| Used for | Training | Live prediction |
| Holds | Full history | Latest values |
| Optimized for | Large scans | Single fast lookups |
| Typical speed need | Minutes | Milliseconds |

Setting up a feature store costs time. For one small model with a handful of simple features, it may be too much. It pays off when many models share features, or when skew has already caused trouble.

## A common mistake {#mistake}
A common slip is adopting a feature store for a single model and then spending more time on the platform than on the model.

Another slip is forgetting the validity period, so a stale feature value is served as if it were fresh.

## Key points {#key-points}
- A feature store defines each feature once and serves it for training and for live use.
- The offline store keeps history. The online store keeps the latest value for fast lookup.
- Training-serving skew comes from computing a feature two ways.
- Feast is an open-source feature store, organized around FeatureViews.
