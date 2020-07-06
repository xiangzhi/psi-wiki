This document contains a [**list of components**](#ListOfComponents) available in the Platform for Situated Intelligence repository, as well as [**pointers to other third-party repositories**](#ThirdParty) containing other Platform for Situated Intelligence components.

<a name="ListOfComponents"></a>

## 1. Components in the \psi Repository

The table below contains the list of \psi components that are available in the current release, together with the namespace in which the component can be found (in general, the NuGet packages in which you can find the component follow the same naming convention, potentially with additional platform suffixes)

| Name / Namespace | Description | Windows | Linux |
| :--- | :---- | :-----: | :----: |
| <h3>Sensors</h3> | | | |
| __AudioCapture__ <br> Microsoft.Psi.Audio | Component that captures and streams audio from an input device such as a microphone |	AnyCPU | Yes |
| __MediaCapture__ <br> Microsoft.Psi.Media | Component that captures and streams video and audio from a video camera (audio is currently supported only on the Windows version) | X64 | Yes |
| __RealSenseSensor__ <br> Microsoft.Psi.RealSense |	Component that captures and streams video and depth from an Intel RealSense camera | X64 | No | 
| __AzureKinectSensor__ <br> Microsoft.Psi.AzureKinect| Component that captures and streams information (video, depth, IMU, tracked bodies etc.) from an Azure Kinect sensor | x64 | Yes | 
| __AzureKinectBodyTracker__ <br> Microsoft.Psi.AzureKinect | Component that performs body tracking from the depth/IR images captured by the Azure Kinect sensor. | x64 | Yes | 
| __KinectSensor__ <br> Microsoft.Psi.Kinect| Component that captures and streams information (video, depth, audio, tracked bodies, etc.) from a Kinect One (v2) sensor | AnyCPU | No | 
| <h3>File sources</h3> | | | |
| __FFMPEGMediaSource__ <br> Microsoft.Psi.Media | Component that streams video and audio from an MPEG file | No | Yes | 
| __MediaSource__ <br> Microsoft.Psi.Media | Component that streams video and audio from a media file | X64 | No |
| __WaveFileAudioSource__ <br> Microsoft.Psi.Audio | Component that streams audio from a WAVE file | AnyCPU | Yes |
| __File writers__ | | | |
| __WaveFileWriter__ <br> Microsoft.Psi.Audio | Component that writes an audio stream into a WAVE file | AnyCPU | Yes |
| __MPEG4Writer__ <br> Microsoft.Psi.Media | Component that writes video and audio streams into an MPEG-4 file | X64 | No |
| <h3>Imaging</h3> | | | |
| __DepthImageEncoder__ <br> Microsoft.Psi.Imaging | Component that encodes a depth image using a specified encoder (e.g. PNG) | AnyCPU | Yes |
| __ImageEncoder__ <br> Microsoft.Psi.Imaging | Component that encodes an image using a specified encoder (e.g. JPEG, PNG) | AnyCPU | Yes |
| __DepthImageDecoder__ <br> Microsoft.Psi.Imaging | Component that decodes a depth image using a specified decoder (e.g. JPEG, PNG) | AnyCPU | Yes |
| __ImageDecoder__ <br> Microsoft.Psi.Imaging | Component that decodes an image using a specified decoder (e.g. JPEG, PNG) | AnyCPU | Yes |
| __ImageTransformer__ <br> Microsoft.Psi.Imaging | Component that transforms an image given a specified transformer | AnyCPU | Yes |
| <h3>Vision</h3> | | | |
| __ImageAnalyzer__ <br> Microsoft.Psi.CognitiveServices.Vision | Component that performs image analysis via [Microsoft Cognitive Services Vision API](https://azure.microsoft.com/en-us/services/cognitive-services/computer-vision/). | AnyCPU | No |
| __FaceRecognizer__ <br> Microsoft.Psi.CognitiveServices.Face | Component that performs face recognition via [Microsoft Cognitive Services Face API](https://azure.microsoft.com/en-us/services/cognitive-services/face/). | AnyCPU | No |
| <h3>Audio processing</h3> | | | |
| __AudioResampler__ <br> Microsoft.Psi.Audio | Component that resamples an audio stream into a different format | AnyCPU | No |
| __AcousticFeaturesExtractor__ <br> Microsoft.Psi.Audio | Component that extracts acoustic features (e.g. LogEnergy, ZeroCrossing, FFT) from an audio stream | AnyCPU | Yes |
| <h3>Speech processing</h3> | | | |
| __SystemVoiceActivityDetector__ <br> Microsoft.Psi.Speech | Component that performs voice activity detection by using the desktop speech recognition engine from `System.Speech__ | AnyCPU | No | 
| __SimpleVoiceActivityDetector__ <br> Microsoft.Psi.Speech | Component that performs voice activity detection via a simple heuristic using the energy in the audio stream | AnyCPU | Yes |
| __SystemSpeechRecognizer__ <br> Microsoft.Psi.Speech | Component that performs speech recognition using the desktop speech recognition engine from `System.Speech`. | AnyCPU | No | 
| __SystemSpeechIntentDetector__ <br> Microsoft.Psi.Speech | Component that performs grammar-based intent detection using the desktop speech recognition engine from `System.Speech`. | AnyCPU | No |
| __MicrosoftSpeechRecognizer__ <br> Microsoft.Psi.MicrosoftSpeech | Component that performs speech recognition using the Microsoft Speech Platform SDK. | AnyCPU | No |
| __MicrosoftSpeechIntentDetector__ <br> Microsoft.Psi.MicrosoftSpeech | Component that performs grammar-based intent detection using the speech recognition engine from the Microsoft Speech Platform SDK. | AnyCPU | No |
| __AzureSpeechRecognizer__ <br> Microsoft.Psi.CognitiveServices.Speech | Component that performs speech recognition using the [Microsoft Cognitive Services Speech to Text Service](https://azure.microsoft.com/en-us/services/cognitive-services/speech/). | AnyCPU | Yes |
| __LUISIntentDetector__ <br> Microsoft.Psi.CognitiveServices.Language | Component that performs intent detection and entity extraction using the [Microsoft Cognitive Services LUIS API](https://www.luis.ai/). | AnyCPU | Yes |
| __PersonalityChat__ <br> Microsoft.Psi.CognitiveServices.Language | Component that generates dialogue responses to textual inputs using the [Microsoft Cognitive Services Personality Chat API](https://labs.cognitive.microsoft.com/en-us/project-personality-chat). | AnyCPU | Yes |
| __BingSpeechRecognizer__ <br> Microsoft.Psi.CognitiveServices.Speech | <div style="color:red;font-weight:bold">[DEPRECATED]</div> Component that performs speech recognition using the [Microsoft Cognitive Services Bing Speech API](https://docs.microsoft.com/en-us/azure/cognitive-services/Speech). | AnyCPU | Yes |
| <h3>Calibration</h3>| | | |
| __ProjectTo3D__ <br> Microsoft.Psi.Calibration | Component that projects 2D color-space points into 3D camera-space points in the depth camera's coordinate system. | AnyCPU | Yes |
| <h3>Output</h3> | | | |
| __AudioPlayer__ <br> Microsoft.Psi.Audio | Component that plays back an audio stream to an output device such as the speakers. | AnyCPU | Yes |
| __SystemSpeechSynthesizer__ <br> Microsoft.Psi.Speech | Component that performs speech synthesis via the desktop speech synthesis engine from `System.Speech`. | AnyCPU | No |


<a name="ThirdParty"></a>

## 2. Repositories with Components by Third Parties

You might also be interested in exploring the repositories below containing components for the Platform for Situated Intelligence ecosystem written by third parties, not affiliated with Microsoft. Microsoft makes NO WARRANTIES about these components, including about their usability or reliability.

| Repo | Description |
| :-- | :-- |
| https://github.com/bsu-slim/psi-components | Components developed by the [SLIM research group](https://coen.boisestate.edu/slim/) at Boise State University. |