# __2020/08/31: Beta-release, version 0.13.38.2__

### __Overview__:

The release brings several important updates to Platform for Situated Intelligence Studio (PsiStudio), including support for temporal data annotations, additional visualizers for cartesian charts and histograms, general updates for various UI elements, as well as bug fixes and other improvements.

At the runtime and infrastructure level, this release adds support for writing custom store readers (via the `IStreamReader` interface) that can provide streaming data to \\psi pipelines as well as in PsiStudio from data sources stored in third-party formats. As an example, we have written a `WaveFileStreamReader` that allows reading data and visualizing data from WAVE files.

Finally, the release also introduces new \\psi components that enable running Onnx machine learning models.

The full, more detailed list of changes is described below:

### __Changes to Visualization and PsiStudio__

* Added support for temporal annotations. Users can now create streams of multi-track annotations based on custom annotation schemas.
* Added support for visualizing a property or field of a stream. Right-clicking on a stream and selecting “Expand members” will display these sub-fields and they can be visualized as well. 
* Added initial support for dynamically loading batch processing tasks from 3rd party assemblies. The tasks are identified by the `[BatchProcessingTask]` attribute, and surfaced in the context menus for Dataset and Sessions. Currently only tasks that have a signature taking a (`Pipeline`, `SessionImporter`, `Exporter`) are surfaced. 
* Added support for opening custom, third-party stores based on the available stream readers (e.g. choose `.wav` in file dialog).
* Temporal selection region is now highlighted.
* Added new visualizers for camera intrinsics (visualizes a camera frustum), depth image point cloud, depth image mesh, and image with intrinsics.
* Added new visualizers for cartesian charts and histograms using LiveCharts.
* `CoordinateSystem` visualizer now supports size and diameter as separate properties.
* `KinectBody` and `AzureKinectBody` visualizers now support individual bone thickness and joint radius.
* Panels now have a `BackgroundColor` property.
* 2D and 3D panels now have a `RelativeWidth` property that allows for variable width instant panels in a container.
* The layout selection combobox and associated buttons have been moved from the main toolbar to the Visualizations tab. Multiple minor bug fixes around switching layouts.
* Several engineering, refactoring, and code clean-up updates, including refactoring visualization for depth images, restructuring context-menu generation, rationalizing names for the `VisualizationObject` attribute, increased readability of type names the property browser.
* Several additional bug fixes in `DepthExtensions.ProjectTo3D`, `IntersectLineWithDepthMesh`, `Point3D` visualizer, message visualization object.
* BREAKING CHANGE: The `Microsoft.Psi.Visualization.Common` namespace has been renamed `Microsoft.Psi.Visualization` and the project Microsoft.Psi.Visualization.Common.Windows has been renamed Microsoft.Psi.Visualization.Windows.


### __Support for Running ONNX Models__

* Added a new set of packages, Microsoft.Psi.Onnx.Gpu and Microsoft.Psi.Onnx.Cpu containing a component called `OnnxModelRunner` that enables running a specified Onnx model.
* Added a new set of packages, Microsoft.Psi.Onnx.ModelRunners.Gpu and Microsoft.Psi.Onnx.ModelRunners.Cpu, containing wrapper components that enable more easily running specific Onnx models. As a first example, `TinyYoloV2OnnxModelRunner` enables running object detection over images with a TinyYoloV2 model. We plan to extend the set of model runners in the future.

### __Custom Store Readers__

* Added `IStreamReader` and `IStreamMetadata` interfaces to support reading streams from custom storage formats.
* `PsiStoreReader` is an implementation of `IStreamReader` for \psi stores.
* `WaveFileStreamReader` is an implementation of `IStreamReader` over WAVE files.
* Added extension methods to add sessions and partitions specific to \psi stores and WAVE file stores.

BREAKING CHANGES:
* `Importer`, `Store`, `Dataset`, `Session` and `Partition` APIs have changed to accommodate arbitrary stream readers.
* APIs returning `PsiStreamMetadata` now return `IStreamMetadata`.
* Static APIs specific to \psi stores have moved from `Store` to `PsiStore` (`Store` APIs are obsolete).
* `Importer` constructor no longer takes name and path; instead it _requires_ an `IStreamReader`.
* `PsiStore.Create(...)` now returns `PsiExporter` (base `Exporter` is now abstract).

### __Other Changes to Runtime__

* Fields with the `[NonSerialized]` attribute are no longer skipped when cloning. To skip cloning such fields, register the containing type with the `SkipNonSerializedFields` cloning flag.
* Improved serialization compatibility for some .NET types across frameworks.
* Consolidated overloads for Pipeline.Create() factory method into a single method.

BREAKING CHANGES: 
* Renamed `PsiStreamMetadata` and `Importer` properties:
    * `Lifetime` -> `StreamTimeInterval`
	* `ActiveLifetime` -> `MessageCreationTimeInterval`
	* `OriginatingLifetime` -> `MessageOriginatingTimeInterval`
	* `FirstMessageTime` -> `FirstMessageCreationTime`
	* `LastMessageTime` -> `LastMessageCreationTime`
* Renamed `IndexEntry.Time` -> `IndexEntry.CreationTime`.
* Renamed `Envelope.Time` -> `Envelope.CreationTime`.
* Cloning objects containing unmanaged pointers and `IntPtr` fields (e.g. delegates) is no longer allowed by default. This behavior may be overridden by registering the containing type using the `KnownSerializers.Register` method with the cloning flags `ClonePointerFields` and `CloneIntPtrFields` respectively.


### __Changes to Kinect and AzureKinect__

* Added `Configuration` member to `AzureKinectSensor`.


### __Changes to Microsoft.Psi.Imaging__

* Checks on Unmanagedbuffer sizes to only copy as much data as is in the actual image (height*stride).
* Fixes to resize/scale to preserve sizes.
* BREAKING CHANGE: Changed `Rotate` and `DetermineRotateWidthHeight` to support loose/tight rotation.

### __Changes to Microsoft.Psi.Audio.Windows__

* BREAKING CHANGE: The `OutputFormat` parameter in `AudioCaptureConfiguration` has been renamed `Format`.
* BREAKING CHANGE: The `InputFormat` parameter in `AudioPlayerConfiguration` has been renamed `Format`.

### __Other__

* BREAKING CHANGE: Renamed `[Task]` attribute used to define tasks to `[BatchProcessingTask]`.
* BREAKING CHANGE: Removed `SimpleVoiceActivityDetector` component.
* Dataset processing APIs now support progress reporting.
* Fixed bug in `ImageAnalyzer` component where region was being ignored.
* Set auto CRLF handling for the whole repository.
* Updated symbol packages to the .snupkg format.


## __2020/06/16: Beta-release, version 0.12.53.2__

### __Overview__:

The major changes in this release include the addition of support for the Azure Kinect sensor (including body tracking), an overhaul of Microsoft.Psi.Imaging, some additional Kinect for Windows component cleanup, visualization updates, as well as an upgrade of projects to Visual Studio 2019 and .NET Core 3.1. Miscellaneous updates were also made to other areas (see more details below).

