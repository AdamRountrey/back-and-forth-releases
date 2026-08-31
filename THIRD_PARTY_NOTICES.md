# Third-Party Notices

This file summarizes third-party software used by back-and-forth or included
in packaged Windows builds. It is informational and does not replace the full
license texts and compliance requirements published by those projects. Release
builders should verify the exact third-party DLL set and license obligations for
each packaged build.

The `LICENSE` file in this repository applies to the back-and-forth source
code. It does not change the terms for NVIDIA software, Qt, or other
third-party components.

## Runtime Components

### Qt 6

back-and-forth uses Qt 6 Widgets for the graphical user interface and packages
Qt runtime DLLs/plugins in Windows builds, including:

- `Qt6Core.dll`
- `Qt6Gui.dll`
- `Qt6Widgets.dll`
- `Qt6PrintSupport.dll`
- `Qt6Xml.dll`
- `platforms/qwindows.dll`
- optional Qt plugins such as `styles/qmodernwindowsstyle.dll`,
  `imageformats/qgif.dll`, and `imageformats/qico.dll`

Qt is available under commercial and open-source licenses. The packaged build
is intended to use Qt's open-source LGPL terms. A binary release should include
the applicable Qt license texts, preserve required notices, and satisfy the
LGPL requirements for dynamically linked Qt libraries, including replacement or
relinking of the Qt components where required by the selected Qt license version.
See the Qt project and Qt Company license documentation for the complete license
terms and compliance requirements for the exact Qt version packaged.

Project: https://www.qt.io/

### zlib

back-and-forth uses zlib for DEFLATE compression, including PNG/ZIP-related
output paths and the packaged `z.dll` runtime.

zlib is distributed under the zlib License.

Project: https://zlib.net/

### NVIDIA CUDA Runtime and cuFFT

CUDA-enabled builds link against NVIDIA CUDA Runtime and cuFFT and packaged
Windows builds may include redistributable CUDA DLLs, such as:

- `cudart64_*.dll`
- `cufft64_*.dll`

These components are NVIDIA software and are governed by NVIDIA's CUDA Toolkit
End User License Agreement and redistributable terms. Users still need a
compatible NVIDIA driver.

Project: https://developer.nvidia.com/cuda-toolkit

### MC33++

back-and-forth includes the MC33++ 5.4 lookup table and uses a CUDA execution
port of its corrected Marching Cubes 33 case-selection logic. The vendored
reference is pinned to commit `252dabd7ea6dea3d0529e66b047adbe878caf2fa`
and is also compiled as an independent CPU oracle in scientific validation
tests.

MC33++ is Copyright (c) 2019 David E. Vega M. and is distributed under the MIT
License. The complete upstream license is preserved at
`third_party/MC33_cpp_library/LICENSE.txt`.

Project: https://github.com/dvega68/MC33_cpp_library

### Matplotlib viridis color data

The optional PLY depth-map visualization uses nine RGB anchor values sampled
from Matplotlib's viridis color map and linearly interpolates between them.
Matplotlib itself is not linked or included in the application runtime.

Copyright (c) 2012- Matplotlib Development Team; All Rights Reserved.
Matplotlib is distributed under a BSD-compatible license based on the Python
Software Foundation license. See the Matplotlib license for the complete terms.

Project and license: https://matplotlib.org/stable/project/license.html

### double-conversion

Packaged Qt builds may include `double-conversion.dll`, used by Qt for binary
to decimal and decimal to binary conversion.

double-conversion is distributed under a BSD-style license.

Project: https://github.com/google/double-conversion

### PCRE2

Packaged Qt builds may include PCRE2 runtime DLLs such as `pcre2-16.dll` and
`pcre2-8.dll`.

PCRE2 is distributed under the BSD 3-Clause License.

Project: https://www.pcre.org/

### MD4C

Packaged Qt builds may include `md4c.dll` and `md4c-html.dll`, used by Qt for
Markdown support.

MD4C is distributed under the MIT License.

Project: https://github.com/mity/md4c

### Microsoft Windows System Libraries

On Windows, back-and-forth uses system APIs such as Windows Imaging Component,
OLE, and OpenGL through system libraries including `windowscodecs`, `ole32`,
and `opengl32`. These are provided by Microsoft Windows and are not distributed
as part of this repository.

## Build And Tooling Dependencies

Building from source may require additional tools and SDKs, including CMake,
Visual Studio Build Tools, vcpkg, Qt development packages, and the NVIDIA CUDA
Toolkit. Those tools are governed by their respective licenses and are not
covered by the back-and-forth software license.
