---
title: Install the clib4 library ports
---

Many times when we develop an application we will need to use open source libraries that will help us on what we are doing, especially when we try to port an application or a game to AmigaOS 4. These libraries need to be compiled for the libc that is used. So, different versions of the libraries are required between newlib and clib4. That means that libraries that compiled for newlib **cannot** be used with clib4.

Trying to help developers move fast, a lot of libraries for clib4 are already ported and are ready to be used. The only thing that needs to be done is to install them in your system. Please, find below how to do it, depending on your dev environment.

## Where to find them

Currently, there are a couple of places to find libraries for clib4.

1. The main choice is an Ubuntu repository we maintain, where you can find a list of the libraries. You can read a detailed list at the [packages file](https://clib4pkg.amigasoft.net/ubuntu/dists/focal/main/binary-amd64/Packages), and you can find the .deb files in the [main folder](https://clib4pkg.amigasoft.net/ubuntu/pool/main/).
2. There is the [AmigaLabs libs-ports](https://github.com/AmigaLabs/libs-ports) repository, where a few libraries are available in lha archives. A few of these archives include newlib and clib2 compatible libraries as well. In this repo the way to build this libraries is shared and feel free to contribute with more, if you would like to see this repo grow.

Please, continue reading on how you can install and use those libraries.

## Cross compiling development environment

If you are using an Ubuntu-based development environment, you can use apt to install clib4 and the libraries you prefer. This is described at the [Clib4 apt packages repository](/clib4/apt-repo) section of this site.

If you are using the Docker and more specifically the [AmigaGCConDocker](https://github.com/walkero-gr/AmigaGCConDocker), then most of the libraries are already installed and in place to be used.

## Native development environment

Although that the .deb archives are meant for Ubuntu-based systems, they can easily be used under AmigaOS 4. Installing them will require a few more manual steps though. Also it will require having clib4 enabled adtools installed in your system.

Here is how you can install them on your system. Remember that the third-party libraries should always be installed under the `SDK:local/clib4/` folder.

For extracting the .deb files you will need to  have the following packages installed in your system. 

- [deark](https://os4depot.net/?function=showfile&file=utility/archive/deark.lha) 
- [xad_xz](https://os4depot.net/?function=showfile&file=utility/archive/xad_xz.lha)

Let's see an example. Let's say that we want to install **libbz2** library:

1. Firstly, check the the [packages file](https://clib4pkg.amigasoft.net/ubuntu/dists/focal/main/binary-amd64/Packages) and find the libbz2. Right now the info we find is
```
Package: libbz2-clib4
Version: 1.0.8.1
Architecture: amd64
Maintainer: Andrea Palmatè <os4test@amigasoft.net>
Filename: pool/main/bzip-1.0.8_amd64.deb
Size: 113252
MD5sum: d857ba37440f76a0c4077c15794dff90
SHA1: 08ef6c159c75f634c87375b90299667da0c34324
SHA256: 821374a0707d291ea8dc3deb6e37211847abedf654d6030efee1411e706f073e
Section: libdevel
Description: libbz2 development library for clib4
```
2. The filename is **bzip-1.0.8_amd64.deb**, which we can download from the [main repository folder](https://clib4pkg.amigasoft.net/ubuntu/pool/main/)
3. Save that file to `Ram:` and open a shell in it.
4. Extract the .deb file with the following command
```
Deark -od Ram: ram:bzip-1.0.8_amd64.deb
```
5. This will output three files, but the one we care about is the `output.002.data.tar.xz`. We can extract it with the following command
```
xadunfile Ram:output.002.data.tar.xz Ram:
```
6. That will create the `output.002.data.tar`. Extract it again with the following command
```
xadunfile Ram:output.002.data.tar Ram:
```
7. This will create the `Ram:./usr/ppc-amigaos/SDK` folder, from which we need to copy all the SDK files with the following command
```
copy ALL CLONE Ram:./usr/ppc-amigaos/SDK/ SDK:
```

The above steps can be followed for every .deb file from the Ubuntu repository.