_A Note About Coordinate Systems_: This release begins to rationalize the handling of coordinate system bases throughout the code base. We adopt the MathNet.Spatial convention everywhere, in which the X-axis represents "forward", the Y-axis represents "left", and the Z-axis represents "up". Matrix representations assume column vectors and are stored in column-major order. These changes particularly affect code and data from Microsoft.Psi.Calibration as well as the Kinect and Azure Kinect sensor and body joint representations.

### __Azure Kinect Component (New)__

* Added `AzureKinectSensor` and `AzureKinectBodyTracker` components that enable using the Azure Kinect sensor with \psi (see more: [Overview](Azure-Kinect-Overview), [Sample](https://github.com/microsoft/psi/tree/master/Samples/AzureKinectSample), [Azure Kinect Documentation](https://azure.microsoft.com/en-us/services/kinect-dk/)).
* The Kinect and AzureKinect body visualizers have moved into their own projects and need to be loaded as [3rd party visualizers for PsiStudio](3rd-Party-Visualizers) to become available.

### __Updates to Microsoft.Psi.Imaging__

A number of updates were made to the imaging infrastructure in Microsoft.Psi.Imaging, including the addition of support for depth images.

BREAKING CHANGES:
* `EncodedImage` now requires `width`, `height` and `pixelFormat` to be specified on construction.
* `EncodedImagePool` is now keyed, and `GetOrCreate()` now requires the encoded image `width`, `height` and `pixelFormat`.
* Renamed `Image.FromManagedImage()` to `Image.FromBitmap()`.
* Renamed `Image.ToManagedImage()` to `Image.ToBitmap()`.
* Renamed `ImagePool.GetOrCreate(Bitmap)` to `ImagePool.GetOrCreateFromBitmap(Bitmap)`.
* `Image.EncodeFrom()` now takes in an `IImageToStreamEncoder` instead of an `Action<>`.
* Moved `ImageEncoder` and `ImageDecoder` \psi components from the platform-specific .Windows and .Linux projects into the cross-platform Microsoft.Psi.Imaging project. These components are now provided an `IImageToStreamEncoder` or `IImageFromStreamDecoder` object to do the actual encoding and decoding. A variety of implementations (e.g. for JPEG, PNG) are available in the corresponding platform-specific .Windows and .Linux projects.
* Removed the static method `ImageEncoder.EncodeFrom(encoder, ...)` and replaced with `encoder.EncodeFrom(...)`.
* Renamed `CompressionMethod.PNG` to `CompressionMethod.Png`.
* Renamed the `Convert()` stream operator for shared images to `ToPixelFormat()`, for clarity and to match the underlying component.
* The 3D related operators from `DepthExtensions` in Calibration now operate only on depth images.
* `KinectSensor.DepthImage` stream now returns a stream of `DepthImage` rather than a stream of `Image`.
* The `Window()` operator now correctly handles `TimeInterval` inclusivity rather than excluding messages _on_ the boundary. Additionally, the operator no longer accepts a `waitForCompleteWindow` parameter and never delays the initial window.

Other changes:

* Added new `DepthImage` and `EncodedDepthImage` types to represent depth images, and a set of associated operators.
* Overhauled the various image processing functions and corresponding stream operators to rationalize the API.
* Added support for preserving the pixel format when decoding an `EncodedImage`.
* Fixed a number of bugs, including issues related to pixel formats and decoding.

### __Changes to Visualization and PsiStudio__

* Kinect body visualization removed from Visualization.Common and put in its own project so that _Visualization.Common and PsiStudio no longer have a dependency on the Kinect SDK_.
* `KinectDepth` visualizer is now a more general `DepthImage` visualizer supporting both mesh and point cloud views.
* Updated handling of Shared objects in instant visualizers.
* PsiStudio now starts up with no default dataset and the main window titlebar now displays the name of the current Dataset and the name of the stream currently being snapped to (if any).
* Updated visualization object selection algorithm to select the visualizer the user most likely wants to use.
* A StreamVisualizationObject can now only be snapped to when it is bound to a source, and if it becomes unbound then we remove snap-to-stream.
* A TimelineVisualizationObject can now only be zoomed to when it is bound to a source.
* Multiple bug fixes to StreamBinding serialization, we now write out the assembly qualified name of StreamAdapterType, and we now correctly serialize child objects of 3D visualization object collections.
* Code cleanup of XyzVisualizationPanel and its view including adding bidirectional properties for camera position, direction and field-of-view.
* Added support for specifying which items in an UpdatableModelVisual3DVisualizationObjectDictionary are visible via a predicate.
* Added PsiStudio cursor nudging feature.
* The pipeline visualizer now shows a dotted edge connection between subpipelines when they contain a connection (perhaps even between deep descendants) spanning them.
* Fixed bug where PsiStudio settings were sometimes not being correctly written when shutting down PsiStudio.
* Fixed bug involving the index view in StreamReaders for InstantVisualizationObjects when the user changed the value of CursorEpsilon.

### __Changes to Runtime__

* Changes to improve backward compatibility with some older stores.
* Added `BridgeTo` operator to `Producer<T>` creating a connector as needed to bridge pipelines.
* Raise `PipelineRun` event for all subpipelines before starting.
* Change pipeline state to `Running` as soon as subpipelines have started.
* Fixed a pipeline shutdown race condition.
* Fixed `DeliveryPolicy<T>.WithGuarantees` to do a union between guaranteed message sets.

### __Changes to Stores__

* BREAKING CHANGE: Removed `Exporter.WriteEmitters`.
* Added `CropInPlace` store operation.
* Added support for supplemental metadata on streams.
* Fixed a bug in `Store.Copy` and `Store.Crop` which were using the `ActiveTimeInterval` instead of the `OriginatingTimeInterval`.

### __Changes to Calibration__

* Added "Pose" fields for `DepthDeviceCalibrationInfo` objects, which are the inverse of Extrinsics.
* Added ability to specify distort/undistort as either normal or reversed. Normal would be the closed form equation which returns distorted points and the reverse version returns undistorted points.

### __Changes to PsiStoreTool__

* Added support for showing stream size information in PsiStoreTool.
* Added verb for removing a stream from a store to PsiStoreTool.

### __Changes to Operators__

* Updated `Sample` operator with a new default, that allows it to sample the nearest message to the clock when no tolerance or relative-time-interval is specified.
* The `Zip` component and corresponding operators were changed to return `T[]`.

### __Changes to Other Components__

* BREAKING CHANGES to `KinectSensor` component:
    - Removed custom Matrix code in `KinectInternalCalibration` and re-implemented using MathNet.
    - Removed `IKinectSensor`, `KinectExtensions`, `DepthExtensions` and all GZip encoding functionality.
    - Removed `DepthToColorConverter` and re-implemented as a generic imaging operator.
    - Moved Orientation functions and LevenbergMarquardt optimization into `Microsoft.Psi.Calibration`.
    - Changed `KinectBody` definition to store Joint poses as MathNet CoordinateSystems, with immediate conversion into MathNet basis (X Forward, Y Left, Z Up).
* Added a constructor for `AudioCapture` component allowing an output format to be specified.
* Added ALSA error codes in exceptions thrown by the Linux `AudioCapture` and `AudioSource` components.
* Fixed Linux audio and video components to work on 32-bit OS.
* Converted `RealSenseSensor` component to use new `DepthImage`.
* Added a constructor for `MediaSource` component that accepts a `System.IO.Stream` as input.
* Fixed an issue with `Mpeg4Writer` where image data was being written in RGB instead of BGR.

### __Miscellaneous Engineering Updates__

* This release moves .NET Core projects to version 3.1. As a result, Visual Studio 2019 is now required to build the code. Building with Visual Studio 2017 will no longer be supported.
* Added code analysis to projects.
* The following NuGet packages were updated to the versions shown:
    - Extended.Wpf.Toolkit (3.6.0)
    - HelixToolkit.Wpf (2.11.0)
    - MathNet.Numerics.Signed (4.9.1)
    - MathNet.Spatial.Signed (0.6.0)
    - MessagePack (2.1.90)
    - Microsoft.NET.Test.Sdk (16.6.1)
    - MSTest.TestAdapter (2.1.1)
    - MSTest.TestFramework (2.1.1)
    - System.ServiceModel.Primitives (4.7.0)
* Various bug fixes, typographical corrections, and code cleanup.


## __2020/04/14: Beta-release, version 0.11.82.2__

### __OVERVIEW__:

This update brings several new developments, and numerous improvements and bug fixes in the \psi Runtime, Tools and Components.

- In __Runtime__ the major changes include the addition of new types of delivery policies, emitter validators, and streamlining of the APIs for running the pipelines (see more details below)
- In __Tools__ a number of significant updates have been brought to PsiStudio, including performance improvements, support for 3rd party visualizers, and matrix layouting, and other extended functionality. Similarly, the performance of the Diagnostics subsystem has been improved and visualization functionality for diagnostics data has been extended. A number of updates have been made also to PsiStoreTool.
- In __Stream Operators__ and __Components__ a number of extensions and bug fixes have been added.

### __Details of Changes to Runtime__

A number changes have been made to clean up and streamline the APIs for pipeline execution. The changes below are breaking changes:
* Removed the `Pipeline.Run(TimeSpan)` method as its meaning was inconsistent with running a pipeline with a finite originating time interval. To run a pipeline for a given wall-clock duration, call `RunAsync()`, followed by `WaitAll(TimeSpan)`, then dispose the pipeline to stop it.
* `ReplayDescriptors` no longer define the `UseOriginatingTime` property as it was redundant. We always assume originating times when reading messages within a given time interval from a store. Consequently, `Pipeline.ProposeReplayTime()` now takes only a single `TimeInterval` parameter representing the originating time interval.
* `Pipeline.WaitAny` has been removed. To be notified of source component completion, use the `Pipeline.ComponentCompleted` event instead.
* Pipeline replay at speeds other than real-time or maximum speed are no longer supported. Consequently, the `replaySpeedFactor` parameter and `ReplaySpeedFactor` property have been removed the `Pipeline.Run` methods and the `ReplayDescriptor` class respectively.
* The `deliveryPolicy` parameter for the Pipeline and Subpipeline constructors has been renamed to `defaultDeliveryPolicy`.

The available set of delivery policies has been extended. See more information in [Delivery Policies](Delivery-Policies) tutorial:
* Added support for throttling policies via DeliveryPolicy.Throttled.
* Added a new lossless policy that attempts synchronous delivery or throttles otherwise via DeliveryPolicy.SynchronousOrThrottle.
* Added support for data-driven delivery policies via typed delivery policies with guarantees.

A number of additional changes have been made to the Runtime:
* `Pipeline.RunAsync` now supports progress reporting by means of an `IProgress<double>` object.
* The pipeline start time is now captured in a new `Pipeline.StartTime` property.
* Subpipelines provide access to the parent pipeline via the protected `ParentPipeline` property.
* When persisting a stream to a store, the stream closing time is now saved in the store catalog.
* Emitters now support optional user-defined message validation.
* Fixed a bug where the presence of an infinite source component within a subpipeline was in some cases causing the pipeline to terminate.
* Added message validation for emitters, and support for pipeline level defaults.


### __Details of Changes to PsiStudio__

A number of significant updates have been brought to PsiStudio, including performance improvements, support for 3rd party visualizers, and matrix layouting, and other extended functionality:

Performance updates: 
* Improved performance when reading data from disk for instant visualization objects.  Previously these reads were occurring on the UI thread whereas now they happen in a worker thread. Now they are optimized so that the same piece of data is not read multiple times for different visualization objects.

New functionality and bug fixes:
* In order to make better use of available screen real estate, instant visualization objects (2D and 3D visualization objects) can now be arranged in a horizontal matrix in the visualization container. These matrices can contain from one to five instant visualization objects.
* PsiStudio can now visualize the Messages and Latencies for all streams, regardless of type.
* The states of the timing info buttons are now persisted across PsiStudio sessions.
* When the use clicks in a visualization panel, PsiStudio displays the properties of the first visualization object in that panel, or if the panel has no visualization objects it displays the properties of the panel itself.  Similarly, whenever the user adds a new visualization object to the visualization container its properties are  immediately displayed.
* All DateTime displays in PsiStudio now show the value down to 1/10,000 of a second.
* Fixed bug where certain line segments were not drawn if the value of any of the data points was NaN or positive infinity or negative infinity.
* Users can now play multiple audio streams simultaneously.
* Added snap to stream context menu item to the timeline panel.
* 3D visualization panel now supports camera manipulation and the execution of camera animation storyboards.

Support for 3rd party visualizers:
* Added support for 3rd party visualizers. Users who have written their own visualizers can now use them in PsiStudio in a much more seamless manner. Add a new node called `<AdditionalAssemblies>` to the PisStudio settings file at `MyDocuments/PsiStudio/PsiStudioSettings.xml`.  Any visualization object marked with the `[VisualizationObject]` attribute, and any stream adapter marked with the `[StreamAdapter]` attribute will be automatically added to PsiStudio’s list of visualizers.

Other updates:
* For historical reasons related to PsiStudio previously supporting a COM model,  Visualization Objects and Visualization Panels each had a separate `Configuration` class.  This configuration has now been merged back into the visualization objects or visualization panels, simplifying the code.
* Updated the 3D visualization objects to have a more hierarchial structure.
* Layout serialization now includes a version number to help prevent loading a layout that is not compatible with the current version of PsiStudio.


### __Details of Changes to Diagnostics__

The diagnostics system enables collecting and visualizing application diagnostics in PsiStudio. This release includes many performance improvements to the diagnostics system. The overhead of collecting diagnostics information has been improved. Message sizes are half what they were and rendering in PsiStudio has been improved by orders of magnitude. Many bug fixes have been made as well, mosting having to do with visualization in PsiStudio. There have been breaking changes to the diagnostics message type affecting visualization.

Visualization:
* To simplify the graph display, we now allow checking a box _Show Exporter Links_ to show/hide connectors and edges from them to Exporters.
* Connectors render with a little icon (☍) and `Joins` now render in their own color with a simple plus (+).
* Type names have been simplified from their fully qualified form to a more natural “code form” such as `(List<string>, Dictionary<char, (int[], double)>)`.
* Connector tooltip now shows target/source.
* Emitter names can now be optionally shown on edge labels.
* Delivery policies can now be optionally shown on edge labels and hilighting of edges by delivery policy is now supported.
* Property panel now allows toggling emitter/receiver names as well as delivery policies.
* A dotted line edge is now shown indicating a connector into a descendant pipeline. That is, an edge to the direct child containing the descendant.

Naming Improvements:
* `Subpipeline` and `Exporter` node label is now the pipeline name (with type name now as a tooltip)
* Components can now provide names via `ToString()` used to render nodes (rather than the default type name)

Miscellaneous:
* Averaging is performed by unit time rather than past n-messages and added dropped & processed averages.

### __Details of Changes to PsiStoreTool__

* Added `concat` verb to PsiStoreTool (as well as `Store.Concat(…)` static method) allowing concatenation of stores.
* Added `crop` verb to PsiStoreTool allowing cropping of stores (exposing `Store.Crop(…)` functionality).
* Added `Microsoft.Psi.TaskAttribute` to mark “task” methods as light-weight jobs that may be executed by PsiStoreTool (and future PsiStudio).
* Added `tasks` and `exec` verbs to `PsiStoreTool` allowing listing and executing tasks.


### __Details of Changes to Stream Operators__

A number of BREAKING CHANGES were made to stream operators:
* [Windowing operators](https://github.com/microsoft/psi/wiki/Windowing-Operators) now produce streams of `T[]` rather than `IEnumerable<T>`, because the latter are not serializable.
* The [mathematical and statistical operators](https://github.com/microsoft/psi/wiki/Basic-Stream-Operators#mathematical-and-statistical-operators) no longer apply to streams of `IEnumerable<T>`, but to streams of `T[]`.
* Removed the `Sequence<T>` operator overloads that take an `IEnumerator` argument. Use the overloads that takes an `IEnumerable` instead.
* Removed the optional `interval` and `alignmentDateTime` parameters from the `Once` and `Return` operators.

A number of other additional enhancements were made to other stream operators:
* Added support for default delivery policies and naming of the `Parallel` composite components and operators.
* Generator changes:
    - Streams produced by `Generators` are now aligned by default to the pipeline start time, unless explicit message times are supplied.
    - Fixed a bug in the time alignment computation which may in rare cases result in a missing first message when an `alignmentDateTime` is supplied.
    - Fixed a bug where some `Generators` were not obeying a `ReplayDescriptor` with a finite end time.
* Added a `DynamicWindow` component and corresponding (`Window(…)` overload) stream operators that enable data-driven windowing.
* Small extension (with optional parameter) to `RelativeTimeWindow` component and corresponding `Window()` stream operator, enabling them to post results immediately before seeing a full window. Default behavior is still to wait for the complete window.
* Added `Merge` operator, which trivially combines one or more streams of type `T` into a single stream of type `Message<T>`, each with generated at the pipeline time when the message arrives at `Merge`, but containing the original envelope.
* Added `Zip` operator, which operates like `Merge` but ensures delivery in originating time order (ordered within single tick by stream ID)
* Added `First(n)` operator, which filters a stream to the first _n_ messages.
* Added a `WriteEnvelopes` method for streams that enables writing only the envelopes from a stream (without the payload) to the store.
* `Importer.OpenStream<T>` now checks that the type of the underlying stream messages matches the supplied type argument `T` and throws an exception if they do not match.
* Performance improvements to the `Std` operator. 
* Minor changes in the way `Generator`, `Importer` and `Exporter` apply delivery policies.

### __Details of Changes to Components__

A number of changes were made in a variety of components:
* AudioCapture component
    - Component now uses WASAPI event-driven capture by default. The previous behavior of pulling data from the engine at the target latency is still available by setting the UseEventDrivenCapture configuration parameter to false.
    - The default audio capture buffer size may now be overridden by setting the `AudioEngineBuffer` configuration parameter. Larger values may help reduce the likelihood of audio glitches.
    - Component now throws an `IOException` if it is unable to create the audio capture device.
* AzureSpeechRecognizer component:
    - Final recognition results are now aligned in originating time to the input voice activity detection signal.
* SystemSpeechSynthesizer component:
    - Added `ReceiveSsml` and `CancelAll` receivers to support issuing SSML-based speech synthesis requests and stopping of speech synthesis.
* SystemVoiceActivityDetection component:
    - Added `InitialSilenceTimeoutMs`, `BabbleTimeoutMs`, `EndSilenceTimeoutMs`, and `EndSilenceTimeoutAmbiguousMs` parameters in the component configuration. More details about these parameters may be found in the [SpeechRecognitionEngine documentation](https://docs.microsoft.com/en-us/dotnet/api/system.speech.recognition.speechrecognitionengine?view=netframework-4.8#properties).
* Face Identification:
    - The `FaceRecognizer` component, which identifies faces in a stream of `Shared<Image>`, has been updated to the latest Azure Face APIs.
    - A `PersonGroup` class supports managing person groups in Azure (creating, adding faces, training, deletion).
    - Also included are `PsiStoreTool` tasks supporting creating, training and testing from a directory of labeled static image files.
* `Microsoft.Psi.Calibration` changes (these are Breaking Changes):
    - Added support for full rational distortion model (with k1-k6 parameters)
    - Calibration representations now uses a canonical coordinate system throughout, in alignment with MathNet (right-handed, X forward, Y Left, Z up, column-vectors). 
    - Moved `KinectCameraCalibration` class and corresponding interface into `Microsoft.Psi.Calibration` and renamed to `DepthDeviceCalibrationInfo`
    - `SystemCalibration` was renamed to `MultiCameraCalibration`
    - Moved `ProjectTo3D` class from `Microsoft.Psi.Kinect` to `Microsoft.Psi.Calibration`.
* Added `Microsoft.Psi.DeviceManagement` project which enables device enumeration. Updated `Microsoft.Psi.Kinect` project to allow for enumerating all Kinect devices.
* The `Microsoft.Psi.CognitiveServices.Vision.Windows` project is now .NET Standard and has been renamed `Microsoft.Psi.CognitiveServices.Vision`.

### __Miscellaneous__

* Updated projects in solution to new .csproj style.
* Updated and consolidated NuGet references to the latest versions.
* Fixed a bug that was causing two unit tests to fail occasionally.


## __2019/08/27: Beta-release, version 0.10.16.1__

### OVERVIEW:

The major changes in this release are updates and added functionality in the stream operators for [fusion and synchronization](Synchronization), operators for [interpolation and sampling](Interpolation-and-Sampling), and operators for [generating streams](Stream-Generators). The release also includes a number of other updates and bug fixes - see more details below.

### Details of Changes to Stream Operators

A number of changes have been made to the set of interpolators, and the synchronization operators (for the latest documentation on synchronization and stream fusion and merging see [here](Stream-Fusion-and-Merging)):

* The set of interpolators has been restructured, generalized and expanded. Interpolators can now produce a result that has a different type than the input message type, and allow for matching with the `Nearest`, `First` or `Last` message. A taxonomy of greedy versus reproducible interpolators has been defined.
* Added support for an adjacent-values-interpolator, and constructed a linear interpolator based on it.
* The existing `Join` operator that allows for reproducible stream fusion and correct synchronization now operates with reproducible interpolators. 
* A `Fuse` operator has been added to enable stream fusion with any generic interpolator (reproducible or greedy). 
* Interpolators now accept a user-specified default value to use.

A number of other operators for sampling, generating streams, and statistical computations have also been updated: 

* Redesign of `Sample` and `Interpolate` stream operators. Both operators are driven by a clock stream. The `Sample` stream operator selects the nearest message to the sampling point (within a tolerance or relative time window). The `Interpolate` operator uses a specified interpolator to generate the result. The documentation for sampling and interpolation is available [here](Interpolation-and-Sampling).
* The `Generators.*` methods have been updated to allow the developer to specify whether the generated streams should stay open or close after the enumeration terminates. The current documentation on stream generators is available [here](Stream-Generators).
* Refactoring and clean-up of [mathematical and statistical operators](https://github.com/microsoft/psi/wiki/Basic-Stream-Operators#mathematical-and-statistical-operators).

### Details of Updates to Pipeline Diagnostics

* Added extension methods to provide access to graph-wide statistics such as total queued messages, total dropped, etc. across the whole graph.
* Enabing diagnostics at pipeline creation time (e.g. `Pipeline.Create(true, ...)`) used to take an optional `TimeSpan` interval. Now it takes an optional `DiagnosticsConfiguration`; containing this and other settings.
* New `DiagnosticsConfiguration.TrackMessageSize` flag determines whether message byte counts are tracked (`false` by default, as it is an expensive operation).

### Details of Changes to Platform for Situated Intelligence Studio:

* Minor icon updates.
* Ensure that dialog boxes appear above the PsiStudio main window.

### Other Updates

* Moved `EncodedImage` and `EncodedImagePool` out of `Microsoft.Psi.Imaging.Windows` and `.Linux` into the central .NET Standard shared `Microsoft.Psi.Imaging`. This is to enable referencing a single `EncodedImage` type across platforms.
* Interop: Updated CSV format docs for new behavior.
* Added sensor_msgs/Image message type to the ROS bridge.
* Added more informative exception message when posting messages on a stream with out-of-order sequence IDs.

### Bug Fixes:

* Fixed bug in pipeline diagnostics capture regarding counts of messages dropped.
* Fixed bug in queue-size-constrained delivery policies (including `DeliveryPolicy.Latest`) which sometimes led to a queue size larger by one than the max, and allowed additional messages to be processed.

### **Breaking Changes:**
* As a result of the re-organization of interpolators, the `Match` static class and the interpolators it contained have been renamed. Specifically: `Match.Best` -> `Reproducible.Nearest`, `Match.Exact` -> `Reproducible.Exact`; similar for the OrDefault versions.
* `Interpolator<T>` was replaced with `Interpolator<TIn, TOut>` to support interpolation to a different output type.
* `Generators.Sequence`, `Generators.Repeat` and `Generators.Range` require specifying an explicit time interval (previously the time interval parameter was optional and defaulted to 1 tick).
* The existing overloads of the `Sample` stream operator that used an interpolator were renamed to `Interpolate`.
* The OpenCV Sample (see Public\Samples\OpenCVSample) now requires OpenCV 4.1.1 to compile. You must also set the environment variable OpenCVDir_V4 to point to your local installation of OpenCV 4.1.1.
* Enabing diagnostics (e.g. `Pipeline.Create(true, ...)`) now takes an optional `DiagnosticsConfiguration` rather than a `TimeSpan` interval.

## __2019/07/18: Beta-release, version 0.9.6.1__

### OVERVIEW:

The main highlights in this release are:

* A major refresh of the `Platform for Situated Intelligence Studio` user interface, including a new color scheme, updated icons, and new functionality.

### Details of Changes to Documentation:

* The installation documentation has been updated to detail how to build the \psi codebase on Windows (using either Visual Studio 2017 or 2019), or on Linux.

### Details of Changes to Platform for Situated Intelligence Studio:

* Updated the look and feel of the user interface, including new colors and icons.
* The `Datasets` treeview and the `Visualizations` treeview now feature `Expand All` and `Collapse All` buttons to fully expand and collapse the tree.
* The `Visualizations` treeview now contains a new button to synchronize between it and the `Datasets` treeview.  When a stream in the `Visualizations` treeview is selected, clicking this button will select the source stream in the `Datasets` treeveiw and bring it into view.  This feature is useful for those times when you forget exactly which source stream a visualization is depicting.
* When in `Live` cursor mode, the timing information for the selection start and end markers is now not displayed as this information is irrelevent when in live mode.
* Added new context menu item `Zoom to Stream Extents` to streams in the `Visualizations` tree view.
* Added new context menu item `Zoom to Panel Extents` to panels in the `Visualizations` tree view.

### Bug Fixes:

* Fixed a bug in the `Join` operator that made it not correctly take into account open relative time intervals. See [this issue](https://github.com/microsoft/psi/issues/27).
* Fixed bug in the `Diagnostics` visualizer where connectors between subpipelines were being incorrectly rendered.
* Fixed bug in the `Diagnostics` visualizer that would cause it to crash if it encountered messages with unserializable fields.  We now return 0 for the size of such messages.
* Fixed bug in `Rodrigues` method in the `Orientation` class of `Microsoft.Psi.Kinect.Windows`.  This method was returning an NaN rotation matrix when passed a zero rotation vector, it now returns an identity matrix instead.

## __2019/07/03: Beta-release, version 0.8.32.1__

**IMPORTANT NOTE:**

In this release we have retargeted all .NET Framework projects to .NET Framework 4.7.2, and all native projects to Windows 10 SDK version 10.0.18362.0. In order to build the source code, please ensure that the following additional components are added to your Visual Studio installation:

* [.NET Framework 4.7.2 targeting pack](https://dotnet.microsoft.com/download/dotnet-framework/net472)
* [Windows 10 SDK (10.0.18362.0)](https://developer.microsoft.com/en-US/windows/downloads/windows-10-sdk)

If you are using Visual Studio 2019, you may install these components using the Visual Studio Installer, or via the download links provided above. For Visual Studio 2017, you may use the Visual Studio Installer to install the .NET Framework 4.7.2 targeting pack, but you will need to download and install the Windows 10 SDK (10.0.18362.0) using the provided link.

### OVERVIEW:

Some of the main highlights in this release include:

* Support for pipeline structure visualization, including message flow and diagnostics information (e.g. latencies, throughputs, stream statistics, etc.).
* Retargeted projects to .NET Framework 4.7.2 and Windows 10 SDK 10.0.18362.0.
* Updated NuGet package references in many projects to more recent versions.
* Runtime API changes and bug fixes.
* Moved documentation to wiki pages.
* Created [gitter](https://gitter.im/Microsoft/psi) channel.

### Breaking Changes:

* The `ISourceComponent.Stop` method now takes a `finalOriginatingTime` argument representing the pipeline shutdown time and a `notifyCompleted` delegate which the component must now call to notify the pipeline that it has completed generating source messages up to the `finalOriginatingTime`. For more details, see [here](https://github.com/microsoft/psi/wiki/Writing-Components#6-source-component-interface).
* Pipeline exception handling is now exposed through new `PipelineException` event rather than overloading the `PipelineCompleted` event to also serve as an exception handler.
* Input and output connectors for subpipelines should now be constructed using the new `CreateInputConnectorFrom` and `CreateOutputConnectorTo` static methods of the `Subpipeline` class.
* The `Match.Any` interpolator has been removed.
* Enabing diagnostics at pipeline creation time (e.g. `Pipeline.Create(true, ...)`) used to take an optional `TimeSpan` interval. Now it takes an optional `DiagnosticsConfiguration`; containing this and other settings.

### Details of Changes to the Runtime:

Several significant changes were made to the pipeline startup and shutdown logic:

* There is now a single scheduler for the main pipeline and all subpipelines. To facilitate management of work items in subpipelines, a new `SchedulerContext` has been introduced to which work items may be assigned.
* Message delivery in a pipeline is now delayed at startup until all source components in the pipeline have been activated.
* To enable correct and reproducibile pipeline shutdown, changes were made to the `ISourceComponent` that will need to be implemented by stream sources. For more details, see [here](https://github.com/microsoft/psi/wiki/Writing-Components#6-source-component-interface).
* Renamed the `PipelineElement` `Start` and `Stop` methods to `Activate` and `Deactivate` respectively.
* Added a check in `Receiver.OnSubscribe` to ensure that emitters and receivers are not connected across pipelines (connectors should be used instead).
* Changes to the pipeline exception handling mechanism via a new `PipelineException` event.
* New `DiagnosticsConfiguration.TrackMessageSize` flag determines whether message byte counts are tracked (`false` by default, as it is an expensive operation).

### Details of Changes to Operators and Components:

* Updated `ParallelSparse` and `Exporter` to use the composite component pattern, i.e. they are now implemented as a single subpipeline component.
* Added `MessageConnector` and `IConnector` interface to support connectors that wrap the entire message as they pass it along (used by `Exporter`).
* Added new static methods `CreateInputConnectorFrom` and `CreateOutputConnectorTo` on `Subpipeline` for creating connectors between pipelines.
* Changed the `StreamEnumerable` component (used by the `ToObservable` operator) to listen to the `Unsubscribed` event of its source rather than the `PipelineCompleted` event.

### Details of Changes to Platform for Situated Intelligence Studio:

* New pipeline diagnostics visualization feature enables visualization of the pipeline structure and message flow statistics in PsiStudio.
* Added a new toolbar button that lets the user toggle whether the navigator's cursor follows the mouse cursor when in manual cursor mode. This setting can be toggled via the button and also via the shortcut "Alt+F" (enabled by default).
* When playing back in repeat/loop mode, we now ensure the cursor remains visible after looping from the end of the stream back to the beginning.
* Added support for visualizing streams of strings.
* Generic message visualizer now shows string representation of message in live legend.
* Added support for formatting in legends.
* Moved rectangle and coordinate system visualizers out of PsiStudio and into Visualization.Common.
* Created interface IView3D as well as a base version for handling collections of 3D visuals.
* Created a single base 3D visualization object, removing a lot of duplicate code.

### Other Bug Fixes:

* Fixed an issue which caused the pipeline to appear to stop responding if an exception is thrown in a subpipeline's attached `PipelineRun` event handler.
* Fixed a bug in PsiStudio where the cursor remains a hand after dragging within a timeline visualization panel.
* Fixes to various visualizers (cleaned-up pattern for InitNew and handling config and property changes).
* Fixed an issue where live stores sometimes had no last message time.
* Fixed an AccessViolation related to `IAudioEndpointVolumeCallback` in the Windows audio components.
* Fixed a bug in one of the tuple-flattening `Join` operator overloads which was causing it to use the incorrect interpolator.
* Fixed a thread safety issue in the `Join` operator.
* Fixed a thread cleanup issue on disposal in `InfiniteFileWriter`.

---

## __2019/04/05: Beta-release, version 0.7.57.2__

### OVERVIEW:

There are many additions and updates in this release, but the major changes can be summarized as:

* Improvements and fixes to the pipeline shutdown procedure, as well as fixes to the `Parallel` and `Join` operators to support reproducible dynamic sub-pipeline construction and teardown via the `Parallel` operator.
* Streamlining the use of `Shared<T>` for more efficient messaging of large objects such as images, and added [in-depth documentation](Shared-Objects) on this topic.
* PsiStudio now supports connecting to live stores and fast layout switching.

### Breaking Changes:

* The pipeline startup and shutdown procedure has been updated to support correct and reproducible shutdown where possible in dynamic pipelines. Specifically:
    * Removed the `RegisterPipelineStart/Stop/Final` handlers. If the component implements `ISourceComponent`, then use the `Start` or `Stop` method, or use one of the following events instead:  `PipelineRun`, `PipelineCompleted`, `Receiver.Unsubscribed`.
    * The authoring of source components has been streamlined, and in the process the `IFiniteSourceComponent` and `ISourceComponent` interfaces have been unified into a single, new `ISourceComponent` interface. Implementers should call the `notifyCompletionTime` delegate supplied in the interface's `Start` method to indicate its completion time (or `DateTime.MaxValue` if it is infinite).
* Removed `GroupBy` operator.
* `MessagePack` no longer uses LZ4 compression during serialization, and now emits `DateTime` objects as Ticks rather than string representations.

### Updates to Pipeline Shutdown Logic:

There have been multiple changes made to the pipeline finalization code to support an orderly shutdown process.

* Subpipeline now has independent scheduler so that it may fully shut down independently of the main pipeline.
* Renamed several `Pipeline` events:
    * `PipelineCompletedEvent` event has been renamed `PipelineCompleted`.
    * `ComponentCompletedEvent` event has been renamed `ComponentCompleted`.
    * `PipelineCompletionEventArgs` class has been renamed `PipelineCompletedEventArgs`.
    * A new event `PipelineRun` has been added, it is raised when `Pipeline.Run` (or `Pipeline.RunAsync`) has been called but before the components have started work.
* `Receiver` component now provides an `Unsubscribed` event.
* `Scheduler` now drops messages that were posted after the pipeline has shut down.
* `Scheduler.Schedule` method now returns an indication of whether the call was ignored due to the pipeline having already shut down.
* Bug fix in `ParallelSparse` so it works correctly with various branch termination policies.
* Pipeline pause for quiescence now includes all child subpipeline schedulers.
* Fixed bug in the `ToEnumerable` operator where empty streams were not being handled correctly.

### Updates to Parallel Operator:

* Default branch termination policy now returns the OriginatingTime of the last message from that branch.
* `Parallel` operators now use a `Connector` to bridge to Subpipelines.
* Added a fix for Parallel with orDefault, which was not working properly in some cases due to errors in the matching function.

### Updates to Join and Match Operators:

* Fixed bug in `Join` operator relating to the dynamic closing of secondary streams.
* Fixed bug in `Match` component's `NearestMatch` function where if there were no available messages we would always return `InsufficientData`. We now first check if the stream is closed, and in this case we return `DoesNotExist` instead.

### New Features in Shared Pool APIs:

* The `Shared<T>.Create` method no longer supports specifying a SharedPool<T> to which shared objects may be recycled. Recyclable shared objects must now be created from a shared pool.
* The `SharedPool<T>.GetOrCreate` method no longer supports specifying a construction delegate, it must now be provided when the `SharedPool<T>` is created.

### New Features in Audio and Speech:

* Added grammar-based intent detectors using `Microsoft.Speech` and `System.Speech`
* We now ensure that messages have monotonically increasing OriginatingTimes in the `MicrosoftSpeechRecognizer` component to comply with the new rule on the `Emitter` component introduced in the last release.
* Added support for `AzureSpeechServices` and marked as deprecated the obsolete component `BingSpeechRecognizer`.
* The Speech event duration is now defined as the OriginatingTime of the last message where the VAD was still false (before it switched to true) to the last message where the VAD output was still true.  Previously we were defining the event duration as the OriginatingTime of the first message where the VAD switched to true to the OriginatingTime of the first message where it went false again.
* Added `Reframe` operator to `Audio`.

### New Psi Components:

* Added new Psi component `PersonalityChat` to the Microsoft.Psi.CognitiveServices.Languages.Windows project that wraps [Project Personality Chat](https://labs.cognitive.microsoft.com/en-us/project-personality-chat).

### New Features in Platform for Situated Intelligence Studio:

* Removed all of the COM features that allowed a Psi application to launch and control PsiStudio.  See next point.
* PsiStudio can now connect directly to the store of a running Psi application in the same way it connects to a previously created store.  Users can seamlessly switch between Live mode and Playback mode when visualizing a live store.
* Simplified the hierarchy of `VisualizationObject` classes.
* Added option to show tracking id in the `kinect body` visualizer.
* When multiple audio streams are being visualized, users can now select which audio source should be played through the user's PC speakers by right-clicking the relevent stream in the `Visualizations` view.
* Added new `Repeat` button to the toolbar so that playback will loop infinitely between the `Selection Start` and `Selection End` markers.
* The position and layout of PsiStudio is now preserved when the application is shut down and is restored the next time PsiStudio is launched.
* Added `Layout Chooser` to the toolbar.  This allows the user to quickly switch between different Visualization layouts.  Users can create new layouts either from scratch or based on an existing layout and then save this new layout.  Layouts are stored in `/MyDocuments/PsiStudio/Layouts/`.
* Loading and then switching between multiple sessions is now correctly supported.  When a Visualization layout is selected, switching to a different session will automatically cause all Visualizers to rebind to the stores associated with this newly selected session.
* Added new icons to indicate when a `Visualization` has no stream to bind to in the current session.

### Other Bug Fixes:

* Fixed bug where the final extent of the store on disk was not being correctly truncated to the actual stream size after pipeline shutdown.
* Fixed bug in PsiStudio where zooming out a long way would cause the app to apparently freeze.
* Fixed bug where `StoreWriters` and `StoreReaders` were generating different names for the same global mutex that indicated a store was currently live and being written to.
* Fixed bug where replay time intervals in parent pipelines did not take into account the replay time intervals in subpipelines.
* Fixed memory leaks in `Microsoft.Psi.Imaging.Operators` class.
* Improved performance of the serialization classes.
* Reduced startup delays in subpipelines.

---

## __2018/11/30: Beta-release, version 0.6.48.2__

### Breaking Changes:

* It is now a requirement that messages posted on an `Emitter` have strictly increasing originating times. Attempting to post multiple messages with the same originating time on the same stream will cause an exception to be thrown.
* The `Buffer`, `History` and `Window` operators have been unified as a single set of `Window` operators which take either an index-based or a relative time-based interval. The index-based variants emit the initial buffer only after the total count of messages within the specified index interval have been accumulated, whereas the time-based variants emit the initial buffer as soon as messages within the specified relative time interval are available.
* The following operators have been removed:
    * `SelectMany`.
    * `Mirror`.
    * `Repeat` - use `Pair` instead.
    * `Buffer` - use `Window` instead.
    * `History` - use `Window` instead.
    * `Previous`
* [Delivery Policies](Delivery-Policies) have been simplified and renamed:
    * The `Throttled` policy has been removed. It will be re-introduced in a later release once issues around throttling have been resolved.
    * The `Default`, `Immediate` and `ImmediateOrThrottle16` policies have been removed.
    * The `QueueSize` property has been renamed `InitialQueueSize`.
    * The `MaximumLag` property has been renamed `MaximumLatency`.
    * The `IsSynchronous` property has been renamed `AttemptSynchronous`.
* As a result of the above changes, many stream operators have been amended to take an optional `DeliveryPolicy` parameter.
* The constructors for the `Connector` component and associated `CreateConnector` extension methods no longer take an `owner` parameter.
* The serialization format for `SystemCalibration` has changed:
    * The `ImageWidth` and `ImageHeight` properties have been removed.
    * The `NumberOfFrames` property has been moved to the `CameraCalibration` class.
* The `ICameraIntrinsics` interface has changed:
    * The signature of the `DistortPoint` method has changed.


New Features in Platform for Situated Intelligence Studio:

* Updated the layout of the main Psi Studio screen. The Datasets tree view and the Visualizations tree view have been moved to the left hand side of the application. Furthermore they are no longer on separate tabs, they appear one above the other so that users no longer have to switch back and forth between tabs while laying out the visualizations. Datasets and Visualizations previously each had their own Properties pages, but now there is a single Properties page on the right hand side of the application that is able to display the properties of either type of object. Both the Datasets and Visualizations tree views and the Properties page can be resized or hidden completely to give more screen real estate to the main Visualization view.
* Added multi track event visualizer `TimeIntervalHistoryVisualizationObject` which is useful for visualizing multiple tracks of events having some finite timespan such as multiple speech-to-text streams. This visualizer will be loaded when visualizing streams containing messages of type `Dictionary<string, List<(TimeInterval, string)>>`. The `Dictionary` keys represent unique track IDs, each element in the `List` represents an event to display in the track and contains a tuple of the `TimeInterval` representing the start and end times of the event and a string representing the text that will be displayed inside the time interval. Note that since this is a history visualizer, each message should contain ALL events that have occurred up until the time of the message. This implies that the last message in the stream contains all of the data required to display the visualization.
* Users can now visualize streams by dragging them from the Datasets tree view directly into the main Visualization panel.
* Added 'Snap to Stream' functionality on certain visualizers to snap the cursor to the messages of the snapped stream.
* Added 'Visualize Messages in New Panel' and 'Visualize Latency in New Panel' commands to the stream context menu.
* Psi Studio now automatically attempts to repair corrupted stores when opening them.


New Features in Platform for Situated Intelligence Runtime/Core:

* Initial version of data interop with the introduction of `Microsoft.Psi.Interop`, with support for MessagePack, JSON and CSV data formats and ZeroMQ transport. See the [Interop topic](Interop) for more details.
* New `dynamic` store reader allows reading any stream from any store to `dynamic` primitives or to `ExpandoObject` of `dynamic` without requiring a reference to the .NET type of the stream messages.
* New `PsiStoreTool` command-line tool which allows exploration of available streams in a store, conversion to other formats using interop, and saving to disk or sending over a message queue for consumption by other platforms and/or languages.
* Exposed `Scheduler` as a parameter to `Pipeline` and `Clock` as a parameter to `Scheduler`.
* Multiple handlers may now be registered on start, stop and final pipeline events.
* Improved `#TRACKLEAKS` debug information in `RecyclingPool`.


New Features in Imaging:

* Added `SetPixel` method to `Image`.
* Added `DrawText` extension method for `Image`.
* Added support for `CameraIntrinsics` and `CoordinateSystem` to `SystemCalibration`.
* The `IKinectCalibration` interface and `KinectCalibration` class have been extended to support conversion from depth coordinates to color space coordinates using the new `ToColorSpace` operator.

Bug Fixes:
 
* Fixed several issues where visualization objects were not being displayed in the correct color in Psi Studio.
* Fixed a bug which would sometimes cause Psi Studio to crash when visualizing image streams.
* Fixed a bug which caused the mouse to move the cursor position during playback in Psi Studio.
* Fixed an issue causing streams to disappear at the end of playback in Psi Studio.
* Fixed a crash in Psi Studio when opening a layout created from a store which has since been deleted.
* Fixed a bug which sometimes caused timeline plots to be truncated when loading a layout in Psi Studio.
* Fixed an exception when closing Psi Studio after a live visualization session.
* Fixed a performance issue reading from Psi stores which occurred at the transition between consecutive data files.
* Fixed a bug where the `ImageCompressor` was not properly disposing of an encoded image after decoding it.
* Fixed a bug where `ImagePool` would sometimes return a recycled image with incorrect dimensions.
* Fixed a bug where the pipeline replay interval would sometimes extend beyond the lifetime of a stream being read from a store.
* Fixed a bug which caused `KinectSample` to crash when it detected no faces.
* Fixed the `Scale` image extension method to throw an exception when attempting to call it on an `Image` with an unsupported format.
* Fixed a few intermittently failing unit tests.
* Fixed a bug which sometimes caused a loss of precision when computing the current pipeline time.

---

## __2018/09/13: Beta-release, version 0.5.48.2__

<div style="color:red;font-weight:bold">IMPORTANT NOTE:</div>

For this release we have added the compiler switch /Qspectre to our C++ projects which helps mitigate against Spectre security vulnerabilities.  In order to successfully compile the Platform for Situated Intelligence solution you must upgrade your Visual Studio instance to version 15.7 or later and add the two following components to your Visual Studio installation:

* VC++ 2017 version *version_number* Libs for Spectre (x86 and x64)
* Visual C++ ATL (x86/x64) with Spectre Mitigations

For more information on the Spectre vulnerabilities and information about how to add the above components to Visual Studio, please see the following pages:

* [Visual C++ Team Blog - Spectre mitigations in MSVC](https://blogs.msdn.microsoft.com/vcblog/2018/01/15/spectre-mitigations-in-msvc/)
* [Microsoft Docs - /QSpectre](https://docs.microsoft.com/en-us/cpp/build/reference/qspectre?view=vs-2017)

New Features in Platform for Situated Intelligence Studio:

* Added new PlotStyles to Timeline Visualizer.  Plots can be rendered with the following styles:
    * Direct (default): A straight line is drawn from each message to the next message to create a standard line plot.
    * Step: Messages are joined by a horizontal line followed by a vertical line for visualizing quantized data.
    * None: No lines are drawn between messages.  If you select this plot style then you should also update the MarkerStyle from its default value of None or nothing will be drawn in the plot.
* Visualization Panels in Psi Studio can now be resized by dragging their bottom edge vertically.
* Visualization Panels can now also be re-ordered with the mouse via drag & drop.
* Users can now drag the visible portion of a Timeline Plot to the left or right using the mouse.
* Added modal window while loading a dataset to inform the user of the progress of the data load operation.
* Added new timing information toolbar buttons.  These buttons can be used to display absolute message times, message times relative to the start of the session, and message times relative to the start of the current selection.
* Performance improvements when plotting messages.

New Features in Runtime:

* Join operator now matches against a final secondary message upon stream closing once it can be proven that no better match will exist.
* Added support for Parallel operators that take an Action rather than a Func.
* Adding components once a pipeline is running is no longer supported and now throws an exception. The recommended approach is to add a Subpipeline.
* Consolidated Windows SDK versions.  Previously different parts of the toolset required different WinSDK versions, consolidated so that the only Windows SDK version that Psi now requires is 10.0.17134.0
* Improved how the pipeline shuts down to ensure that all existing messages are drained from it before stopping.

Bug Fixes:
 
* Fixed bug where switching Psi Studio from "Realtime" mode to "Playback" mode would result in the user being unable to move the timeline cursor or reposition the timeline.
* Fixed bug where we were trying to calculate the relative path of a partition to a dataset when the dataset was stored on the local disk but the partitions within the dataset were stored on a network share.
* Fixed bug where loading very large datasets would sometimes crash PsiStudio.

BREAKING CHANGES in this release:

* Removed several versions of the Parallel operator
* Renamed several components for consistency:
    *	AudioSource -> AudioCapture
    *	AudioSourceConfiguration -> AudioCaptureConfiguration
    *	AudioConfiguration (on linux) was eliminated and replaced by two corresponding classes AudioCaptureConfiguration and AudioPlayerConfiguration
    *	TransformImageComponent -> ImageTransformer
    *	AcousticFeatures -> AcousticFeaturesExtractor
    *	AcousticFeaturesConfiguration -> AcousticFeaturesExtractorConfiguration
* Renamed Parallel component to ParallelFixedLength to match naming convention with the other components.
* Partial speech recognition results for the SystemSpeechRecognizer, MicrosoftSpeechRecognizer and BingSpeechRecognizer components are now posted on a new stream named PartialRecognitionResults. The default Out stream now contains only final recognition results.
* Psi Studio will no longer load third party visualizers, for the time being it will only display its built-in visualizers.

---

## __2018/07/02: Beta-release, version 0.4.216.2__

Interim release with support for new devices, runtime enhancements and several API changes, as well as minor bug fixes:

* Added support for RealSense depth camera
* Added the `FFMPEGMediaSource` component for Linux
* Added a [`Subpipeline`] class, enabling [nested pipelines](Writing-Components#SubPipelines)
* [`Parallel`](Parallel-Operator) now uses subpipelines
* [`Sequence`, `Repeat` and `Range` generators](Stream-Generators) now allow time-aligned messages
* Additional minor bug fixes.

Several API changes have been made:

* `Generators.Timer(...)` is now `Timers.Timer(...)`
* `IStartable` has been [replaced by `ISourceControl`/`IFiniteSourceControl`](Writing-Components#SourceComponents) and the [way that components get notified about the pipeline starting and stopping](Writing-Components#PipelineStartStop) has changed

---

## __2018/04/04: Beta-release, version 0.3.16.5__

Interim release with a few changes to the samples and some minor bug fixes:

* ArmControlROSSample is now RosArmControlSample.
* PsiRosTurtleSample is now RosTurtleSample.
* Added LinuxSpeechSample.
* KinectFaceDetector component now outputs an empty list if no face is detected.
* NuGet packages are now marked beta.
* Additional minor bug fixes.

---

## __2018/03/17: Beta-release, version 0.2.123.1__

Initial, beta version of the Platform for Situated Intelligence. Includes the Platform for Situated Intelligence runtime, visualization tools, and an initial set of components (mostly geared towards audio and visual processing). Relevant documents:

* [Documentation](/psi/) - top-level documentation page for Platform for Situated Intelligence.
* [Platform for Situated Intelligence Overview](Platform-Overview) - high-level overview of the platform.
* [NuGet Packages List](List-of-NuGet-Packages) - list of NuGet packages available in this release.
* [Building the Code](Building-the-Codebase) - information about how to build the code.
* [Brief Introduction](Brief-Introduction) - brief introduction on how to write a simple application.
* [Samples](Samples) - list of samples available. 

The [Roadmap](Roadmap) document provides insights about future planned developments.
