Platform for Situated Intelligence Studio (PsiStudio in short) now supports loading custom 3rd party visualizers from external DLLs. 

To enable PsiStudio to load external DLLs, you can add an `<AdditionalAssemblies>` xml tag inside the main `<PsiStudioSettings>` tag in the PsiStudioSettings.xml file. This settings file is stored in the PsiStudio folder under your user Documents folder. You can specify the assemblies containing visualizers that you would like PsiStudio to load with an `<AdditionalAssemblies>` tag, for example:

```text
    <AdditionalAssemblies>C:\MyVisualizers\MyVisualizers.dll</AdditionalAssemblies>
```

Multiple assemblies can be specified via a semicolon-separated list, like below:

```text
    <AdditionalAssemblies>C:\MyVisualizers\MyVisualizers.dll;C:\MyOtherVisualizers\MyOtherVisualizers.dll</AdditionalAssemblies>
```

Some of the visualizers that \\psi provides are implemented in projects/DLLs outside of PsiStudio -- this was done to remove some of the dependencies PsiStudio had because of these visualizers. If desired, these visualizers can be loaded by PsiStudio via the 3rd party mechanism described above. Here is a list of current projects / DLLs from \\psi that contain visualizers:

| Project | Visualizers |
| :-------- | :----------------- |
| `Microsoft.Psi.AzureKinect.Visualization.Windows.x64` | Visualizers for AzureKinect tracked bodies. |
| `Microsoft.Psi.Kinect.Visualization.Windows` | Visualizers for Kinect tracked bodies. |
