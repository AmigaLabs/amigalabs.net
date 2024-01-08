---
title: Porting third-party libraries
---

[https://github.com/AmigaLabs/libs-ports](https://github.com/AmigaLabs/libs-ports)

Having a new libc means that third-party libraries needs to be compiled and ready to be used with it in various projects, like the WebKit port, games ports etc. We maintain [a repository](https://github.com/AmigaLabs/libs-ports) that contains different libraries and a `Makefile` that can be used to compile the library for all/some of the available libc we have in AmigaOS 4.

### Benefits
The benefits of having such a repository:
- The code can be used as documentation on how a library can be ported
- We have a unified place to compile third-party libraries for the AmigaOS 4 available libcs
- This repository can work as the single place of truth to find latest versions of libraries
- The code is available for anyone to contribute
- If someone would like to update a library, this can be done easily by adapting the Makefile or even the patch file
- The code can be viewed by anyone, and help them understand how a library is possiblee to be ported and apply similar code for their projects


### Files & Guidelines

In the repository, every library has its own folder, which contains the following files:

- A Makefile, which has the needed code to compile the library. This downloads a stable release archive, extracts the files, applies patches (if needed) and does the compilation, supporting multiple libcs. It has also jobs to create an Amiga ready release `lha` file, which has an SDK compatible structure
- An lha archive, which is the ready to be used in AmigaOS 4 SDK. It contains the `local` folder and in there the structure of having the different libraries (static or shared), bin, documentation and header files. This should be structured in a way that the use will only need to extract it in the SDK folder.
- A README.md file, which contains information about the library, the source website this was downloaded from, the supported libcs, the basic commands a user needs to follow to do the compilation and a section with remaining work that needs to be done and known bugs.
- A diff file, which is only necessary if we have to patch the official release to make it work for AmigaOS 4. This is created with the following method, and applied when we initialise the compilation.
    * Initialise the package by downloading and extract the release
    * Do all the necessary changes in the code for it compile succesfully
    * Rename that folder to something else, like `<libname>-patched`
    * Initialise the package again. Now there should be the `<libname>` folder and the `<libname>-patched`
    * Create a patch file with diff

        `diff -ruN <libname> <libname>-patched > patch.diff` 
    
    * Make changes in the make file `init` job to apply that patch

