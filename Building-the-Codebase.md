To build, first you will need to clone the Platform for Situated Intelligence [github repo](https://github.com/microsoft/psi "\psi"). Then, depending on which operating system you are, follow the steps below.

## On Windows

Starting with release 0.12.53.2, Visual Studio 2019 is required to build the `Psi.sln` solution on Windows.

__Setup Visual Studio 2019__:

Install [Visual Studio 2019](https://www.visualstudio.com/vs/). The Community Edition of Visual Studio is sufficient. Make sure the following features are installed (you can check these features by running the Visual Studio Installer again and looking at both the Workloads and Individual components tabs):

* Workloads:
  * __.NET desktop development__
  * __.NET Core cross-platform development__
  * __Desktop development with C++__
* Individual Components:
  * __MSVC v142 - VS 2019 C++ x64/x86 Spectre-mitigated libs (v14.xx, latest version)__
  * __C++ ATL for latest v142 build tools with Spectre Mitigations (x86 & x64)__
  * __Windows 10 SDK (10.0.18362.0)__

__Optional prerequisites__:

A couple of the projects in the Platform for Situated Intelligence codebase have install prerequisites. If you want to build these projects as part of the solution, you will need to install the prerequisites below. If the prerequisites are not found, these projects will not be build (the rest of the solution will build correctly.)

* __Open CV Sample__: the __OpenCVSample__ and __OpenCVSample.Interop__ sample projects (from the `Samples` folder) require an installation of OpenCV. OpenCV can be obtained [here](http://opencv.org/releases.html). The sample relies on version 4.1.1. For these projects to build correctly, will need to set an environment variable named `OpenCVDir_V4` that points to your OpenCV installation. The path should be the root of OpenCV which contains the _sources_ directory (along with the license). For instance, `D:\OpenCV-4.1.1\opencv`.

* __RealSense Support__: the __Microsoft.Psi.RealSense.Windows.x64__ and __Microsoft.Psi.RealSense_Interop.Windows.x64__ projects (from the `Sources\RealSense` folder) provide support for the Intel RealSense depth camera. This requires the [Intel RealSense SDK v2.0](https://realsense.intel.com/sdk-2/) to be installed. Additionally, you will need to set an environment variable named `RealSenseSDKDir` that points to the location in which you installed the SDK (typically _C:\Program Files (x86)\Intel RealSense SDK 2.0_).

* __FFMPEG Support__: the __Microsoft.Psi.Media.Windows.x64__ and __Microsoft.Psi.Media_Interop.Windows.x64__ projects (from the `Sources\Media` folder) provide support for playback of media files via [FFMPEG](https://ffmpeg.org/). Use of FFMPEG requires the [FFMPEG SDK](https://ffmpeg.org/download.html#build-windows) to be installed. Additionally, you will need to set an environment variable named `FFMPEGDir` that points to the location in which you installed the SDK. The path should be the root of the FFMPEG folder which contains the _LICENSE.txt_ file. For instance, `D:\FFMPEG\ffmpeg-20180227-fa0c9d6-win64-dev`.

* __Microsoft.Speech Recognizer__: the __Microsoft.Psi.MicrosoftSpeech.Windows__ project (from the `Sources\Integrations` folder), which includes the \\psi components for the Microsoft.Speech recognizer requires the [Microsoft Speech Platform SDK v11.0](http://go.microsoft.com/fwlink/?LinkID=223570). Note that only the 64-bit version of the SDK is currently supported. Additionally, you will need to set an environment variable named `MsSpeechSdkDir` that points to the location in which you installed the SDK. The path should be the root of the SDK folder which contains the _Assembly_ directory. By default, this is `C:\Program Files\Microsoft SDKs\Speech\v11.0`. In order to run applications developed using this component, you will also need to install the [Microsoft Speech Platform Runtime v11.0](http://go.microsoft.com/fwlink/?LinkID=223568) as well as the applicable [Language Pack](http://go.microsoft.com/fwlink/?LinkID=223569) for the speech recognition language you wish to use (e.g. en-US).

__Build__:

* Launch Visual Studio.
* Open the `Psi.sln` solution from the root of your cloned repo.
* From the *Build* menu choose *Rebuild Solution*.
  * This will build the currently selected configuration (Release or Debug).
  * It is a good idea to build both configurations - select the other configuration and rebuild.

## On Linux

Under Linux, we recommend using [Visual Studio Code](https://code.visualstudio.com/). A subset of projects are cross-platform and some offer Linux-specific implementations.

__Prerequisites__:

You will need .NET Core 3.1 on Linux. You can find the [installation instructions here](https://docs.microsoft.com/en-us/dotnet/core/linux-prerequisites?tabs=netcore31). Be sure to install the 3.1 version, as some of Psi packages use this. To check your installation, run `dotnet --info` in your terminal. You should see lists of information such as Runtime Environment and SDK Versions.

Although \psi is built on .NET Standard, IL assembly still depends on _Mono's_ `ilasm` tool. [Install at least `mono-devel`](https://www.mono-project.com/download/stable/#download-lin). It is important to install (or reinstall) Mono _after_ the .NET SDK. If you see an error such as `IL/MemoryAccess.il (16, 0): error : irrecoverable synatx error` it is likely due to an incompatible `ilasm` installation (try reinstalling Mono).

__Build__:

To build, launch the `./build.sh` script. This will build all individual projects that support Linux by calling individual `build.sh` scripts that are associated with each of these projects.