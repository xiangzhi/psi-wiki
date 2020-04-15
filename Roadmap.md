This is an initial, beta version of Platform for Situated Intelligence. The current version includes the Platform for Situated Intelligence runtime, visualization tools, and an initial set of components (mostly geared towards audio and visual processing).

The codebase is currently in beta and various aspects of the platform are at different levels of completion and robustness. The are probably still bugs and we will likely be making breaking API changes. 

Moving forward, we plan to prioritize addressing the bugs and issues that are discovered. The roadmap below describes some of the areas we plan to focus our efforts on moving forward (note that this roadmap is tentative and plans may change without notice as we re-assess priorities, etc.):

Apart from fixing bugs and issues, in the next few updates we expect to focus our efforts on: 

### Runtime

- improving the debugging experience by creating tools that allow for visualizing the application pipeline at runtime;
- generalized data importers and exporters;
- evolving the documentation with additional topics;

### Visualization and Platform for Situated Intelligence Studio 

- enabling data annotation scenarios;
- ability to work with other data sources besides \\psi stores;
- refining existing visualizers;
- evolving and documenting the support for third-party visualizers and batch task processing in PsiStudio;

### Data processing and Machine Learning

- evolving and better documenting data-processing APIs;
- providing infrastructure and APIs for supporting the machine-learning development loop; 

### Components

- adding component for AzureKinect;

### Interaction Toolkit

- providing a toolkit (set of components) that enable rapid development of physically situated language interactive systems;