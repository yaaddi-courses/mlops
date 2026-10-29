# Packaging a model: artifacts, inference code and ONNX

**Covers:** artifact, model artifact, inference, inference code, ONNX, PyTorch, MP3, USB

## What actually gets shipped {#overview}
To deploy a model, you do not ship the training notebook. You ship the result of training, together with a small amount of code that uses it. Getting this package right decides whether the model works the same in production as it did during testing.

## Artifact {#artifact}
In experiment tracking, an artifact is any output file that a training run saves and logs next to its settings and metrics: a trained model file, a plot, a list of the features used. Artifacts are stored with the run, so you can fetch them later by the run's identifier.

## Model artifact {#model-artifact}
A model artifact is the packaged output of training: the trained weights plus information such as the framework and its version. It is the thing you deploy. The metadata matters as much as the weights, since a file saved with one version of a library may fail to load, or load wrongly, with another.

## Inference {#inference}
Inference is running a trained model on new input to get a prediction, as opposed to training, the process that produced the model. Training happens occasionally and takes hours. Inference happens constantly and often must take milliseconds.

## Inference code {#inference-code}
Inference code is the small amount of code that loads the model's weights and produces a prediction from new input. It also includes the preparation of the input, such as the same scaling used in training. It travels together with the weights, since weights alone cannot be used.

## PyTorch {#pytorch}
PyTorch is a free Python library, widely used in research and industry, for building and training neural networks. Models saved by it are, by default, tied to PyTorch's own format, which means that using them elsewhere needs PyTorch or a conversion.

## ONNX {#onnx}
ONNX, the Open Neural Network Exchange, is a shared open file format for trained models. It lets a model trained in one framework, say PyTorch, be exported and used in another place, with a different tool. It separates "where the model was trained" from "where it is used", which suits deployment on varied hardware.

## MP3 {#mp3}
MP3 is a compressed audio file format. It is not a machine learning model format, and appears here only as a contrast: a format that stores sound, not the weights of a model. The point is that file extension and purpose must match.

## USB {#usb}
USB, Universal Serial Bus, is the standard cable connection for plugging devices into a computer. It carries data between a computer and a device, but is not a format for models. Like MP3, it is a contrast: a way to connect hardware, not a way to package a model.

## A worked example {#example}
A team trains an image classifier in PyTorch. The training run produces files, all of them artifacts:

| File | Size | Role |
|---|---|---|
| `weights.pt` | 90 MB | Trained weights |
| `labels.json` | 2 KB | Names of the classes |
| `confusion.png` | 40 KB | A plot for the report |
| `env.txt` | 1 KB | Library versions |

The model artifact is `weights.pt` plus `labels.json` plus the version information in `env.txt`. The plot is an artifact of the run, but is not needed in production.

Inference code for this model has three parts:

1. Load the weights once, at start-up.
2. Resize and scale the input image exactly as in training.
3. Run the model and map the highest score to a name from `labels.json`.

A frequent bug: in training, images were scaled to values between 0 and 1, but the serving code forgets to do it, and passes values 0 to 255. Predictions become nonsense, with no error. So the preparation steps must travel in the package.

To use the model somewhere without PyTorch, export it to ONNX:

```text
torch.onnx.export(model, example_input, "model.onnx")
```

The exported file can be run by an ONNX runtime, on a server, in a browser or on a phone. Check the result: feed the same 100 test images to both versions and compare the predictions. They should match to within a tiny rounding difference. If 99 of 100 match, investigate the 1.

Size and speed also change. A 90 MB PyTorch file may become an 88 MB ONNX file, and inference on a CPU may speed up, say from 40 ms to 25 ms per image, thanks to optimizations in the runtime. These are examples, and real gains vary.

Do not confuse formats. A file named `model.mp3` is not a model, and a USB stick is a way to carry files, not a packaging format. Keep names honest: `.pt` for PyTorch weights, `.onnx` for ONNX.

## A common mistake {#mistake}
Shipping weights without the preparation code, or without the library versions. The model loads and returns wrong answers. Package the code and the metadata with the weights. Another mistake is exporting to ONNX and never comparing the outputs.

## Key points {#key-points}
- An artifact is a file saved by a run. A model artifact is the weights plus the metadata needed to use them.
- Inference is using a trained model. Inference code includes the input preparation.
- ONNX lets a model move between tools. MP3 and USB are not model formats.
- Compare predictions before and after any export.
