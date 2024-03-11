---
title: Configure ncurses terminal files
---


## How to configure ncurses terminal files

Clib4 now has a working ncurses library support. But to use the library you need to configure AmigaOS4 so the library can find the correct terminal.

In the package there is a directory called `terminfo` that contains clib4 terminal specifications.  
This directory needs to be copied into OS4 `ENVARC:` folder. Final tree must be this one:

* ENVARC:
    * terminfo
        * a
            * amiga-clib4
            * amiga-clib4-mono

Once files are copied you can use `setenv` command in your shell to choose the right file you want to use.  
The default one is **amiga-clib4**. This version supports 16 colors and it is working very well even if it has some flaws occasionally. 
**amiga-clib4-mono** is the mono version but it has some problems when you scroll a line horizontally and exceed the last column.

So for example to set **amiga-clib4** terminal, use:

`setenv TERM amiga-clib4`

Both files are created via **termcap.os4** file using `tic` command present into ncurses package.

To use ncurses library, install or compile [ncurses](https://github.com/afxgroup/ncurses6.3) package and add -I{SDK_PATH}/local/clib4/ncurses to your makefile

To compile ncurses use this configure command line:

```bash
./configure --host=ppc-amigaos --prefix={SDK_PATH}/local/clib4 CFLAGS="-O3 -mcrt=clib4 -gstabs" CXXFLAGS="-O3 -mcrt=clib42 -gstabs" LDFLAGS="-mcrt=clib4 -athread=native" --without-dlsym --without-ada --with-termpath=ENVARC:termcap.os4 --disable-db-install --without-manpages -disable-rpath-hack --enable-getcap --disable-stripping --enable-termcap --enable-wgetch-events --enable-sigwinch --enable-ext-colors --with-terminfo-dirs=ENVARC:terminfo --with-default-terminfo-dir=ENVARC:terminfo
```

Replace _{SDK_PATH}_ with your SDK installation path.

It is also possible to enable wide char support but since AmigaOS4 shell doesn't support unicode it is useless and charaters are rendered wrong.

* Make sure you have a _mono_ font set in Console prefs under Amiga OS4.
* Set "Full ANSI Palette" in Console prefs under Amiga OS4