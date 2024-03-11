---
title: clib4
---

## Welcome to the clib4 wiki!

Here you can find some informations or tips that can help you to use clib4 correctly.

## Clib4 structure

Compared to the original clib2, this version has been refactored to use different directories for source code.
The old one had all files in src folder that was difficult to browse.
The new structure is:

- root 
     - documentation 
     - library
          - amiga
          - argz
          - byteswap
          - ...
     - libs
     - test_programs

In **library** folder the source code is organized by include when possible so if you are searching for a `stdio` function you will find it in `stdio` folder.

In the test_programs folder you will find all examples and tests that are actually available for clib4.  
There is also a _GNUMakefile.os4_ that will compile everything so you will be able to test them without compiling file by file. It will create a **build** directory with all compiled files
