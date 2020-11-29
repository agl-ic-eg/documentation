---
title: Project Architecture
---

The following tree structure represents a typical CMake project
directory structure:

```tree
<project-root-path>
|
├── CMakeLists.txt
│
├── autobuild/
│   ├── agl
│   │   └── autobuild
│   ├── linux
│   │   └── autobuild
│   └── windows
│       └── autobuild
├── conf.d/
│   ├── packaging/
│   │   ├── rpm
│   │   │   └── package.spec
│   │   └── deb
│   │       ├── package.dsc
│   │       ├── debian.package.install
│   │       ├── debian.changelog
│   │       ├── debian.compat
│   │       ├── debian.control
│   │       └── debian.rules
│   ├── cmake
│   │   ├── 00-debian-osconfig.cmake
│   │   ├── 00-suse-osconfig.cmake
│   │   ├── 01-default-osconfig.cmake
│   │   └── config.cmake
│   └── wgt
│       ├── icon.png
│       └── config.xml.in
├── <target>
│   └── <files>
├── <target>
│   └── <file>
└── <target>
    └── <files>
```

| File or Directory | Parent | Description |
|----|----|----|
| *root_path* | n/a | CMake project root path. Holds the master CMakeLists.txt file and all general project files.
| CMakeLists.txt | The master CMakeLists.txt file.
| autobuild/ | *root_path* | Scripts generated from app-templates to build packages the same way for differents platforms.
| conf.d/ | *root_path* | Holds needed files to build, install, debug, and package an AGL application project.
| packaging/ | conf.d/ | Contains output files used to build packages.
| cmake/ | conf.d/ | Minimally contains the config.cmake file, which is modified from the sample provided in the app-templates submodule.
| wgt/ | conf.d/ | Contains config.xml.in and optionaly the test-config.xml.in template files that are modified from the sample provided with the CMake module for the needs of the project.  For more details, see the config.xml.in.sample and test-config.xml.in.sample files.
| *target* | *root_path* | A target to build, which is typically a library or executable.

When building projects using CMake, the build process automatically detects
the `CMakeLists.txt` and `*.cmake` files.
To help with this process, the `PROJECT_SRC_DIR_PATTERN` variable
is used for recursive pattern searching from the CMake project's
*root_path* downward.
Each sub-folder below *root_path* in the project is searched and included
during compilation.
The directories matching the pattern `PROJECT_SRC_DIR_PATTERN` variable
are scanned.

**NOTE:** The `PROJECT_SRC_DIR_PATTERN` variable defaults to "*".

When the `CMakeLists.txt` file is found, the directory in which it is found
is automatically added to the CMake project.

Similarly, when a file whose extension is `.cmake` is found, the directory in
which that file resides is also added to the CMake project.