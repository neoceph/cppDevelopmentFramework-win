# A comprehensive framework for C++ development
- This CMake structure includes googletest as an external library using git submodule and organizes unittests in a separate executable.
- Organizes codes on a separate src directory where headers and sources have corresponding headers and sources directory.
- Developed on Ubuntu 18.04

## Requirements for Windows
- CMake 3.21 or above
- Ninja from [here](https://github.com/ninja-build/ninja/releases)
- NSIS 3.0 and up (required for windows installer package generation)
    - For windows NSIS can be downloaded and installed from [here](https://nsis.sourceforge.io/Download)
## Packing installer
1. Assigning a git tag is easy 
    - `git tag -a v1.0.3 -m 'Version 1.0.3 - Feature 1'`
    - `git tag` lists all the available tags.
    - Check other `git tag` details [here](https://git-scm.com/book/en/v2/Git-Basics-Tagging).
2. Build the code 
    - Use vscode's 'Build' button with 'CmakeTool' extension otherwise,
    - create a directory named 'build' with `mkdir build`
    - change to build directory with `cd build`
    - copy the configure.sh to build directory with `cp ../configure.sh .`
    - execute configure .sh with `./configure.sh`. You may need to change executable permission with `chmod +x configure.sh`
    - compile the code `make -j $(nproc)`. The  `$(nproc)` part of the code identifies the number of cpus and use all of them to compile the code.
3. From within the build directory issue `cpack` command to create installer. All the .sh, .tar.gz, and .tar.Z files are the installer in different format.

## Changing the name of the project
- Project name needs to be changed in two places
    1. Line #3 of the root directory CMakeLists.txt file `set(Project Project-Name)`. The 'Project-Name' can be changed to any appropriate name.
    2. Line #35 `PROJECT_NAME = "Test Projects"` of the [Doxyfile.in](./documentation/Doxyfile.in) located in the *./documentation* directory. 
    