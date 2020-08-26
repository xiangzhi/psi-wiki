# Welcome

The wiki contains the main documentation pages for Platform for Situated Intelligence. Please use the sidebar to the right to navigate the wiki. If you would like to make edits, please fork and send a pull request.

# Platform for Situated Intelligence: A Quick Overview

<b>P</b>latform for <b>S</b>ituated <b>I</b>ntelligence (which we abbreviate as \\psi, pronounced like the greek letter) is __an open, extensible framework that accelerates research and development, on multimodal, integrative-AI systems__. These are systems that operate with different types of streaming data, e.g. video, audio, depth, IMU data, etc., and leverage multiple component technologies to process this data at low latency. Example range from systems that sense and act in the physical world, such as interactive robots, drones, virtual interactive agents, personal assistants, interactive instrumented meeting rooms, to software systems that mesh human and machine intelligence, all the way to applications based on small devices that process streaming sensor data. 

In recent years, we have seen significant progress with machine learning techniques on various perceptual and control problems. At the same time, building this type of end-to-end, multimodal, integrative-AI systems that can sense, reason about and act in the open world remains a challenging, error-prone and time-consuming engineering task. Numerous challenges stem from the sheer complexity of these systems and are amplified by the lack of appropriate infrastructure and development tools.

Platform for Situated Intelligence aims to address these issues and provide a robust basis for developing, fielding and studying multimodal, integrative-AI systems. By releasing the code under an open-source [MIT License](https://github.com/Microsoft/psi/blob/master/LICENSE.txt), we hope to enable the community to contribute, grow the \\psi ecosystem, and further lower the engineering barriers and foster more innovation in this space.

Concretely, the framework provides (1) a **runtime** for working with multimodal, temporally streaming data, (2) a set of **tools** to support development, debugging, and maintenance, and (3) an ecosystem of **components** that simplifies application writing and facilitates technology reuse. 

**Runtime**. The \\psi runtime and core libraries provide a parallel, coordinated computation model centered on online processing of streaming data. Time is a first-order citizen in the framework. The runtime provides abstractions for computing over streaming data and for reasoning about time and synchronization and is optimized for low-latency from the bottom up. In addition, the runtime provides fast persistence of generic streams, enabling data-driven development scenarios.

**Tools**. \\psi provides a powerful set of tools that enable testing, data visualization, annotation replay, analytics and machine learning development for integrative-AI systems. The visualization subsystem allows for live and offline visualization of streaming data. A set of data processing APIs allow for re-running algorithms over collected data, data analytics and feature extraction for machine learning. The image below illustrates Platform for Situated Intelligence Studio, i.e. the multimodal data visualization tool.

![Platform for Situated Intelligence Studio](PsiStudio.jpg)

**Components**. \\psi provides a wide array of AI technologies encapsulated into \\psi components. \\psi applications can be easily developed by wiring together \\psi components. The [initial set of components](List-of-Components) focuses on multimodal sensing and processing, and includes sensor components for cameras and microphones, audio and image processing components, as well as components that provide access to Azure Cognitive Services. We hope to create through community contributions a broader ecosystem of components that will allow us to more easily leverage each other's work.

More information about upcoming features in Platform for Situated Intelligence is available in the [Roadmap](Roadmap) document.
