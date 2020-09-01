Platform for Situated Intelligence supports running machine learning models represented in the [ONNX](https://github.com/onnx/onnx/) format.

# Basic Components

Support for running custom ONNX models is provided by instantiating an `OnnxModelRunner` component with an ONNX model file, and providing a specification of the model inputs and outputs. This component is based on functionality provided by [ML.NET](https://docs.microsoft.com/en-us/dotnet/machine-learning/) and the [ONNX Runtime](https://microsoft.github.io/onnxruntime/), and is available in either the `Microsoft.Psi.Onnx.Cpu` or `Microsoft.Psi.Onnx.Gpu` packages, depending on whether or not GPU (CUDA) execution is required.

# Common Patterns of Usage

The following example shows how to run inference on a stream by running a custom ONNX model. This assumes that you have pre-trained a model and exported it to an ONNX model file (opset version 7 or greater).

```csharp
using (var pipeline = Pipeline.Create())
{
    var modelRunner = new OnnxModelRunner(
        pipeline,
        new OnnxModelConfiguration()
        {
            ModelFileName = "model.onnx",
            InputVectorSize = 25,
            InputVectorName = "data",
            OutputVectorName = "score",
        });
    ...
```

Here we instantiate a model runner with an `OnnxModelConfiguration` object containing information about the model. In this example, we assume the pre-trained ONNX model file named `model.onnx` has and input vector named `data` with size 25, and an output vector named `score`. It is assumed that the application will know how to interpret the results represented by the output vector. In the current implementation of `OnnxModelRunner`, input and output vectors are represented as single-dimensional arrays of type `float[]`.

Assuming we have a stream `inputStream` representing the raw input data, we can simply pipe this to the `modelRunner` and run it to get the output predictions:

```csharp
    ...
    IProducer<float[]> inputStream = ...; // assumes an input stream of raw vector data

    ...

    // assumes an output vector of size 1
    var predictions = inputStream.PipeTo(modelRunner);
    predictions.Do(x => Console.WriteLine($"Output: {x[0]}"));

    pipeline.RunAsync();
    Console.ReadKey();
}
```

# Additional Components

The `OnnxModelRunner` provides a generalized component for running any ONNX model on a stream of raw vector inputs.

Additional model runner components for specific pre-trained models are provided in a separate package (`Microsoft.Psi.Onnx.ModelRunners.Cpu` or `Microsoft.Psi.Onnx.ModelRunners.Gpu`). Currently only the [Tiny YOLOv2](https://github.com/onnx/models/tree/master/vision/object_detection_segmentation/tiny-yolov2) model runner has been implemented. More model runners will be added in the future.

# GPU Execution

The `Microsoft.Psi.Onnx.Cpu` and `Microsoft.Psi.Onnx.Gpu` projects both share common code, and are intended to be interchangeable without requiring any modification to the application code that references them. Similarly for the `Microsoft.Psi.Onnx.ModelRunners.Cpu` and `Microsoft.Psi.Onnx.ModelRunners.Gpu` projects.

In order to enable GPU execution, simply replace the reference to the `Microsoft.Psi.Onnx.Cpu` project/package with a reference to `Microsoft.Psi.Onnx.Gpu`. Then, set the `GpuDeviceId` property in the `OnnxModelConfiguration` object used to instantiate the `OnnxModelRunner` component to a valid, non-negative integer. Typical device ID values are 0 or 1. Set the `GpuDeviceId` property to `null` to revert to CPU execution.

## System Requirements for GPU Execution

The GPU version requires a [CUDA supported GPU](https://developer.nvidia.com/cuda-gpus#compute), the [CUDA 10.1 Toolkit](https://developer.nvidia.com/cuda-downloads) and [cuDNN 7.6.5](https://developer.nvidia.com/cudnn) (as indicated in the [ONNX Runtime documentation](https://github.com/Microsoft/onnxruntime#system-requirements)).

# ONNX Compatibility

Note that the [ONNX Runtime](https://microsoft.github.io/onnxruntime/) is [backwards compatible](https://github.com/microsoft/onnxruntime/blob/master/docs/Versioning.md#backwards-compatibility) to ONNX opset version 7, so models will first need to be [converted](https://github.com/onnx/onnx/blob/master/docs/VersionConverter.md) if they were exported to an older opset version.

