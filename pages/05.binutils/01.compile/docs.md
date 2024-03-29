---
title: Compile
---

### Compile with clib4

To compile binutils there is a need to use a cross compiling environment, preferably based on Linux. The `clib4` is required to be installed and adtools supporting it.

We are using Docker because it simplifies the process a lot. This means that Docker needs to be installed and running on the system to compile the binutils. Otherwise, a similar development environment needs to be created.  

The following steps describe the compilation process:

```bash
docker run -it --rm --name binutils_clib4 -v "${PWD}:/opt/code" -w /opt/code walkero/amigagccondocker:os4-gcc11-exp /bin/bash

su amidev

git clone https://github.com/AmigaLabs/binutils-gdb.git --branch nativeOS4-build-clib4 --depth 1

mkdir -p binutils-gdb/native-build
cd binutils-gdb/native-build

rm ../gas/doc/.dirstamp

CFLAGS="-mcrt=clib4 -DHAVE_POSIX_SIGNALS" CXXFLAGS="-mcrt=clib4 -DHAVE_POSIX_SIGNALS" ../configure --disable-plugins --disable-gdb --disable-sim --host=ppc-amigaos --target=ppc-amigaos --prefix="$(realpath ../dist-clib4)"

make -j$(shell nproc)
make install
```

### Compile with newlib

Using the same Docker image and similar process it is possible to compile binutils with newlib.

The following steps describe this compilation process:

```bash
docker run -it --rm --name binutils_clib4 -v "${PWD}:/opt/code" -w /opt/code walkero/amigagccondocker:os4-gcc11-exp /bin/bash

su amidev

git clone https://github.com/AmigaLabs/binutils-gdb.git --branch nativeOS4-build-newlib --depth 1

mkdir -p binutils-gdb/native-build
cd binutils-gdb/native-build

rm ../gas/doc/.dirstamp

CFLAGS="-mcrt=newlib -Wno-sign-compare" LDFLAGS="-lunix" CXXFLAGS="-mcrt=newlib" ../configure --disable-plugins --disable-gdb --disable-sim --host=ppc-amigaos --target=ppc-amigaos --prefix="$(realpath ../dist-newlib)"

make -j$(shell nproc)
make install
```

### files

The new native binutils can be found in the `binutils-gdb/dist-newlib` or `binutils-gdb/dist-clib4` folder, depending on which library and method you used to compile it from above. These files can be used and replace the SDK ones.

**Attention**: This version is still experimental, and there might be non-discovered bugs. If you discover an issue, please report it at [our repository](https://github.com/AmigaLabs/binutils-gdb/issues)