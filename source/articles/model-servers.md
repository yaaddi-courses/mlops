# Model servers: TensorFlow Serving, TorchServe and Triton

**Covers:** model server, TensorFlow Serving, TorchServe, Triton Inference Server, TensorFlow, TF, GPU, NVIDIA

## Do not write the server yourself {#overview}
You could wrap a model in a small web program of your own. Many teams begin that way. But a serious service needs batching of requests, version management, health checks and speed tuning, and all of that is repeated work. Dedicated model servers have done it already.

## Model server {#model-server}
A model server is a dedicated piece of software that loads a trained model and exposes it over a network interface, such as REST or gRPC, handling model versions, batching and monitoring hooks. You give it a folder of model files. It listens for requests, runs the model and returns the answers.

One useful feature is dynamic batching: the server waits a few milliseconds to group requests that arrive close together and runs them as one batch. This uses the hardware better, and raises how many requests per second it can serve.

## TensorFlow {#tensorflow}
TensorFlow is a free machine learning library from Google, used to define, train and save models. TF is its common short name, used in file names and package names.

## TF {#tf}
TF is short for TensorFlow, as in `tf.keras` or "a TF model".

## TensorFlow Serving {#tensorflow-serving}
TensorFlow Serving is Google's dedicated open-source server for deploying trained TensorFlow models. It exposes them over gRPC or REST, and handles model versions: it can load a new version while the old one still answers, and switch with no gap.

## TorchServe {#torchserve}
TorchServe is the PyTorch community's equivalent: a dedicated server built for models trained in PyTorch. You package a model into an archive with a handler that describes how to prepare the input and format the output, and the server does the rest.

## GPU {#gpu}
A GPU, graphics processing unit, is a chip with thousands of small cores, used to train and run neural networks fast. Many of the same small calculations can be done at once on it. GPUs, the plural, cost much more than ordinary processors and must be kept busy to be worth it.

## NVIDIA {#nvidia}
NVIDIA is the company that makes most of the GPUs used to train and run AI models. Its software tools for these chips are part of why they dominate.

## Triton Inference Server {#triton-inference-server}
NVIDIA Triton Inference Server is a model-serving tool that can host models from several frameworks at once: TensorFlow, PyTorch, ONNX and more. It is designed to get the most from GPUs, and to serve different models from one place.

## A worked example {#example}
A company has three models: a TensorFlow model for recommendations, a PyTorch model for text, and one exported to ONNX for images.

**Option A: three servers.** TensorFlow Serving for the first, TorchServe for the second, and a custom program for the third. Three tools to learn, monitor and update.

**Option B: one Triton.** All three models in one model folder:

```text
models/
  recs/    (TensorFlow)
  text/    (PyTorch)
  images/  (ONNX)
```

One server, one API style, one place to monitor.

Now throughput on a GPU. A model takes 10 ms for one request, and 16 ms for a batch of 8, because the chip does the 8 in parallel.

- Without batching: 1 request / 0.010 s = 100 requests per second.
- With batches of 8: 8 requests / 0.016 s = 500 requests per second.

That is five times more from the same GPU. The cost: a request may wait a few milliseconds to be grouped, so the delay per request grows slightly. Set a maximum wait time such as 5 ms, to balance the two.

At a price of, say, 2 dollars an hour for a GPU, serving 500 requests per second at 100 per second per server needs 5 servers, which is 10 dollars an hour. With batching, one GPU is enough: 2 dollars an hour. The batching saved 8 dollars an hour, which is about 5,800 dollars a month (8 × 24 × 30 = 5,760).

| Server | Best for | Limit |
|---|---|---|
| TensorFlow Serving | TensorFlow models | Mostly that framework |
| TorchServe | PyTorch models | Mostly that framework |
| Triton | Mixed frameworks, GPU efficiency | More complex configuration |

When the load is low, a GPU can sit idle most of the time, and a CPU server may be cheaper. Measure the real demand before buying hardware.

## A common mistake {#mistake}
Putting a model behind a hand-written web program and later discovering that it has no batching, no version switching and no health checks. Another mistake is using a GPU for a small model with little traffic, and paying for an idle chip.

## Key points {#key-points}
- A model server loads models and serves them over a network, with versions and batching.
- TensorFlow Serving is for TensorFlow models and TorchServe is for PyTorch models. Triton serves several frameworks.
- Batching can raise a GPU's throughput several times.
- Match the hardware to real demand.
