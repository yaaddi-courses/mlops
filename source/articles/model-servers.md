# Model servers: TensorFlow Serving, TorchServe and Triton

**Covers:** model server, TensorFlow Serving, TorchServe, Triton Inference Server, TensorFlow, TF, GPU, NVIDIA

## Do not write the server yourself {#overview}
You could wrap a model in a small web program of your own, and many teams begin that way. But a serious service also needs batching of requests, version management, health checks and speed tuning, and writing all that is repeated work. Dedicated model servers have already done it. Let's see what they offer and when to pick which.

## Model server {#model-server}
A model server is dedicated software that loads a trained model and exposes it over a network interface, such as REST or gRPC, with model versions, batching and monitoring hooks built in. You give it a folder of model files. It listens for requests, applies the model and returns the answers.

One feature is worth knowing now: dynamic batching. The server waits a few milliseconds to group requests that arrive close together and processes them as one batch. That uses the hardware better and raises the number of requests per second it can serve.

## TensorFlow {#tensorflow}
TensorFlow is a free machine learning library from Google, used to define, train and save models. Its common short name, TF, appears in file and package names.

## TF {#tf}
TF is short for TensorFlow, as in `tf.keras` or "a TF model".

## TensorFlow Serving {#tensorflow-serving}
TensorFlow Serving is Google's open-source server for deploying trained TensorFlow models. It offers them over gRPC or REST and manages versions: it can load a new version while the old one still answers, and then switch with no gap.

## TorchServe {#torchserve}
TorchServe is the PyTorch community's counterpart, built for models trained in PyTorch. You package a model into an archive with a handler that says how to prepare the input and format the output, and the server handles the rest.

## GPU {#gpu}
A GPU, a graphics processing unit, is a chip with thousands of small cores, used to train and serve neural networks quickly. It can perform many of the same small calculations at once. GPUs cost much more than ordinary processors, so you must keep them busy to make them worth it.

## NVIDIA {#nvidia}
NVIDIA is the company that makes most of the GPUs used for AI models. Its software tools for these chips are part of why it dominates.

## Triton Inference Server {#triton-inference-server}
NVIDIA Triton Inference Server can host models from several frameworks at once: TensorFlow, PyTorch, ONNX and more. It is designed to get the most from GPUs and to serve different models from one place.

## A worked example {#example}
Imagine a company with three models: a TensorFlow model for recommendations, a PyTorch model for text, and one exported to ONNX for images.

**Option A: three servers.** TensorFlow Serving for the first, TorchServe for the second and a custom program for the third. That is three tools to learn, monitor and update.

**Option B: one Triton.** All three models in one model folder:

```text
models/
  recs/    (TensorFlow)
  text/    (PyTorch)
  images/  (ONNX)
```

You get one server, one API style and one place to monitor.

Now let's look at throughput on a GPU. A model takes 10 ms for one request, and 16 ms for a batch of 8, because the chip handles the 8 in parallel.

- Without batching: 1 request / 0.010 s = 100 requests per second.
- With batches of 8: 8 requests / 0.016 s = 500 requests per second.

That is five times more from the same GPU. The price is that a request may wait a few milliseconds to be grouped, so the delay per request grows a little. Choose a maximum wait such as 5 ms to balance the two.

Put money on it. Say a GPU costs 2 dollars an hour. To serve 500 requests per second at 100 per second per server, you need 5 servers, which is 10 dollars an hour. With batching, one GPU is enough: 2 dollars an hour. Batching saves 8 dollars an hour, about 5,760 dollars a month (8 × 24 × 30).

| Server | Best for | Limit |
|---|---|---|
| TensorFlow Serving | TensorFlow models | Mostly that framework |
| TorchServe | PyTorch models | Mostly that framework |
| Triton | Mixed frameworks, GPU efficiency | More complex configuration |

When the load is low, a GPU can sit idle most of the time, and a CPU server may cost less. Measure the real demand before buying hardware.

## A common mistake {#mistake}
A common slip is putting a model behind a hand-written web program and finding out later that it has no batching, no version switching and no health checks.

Another slip is using a GPU for a small model with little traffic, and paying for an idle chip.

## Key points {#key-points}
- A model server loads models and serves them over a network, with versions and batching.
- TensorFlow Serving is for TensorFlow models and TorchServe is for PyTorch models. Triton serves several frameworks.
- Batching can raise a GPU's throughput several times.
- Match the hardware to real demand.
