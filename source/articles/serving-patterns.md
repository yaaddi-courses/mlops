# Serving patterns: batch, online and streaming

**Covers:** batch inference, online inference, streaming inference, API, HTTP, OS

## Three ways to deliver predictions {#overview}
A trained model is only useful when someone receives its predictions. How they should arrive depends on how fast they are needed and how many there are. Three patterns cover almost every case, and picking the wrong one is costly: either too slow for the user, or far more expensive than it needs to be. Let's meet them one by one, and then the three technical terms that sit underneath them.

## Batch inference {#batch-inference}
In batch inference, you apply a trained model to a large pile of stored data at once, on a schedule, with nobody waiting for an immediate answer. The results are written to a table or a file and read later. A nightly job that scores every customer for churn risk is a typical case.

## Online inference {#online-inference}
Online inference, also called real-time inference, answers one prediction request at a time through a live call, with a reply expected in milliseconds. Think of a card payment that is checked for fraud while the customer waits. The model must always be up and ready.

## Streaming inference {#streaming-inference}
Streaming inference sits between the other two. The model works continuously on a live feed of events as they arrive. Results come within seconds, not milliseconds, and the work is done in small continuous pieces instead of one big nightly job. A monitor that scores sensor readings from machines as they arrive is a typical case.

## API {#api}
How does an application ask for a prediction? Through an API, an application programming interface, which is a defined way for one program to ask another for something. In serving, the application sends features and gets a prediction back. The caller needs to know only the format of the request and the reply, and nothing about the model inside.

## HTTP {#http}
Most online model APIs speak HTTP, the HyperText Transfer Protocol that browsers and servers use to exchange pages and data. A request carrying the input travels to a server address, and the reply carries the prediction, usually as JSON text.

## OS {#os}
OS stands for operating system, the software that manages a computer's hardware and starts other programs. Serving software sits on top of one, and differences between systems, such as library versions, are a common cause of "works on my machine" trouble. Packaging the model together with its environment avoids it.

## A worked example {#example}
Let's match three requests to three patterns.

**1. A monthly marketing list.** The company wants a score for each of 2 million customers, once a month, to decide who gets an offer. Nobody is waiting, so batch is right.
- Scoring speed: 5,000 customers per second.
- Time for the whole job: 2,000,000 / 5,000 = 400 seconds, under 7 minutes.
- The job starts on a schedule at night, and the machine can be switched off afterwards, so the cost is low.

**2. A payment check.** A customer taps "pay" and waits. The reply is needed within 100 milliseconds, so online is right. The API receives the payment details over HTTP, the model scores them in about 20 ms, and the reply says "allow" or "review".
- Peak traffic: 300 requests per second.
- If one server handles 100 per second, you need three at the peak, plus a spare. The cost is higher because they stay on all day.

**3. Machine sensors.** 5,000 machines send a reading every 10 seconds, which is 500 readings per second. Alarms matter within a few seconds, but not within milliseconds. Streaming is right: a continuous job reads the feed, scores each reading and raises alerts.

| | Batch | Online | Streaming |
|---|---|---|---|
| Delay of results | Hours | Milliseconds | Seconds |
| Input | Stored table | One request | Live events |
| Compute | Only while working | Always on | Always on |
| Cost per prediction | Lowest | Highest | Medium |

Now imagine doing the first case online. Serving 2 million requests spread over a month with always-on servers costs far more than a 7-minute job. And using batch for the payment check is impossible, since the answer would arrive hours after the customer left.

A hybrid is common. Score everyone in batch overnight and store the results, then answer the online request with a simple lookup. It is fast and cheap when predictions change slowly, such as a customer's long-term interest.

The environment matters in all three cases. Deploy on the same operating system and library versions that you used for testing, or package both together.

## A common mistake {#mistake}
A common slip is choosing online serving because it sounds modern, when a nightly batch would do. It adds cost and operational burden for no benefit.

The reverse slip also happens: using stale batch predictions for a decision that depends on the last minute.

## Key points {#key-points}
- Batch scores stored data on a schedule. Online answers one request in milliseconds. Streaming scores a live feed.
- Pick by how fast the answer is needed.
- Online and streaming need always-on systems, so they cost more.
- Package the environment with the model.
