* Why is my build failing with the error message, "Spectre-mitigated libraries are required for this project"?
    * Launch the Visual Studio Installer -> Modify Under the Individual Components tab, install the following components (see also, [Building the Codebase](Building-the-Codebase)):
        * MSVC v142 - VS 2019 C++ x64/x86 Spectre-mitigated libs (v14.xx, latest version)
        * C++ ATL for latest v142 build tools with Spectre Mitigations (x86 & x64)Tes