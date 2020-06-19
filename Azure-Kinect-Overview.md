Platform for Situated Intelligence supports using a Microsoft Azure Kinect Camera (see also [Microsoft Kinect V2](Kinect-Overview) support). This includes capture of video, infrared, depth, audio, and body tracking streams from the Azure Kinect. A [sample application available here](https://github.com/microsoft/psi/tree/master/Samples/AzureKinectSample) demonstrates its use.

**Please note**: <br/>Supported platforms for Azure Kinect currently includes 64-bit Windows 10 and Ubuntu 18.04 only ([vote for Mac support here](https://feedback.azure.com/forums/920053-azure-kinect-dk/suggestions/38061958-please-allow-for-macos-support-should-be-minima)).

# Basic Components

Basic Azure Kinect capabilities are provided by instantiating an `AzureKinectSensor` component. Support for Azure Kinect body tracking is provided by instantiating a `AzureKinectBodyTracker` component. These two components are available in the `Microsoft.Psi.AzureKinect` namespace / nuget package.

# Common Patterns of Usage

Below we present some examples of how to use the Azure Kinect sensor and body tracking in \\psi.

## Capturing Video

The following example shows how to create an `AzureKinectSensor` and stream images from the Kinect's color camera.

```csharp
using (var pipeline = Pipeline.Create())
{
    var azureKinectSensor = new AzureKinectSensor(
        pipeline,
        new AzureKinectSensorConfiguration()
        {
            OutputColor = true,
            CameraFPS = FPS.FPS30,
            ColorResolution = ColorResolution.R1080p,
        });

    azureKinectSensor.ColorImage.Do(img =>
    {
        // Do something with the image
    });

    pipeline.RunAsync();
}
```

Similarly, the `DepthImage` and `InfraredImage` streams are available when the `OutputDepth` and `OutputInfrared` flags are set to `true` in the configuration.

The following are available as part of the `AzureKinectSensorConfiguration`:

- **DeviceIndex:** The index of the device to open (default 0). 
- **ColorResolution:** The resolution of the color camera (default 1080p).
- **DepthMode:** The depth camera mode (default NFOV unbinned).
- **CameraFPS:** The desired frame rate (default 30 FPS).
- **SynchronizedImagesOnly:** Whether color and depth captures should be strictly synchronized (default `true`).
- **OutputColor:** Whether the color stream is emitted (default `true`).
- **OutputDepth:** Whether the depth stream is emitted (default `true`).
- **OutputInfrared:** Whether the infrared stream is emitted (default `true`).
- **OutputImu:** Whether to use the Azure Kinect's IMU (default `false`).
- **OutputCalibration:** Whether the Azure Kinect outputs its calibration settings (default `true`).
- **BodyTrackerConfiguration:** The body tracker configuration (default null, no body tracking).
- **DeviceCaptureTimeout:** The timeout used for device capture (default 1 minute).
- **FrameRateReportingFrequency:** The frequency at which frame rate is reported on the `FrameRate` emitter (default 2 seconds).

## Capturing the IMU

The following example shows how to create an `AzureKinectSensor` component and process the stream of IMU data from the device.

```csharp
using (var pipeline = Pipeline.Create())
{
    var azureKinectSensor = new AzureKinectSensor(
        pipeline,
        new AzureKinectSensorConfiguration()
        {
            OutputImu = true,
        });

    azureKinectSensor.Imu.Do(imu =>
    {
        void Output(string name, System.Numerics.Vector3 sample)
        {
            Console.WriteLine($"{name}: x={sample.X} y={sample.Y} z={sample.Z}");
        }

        Output("Accelerometer", imu.AccelerometerSample);
        Output("Gyro", imu.GyroSample);
    });

    pipeline.RunAsync();
}
```

## Capturing Temperature

The following example shows how to create an `AzureKinectSensor` and process the stream indicating the internal temperature of the device.

```csharp
using (var pipeline = Pipeline.Create())
{
    var azureKinectSensor = new AzureKinectSensor(pipeline, new AzureKinectSensorConfiguration());
    azureKinectSensor.Temperature.Do(temp => Console.WriteLine($"Temperature: {temp} °C"));
    pipeline.RunAsync();
}
```

## Capturing Audio

The Azure Kinect has a microphone array. The audio streams can be captured directly by using the `AudioCapture` component from the `Microsoft.Psi.Audio` namespace. See the [Audio Overview here](Audio-Overview).

```csharp
var audio = new AudioCapture(pipeline, new AudioCaptureConfiguration()
{
    DeviceName = "Microphone Array (Azure Kinect Microphone Array)", // your device name may differ
});

audio.Do(buffer =>
{
    // Do something with the audio buffer
});
```

## Body Tracking

This final section demonstrates how to use the Azure Kinect components to perform body tracking.

The `AzureKinectSensor` component uses depth and infrared information to produce a stream of bodies.

```csharp
var azureKinect = new AzureKinect(
    pipeline,
    new AzureKinectSensorConfiguration()
    {
        BodyTrackerConfiguration =
            new AzureKinectBodyTrackerConfiguration()
            {
                CpuOnlyMode = true,    // set to false if a CUDA supported GPU is available
                TemporalSmoothing = 0.5f,
            },
    });

azureKinect.Bodies.Do(bodies =>
{
    Console.WriteLine($"Bodies: {bodies.Count}");
});
```

The following are available as part of the `AzureKinectBodyTrackerConfiguration`:

- **TemporalSmoothing:** The temporal smoothing to use across frames for the body tracker. Set between 0 for no smoothing and 1 for full smoothing (default 0.5 seconds).
- **CpuOnlyMode:** Whether to perform body tracking computation only on the CPU. If false, the tracker requires CUDA hardware and drivers (default `false`).
- **SensorOrientation:** The sensor orientation used by body tracking (default upright).

## Decoupled Body Tracking Component

Body tracking functionality is also _independently_ implemented by the `AzureKinectBodyTracker` component.

```csharp
var bodyTracker = new AzureKinectBodyTracker(
    pipeline,
    new AzureKinectBodyTrackerConfiguration()
    {
        CpuOnlyMode = true, // false if CUDA supported GPU available
    });
```

This component consumes `DepthImage` and `InfraredImage` streams as well as a the 'AzureKinectSensorCalibration` stream that contains the sensor calibration information; these streams are produced by the `AzureKinectSensor` component, and can be persisted and leveraged for running the tracker at a later time.

For instace, assuming these streams were persisted into a store, we can open them up as follows: 

```csharp
var store = Store.Open(pipeline, "MyRecording", @"C:\Data");
var depth = store.OpenStream<Shared<DepthImage>>("DepthImage"); // DepthImage
var infrared = store.OpenStream<Shared<Image>>("InfraredStream"); // ColorImage
var calibration = store.OpenStream<Calibration>("AzureKinectSensorCalibration"); // AzureKinectSensorCalibration
```

The depth and infrared streams are joined and piped to the body tracker. The calibration stream is also separately piped to the body tracker. The tracker generates the resulting bodies on it's `Bodies` output stream.

```csharp
depth.Join(infrared).PipeTo(bodyTracker);
calibration.PipeTo(bodyTracker.AzureKinectSensorCalibration);

var bodies = bodyTracker.Bodies;
```