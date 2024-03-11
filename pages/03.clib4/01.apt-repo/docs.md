---
title: Clib4 apt packages repository
---


clib4 now has a working in progress ubuntu packages repository.

To use it you have to:

1) Download the [public key](https://clib4pkg.amigasoft.net/ubuntu/clib4.gpg) using this command `curl -fsSL https://clib4pkg.amigasoft.net/ubuntu/clib4.gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/clib4.gpg`
2) Add repository using: `echo "deb https://clib4pkg.amigasoft.net/ubuntu/ focal main" | sudo tee /etc/apt/sources.list.d/clib4.list`.
3) Execute `apt update`

## How to create a package

To create an apt package you have to follow some specific rules. You can find an example [here](https://ivanitlearning.wordpress.com/2019/10/27/writing-your-own-deb-package/)  
However an example of package should have this format:

<img width="338" alt="image" src="https://github.com/amigalabs/clib4/assets/484672/5ab8c555-c381-4122-98c9-369696d0b624">  

### Add control file

In the DEBIAN/control file you have to fill all the informations about package. This is an example of control file:

```
Package: gdbm-clib4
Version: 1.19  
Maintainer: Andrea Palmatè <os4test@amigasoft.net>  
Architecture: amd64  
Section: libdevel  
Description: GNU dbm for clib4
```

### Package deb file

After creating the correct tree you can use the `dpkg` to create the `.deb` package. For example:

`dpkg --build gdbm_1.19_amd64`
