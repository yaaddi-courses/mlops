# Packaging a model: artifacts, inference code and ONNX

**Covers:** artifact, model artifact, inference, inference code, ONNX, PyTorch, MP3, USB

## What actually gets shipped {#overview}
When you deploy a model, you do not ship the training notebook. You ship the result of training together with a small amount of code that uses it. Getting this package right decides whether the model behaves in production the way it did in testing. Let's look at what goes into the package, one piece at a time.

## Artifact {#artifact}
In experiment tracking, an artifact is any output file that a training run saves and logs next to its settings and metrics: a trained model file, a plot, a list of the features used. Artifacts are stored with the run, so you can fetch them later by the run's identifier.

## Model artifact {#model-artifact}
Of all the artifacts, one matters most for deployment. A model artifact is the packaged output of training: the trained weights plus information such as the framework and its version. It is the thing you deploy. The extra information matters as much as the weights, because a file saved with one version of a library may fail to load, or load wrongly, with another.

## Inference {#inference}
Inference means using a trained model on new input to get a prediction, as opposed to training, which produced the model. Training happens occasionally and takes hours. Inference happens constantly and often has to take milliseconds.

## Inference code {#inference-code}
Weights alone cannot make a prediction, so they travel with inference code. This is the small amount of code that loads the weights and turns new input into a prediction. It also prepares the input, for example with the same scaling used in training.

## PyTorch {#pytorch}
PyTorch is a free Python library, widely used in research and industry, for building and training neural networks. A model saved by it is tied by default to PyTorch's own format, so using it elsewhere needs PyTorch or a conversion.

## ONNX {#onnx}
That conversion is where ONNX comes in. ONNX, the Open Neural Network Exchange, is a shared open file format for trained models. A model trained in one framework, say PyTorch, can be exported and used in another place with a different tool. It separates where a model was trained from where it is used, which suits deployment on varied hardware.

## MP3 {#mp3}
MP3 is a compressed audio file format. It is not a machine learning model format, and it appears here as a contrast: it stores sound, not the weights of a model. The point is that a file's extension and its purpose must match.

## USB {#usb}
USB, Universal Serial Bus, is the standard cable connection for plugging devices into a computer. It carries data between a computer and a device, but it is not a format for models. Like MP3, it is a contrast: a way to connect hardware, not a way to package a model.

## A worked example {#example}
Let's say a team trains an image classifier in PyTorch. The training run produces files, and all of them are artifacts:

| File | Size | Role |
|---|---|---|
| `weights.pt` | 90 MB | Trained weights |
| `labels.json` | 2 KB | Names of the classes |
| `confusion.png` | 40 KB | A plot for the report |
| `env.txt` | 1 KB | Library versions |

The model artifact is `weights.pt` plus `labels.json` plus the version information in `env.txt`. The plot belongs to the run but is not needed in production.

The inference code for this model has three parts:

1. Load the weights once, at start-up.
2. Resize and scale the input image exactly as in training.
3. Apply the model and map the highest score to a name from `labels.json`.

Here is a frequent bug. In training, images were scaled to values between 0 and 1, but the serving code forgets to do that and passes values from 0 to 255. Predictions become nonsense, with no error message. That is why the preparation steps must travel in the package.

To use the model somewhere without PyTorch, export it to ONNX:

```text
torch.onnx.export(model, example_input, "model.onnx")
```

An ONNX runtime can then use the exported file on a server, in a browser or on a phone. Check the result by feeding the same 100 test images to both versions and comparing the predictions. They should match to within a tiny rounding difference. If 99 of 100 match, investigate the one that does not.

Size and speed may also change. A 90 MB PyTorch file might become an 88 MB ONNX file, and inference on a CPU might drop from 40 ms to 25 ms per image, thanks to optimizations in the runtime. Those numbers are examples, and real gains vary.

Do not confuse formats. A file named `model.mp3` is not a model, and a USB stick is a way to carry files, not a packaging format. Keep names honest: `.pt` for PyTorch weights, `.onnx` for ONNX.

## A common mistake {#mistake}
A common slip is shipping weights without the preparation code or the library versions. The model loads and returns wrong answers. Package the code and the metadata with the weights.

Another slip is exporting to ONNX and never comparing the outputs.

## Key points {#key-points}
- An artifact is a file saved by a run. A model artifact is the weights plus the metadata needed to use them.
- Inference is using a trained model. Inference code includes the input preparation.
- ONNX lets a model move between tools. MP3 and USB are not model formats.
- Compare predictions before and after any export.
