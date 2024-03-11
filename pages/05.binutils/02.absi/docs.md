---
title: Power Architecture(tm) 32-bit Application Binary Interface Supplement - AmigaOS
---

**Table of content**
- [Preface](#preface)
	- [Intention](#intention)
- [Introduction](#introduction)
- [Object Files](#object-files)
	- [ELF Header](#elf-header)
		- [Additional Information](#additional-information)
		- [References:](#references)
	- [Special Sections](#special-sections)
		- [References:](#references-1)
	- [Symbol Table](#symbol-table)
		- [Marker Symbols](#marker-symbols)
		- [Entry Symbol](#entry-symbol)
		- [Small Data Model Symbol](#small-data-model-symbol)
		- [PPC marker](#ppc-marker)
		- [Additional information](#additional-information-1)
	- [Relocation Types](#relocation-types)
		- [PPC-AmigaOS Specific Relocations](#ppc-amigaos-specific-relocations)
- [Target specific](#target-specific)
	- [Base Relative Address Model](#base-relative-address-model)
	- [Specific strip operation](#specific-strip-operation)
		- [Additional Information](#additional-information-2)
		- [References:](#references-2)
- [Other Oddities](#other-oddities)
	- [Pointer Equality](#pointer-equality)
	- [Keep Unused Section Symbol](#keep-unused-section-symbol)
	- [GOT Is Marked as Executable](#got-is-marked-as-executable)
	- [Dynamic Sections Are Read-Only](#dynamic-sections-are-read-only)
	- [Eliminate Copy Relocations](#eliminate-copy-relocations)
	- [Constructor List](#constructor-list)
	- [Default linking](#default-linking)
- [Revision History](#revision-history)



## Preface

### Intention

This article is an attempt to further understand and document the AmigaOS 32bit PPC ABI (shorten with `pcc-amigaos`); inspired, in no small part, due to a lack of existing documentation that is publicly available. There is information scattered around the internet in forums and porting progress, but, they describe just a tiny bit and it is hard to get the whole picture from the source. This document tries to give a complete picture of the specific needs of `pcc-amigaos` as single source of documentation.

Special thanks go to the following members of various forums for giving motivation and helpful insight information (in no particular order):

* afxgroup
* elfpipe
* futura
* joerg
* kas1e
* rjd324
* walkero
* and many more, which I forget to mention (sorry)

## Introduction

To understand the Application Binary Interface used by `ppc-amigaos` other documents also need to be considered:

The System V Application Binary Interface, or System V ABI, defines a system interface for compiled application programs. Its purpose is to establish a standard binary interface for application programs on systems that implement the interfaces defined in the System V Interface Definition, Issue 3. This includes systems that have implemented UNIX System V Release 4.

The System V Application Binary Interface e500 Processor Supplement (e500 Processor ABI Supplement) described in [PowerPC(tm) e500 Application Binary Interface User's Guide](https://www.nxp.com/docs/en/reference-manual/E500ABIUG.pdf) ([repo copy](https://github.com/AmigaLabs/binutils-gdb/wiki/E500ABIUG.pdf)) is a supplement to the generic System V ABI and contains information specific to System V implementations built on the e500 Architecture. The generic System V ABI and this supplement together constitute a complete System V Application Binary Interface specification for systems that implement the e500 architecture of the e500 processor family.

The [Power Architecture(tm) 32.bit Application Binary Interface Supplement 1.0 - Linux(r) & Embbedded](https://example61560.files.wordpress.com/2016/11/powerpc_abi.pdf) ([repo copy](https://github.com/AmigaLabs/binutils-gdb/wiki/powerpc_abi.pdf)) is the processor-specific supplement for use with ELF on 32-bit Power Architecture processor systems.

This document is the AmigaOS specific supplement for use with ELF on 32-bit big-endian Power Architecture process systems.

## Object Files

### ELF Header

The `e_ident[EI_OSABI]` member of the ELF Header **cannot** identify the target operation system ABI, because `ppc-amigaos` has not been assigned a dedicated value. Thus, this value is ignored to identify an object file for `ppc-amigaos`. Instead, a marker symbol is used. See chapter "Marker Symbols":#Marker-Symbols for further information.

As know, currently the dynamic linker does ignore this field and the marker symbol. It just assumes the loaded object file is for the `ppc-amigaos` target. Hence the marker and _targetOS_ field is just used by a linker and its tool.

The `entry` member of the ELF Header has a fixed start address of **0x01000000**. Even that it is not needed for `ppc-amigaos`, because all programs run in a shared address model. Thus, during loading the program always gets relocated. So, the start address will never fit. Instead, the dynamic linker uses the `_start` symbol to find the entry point from where the process starts executing.

#### Additional Information

The start address has not always been **0x01000000**. Till the SDK 53.3 (out in 2008), the start address had the value **0x00000000**. Then when shared objects (.so) support were added in said SDK 53.3, it were necessary to change start address to anything else, since these did add a .plt section, potentially in front of the .text (which means 0 as text base address was no longer working). So while start address is a placeholder, it shouldn't be **0x00000000** at least.

That change of the start addresses to be non-zero had unexpected side effects of the profiling executable: Profiling is a complex task, and diverse binaries, API, and so on must support it. And while current C libraries for OS4 such as CLIB2 and NewLIB had profiling code supporting "gprof" (by generating gmon.out and using performance.monitor feature of kernel) , code were hard-coded and expected that the start address was always zero. Thus profiling were broken for a long time, until it were rewritten for new CLIB4, to not depending on hard-coded values of the start address.

 
#### References:

 * [Comment #47 of thread 'Is gprof ever works on os4 ? It is! And can be still!'](https://www.amigans.net/modules/newbb/viewtopic.php?post_id=128298#forumpost128298)

### Special Sections

Normally the *.rodata* section is directly placed in the same segment after where *.text* and optional *.plt* are. For shared object (sobjs) targeting `ppc-amigaos`
this is not acceptable. Any segment containing the *.rodata* section will be treated differently, than other segments. Instead of loading the complete the segment
into a single memory block, each section is individually loaded into separate memory blocks. This will break *.plt*if the section is in the same segment as .*rodata*. 
Thus it is desired to place the *.rodata* section a segment of its own so that the current available dynamic linker (elf.library V53.30) can load the executable, without
resulting in crash during execution. The *.rodata* section can have additional section in its segment, as long the segment is flagged as read only and not executable.

The reason for this seems to be the support of 68k cross calls from/to the sobj. It seems that the *.rodata* section contains 68k code.

#### References:

 * [Comment #389 ff. of thread 'gcc 9 and 10'](https://www.amigans.net/modules/newbb/viewtopic.php?post_id=144308#forumpost144308)


### Symbol Table

#### Marker Symbols

There are two marker symbols used. One for executable, object files and the other for sobjs.

**Executable, object files**

The marker symbol `__amigaos4__` for executable is necessary because there is no distinct value defined for identifying the `ppc-amigaos` target in the [ELF Header](#ELF-Header). I even think this is ignored by the dynamic linker, this marker symbol is required by the linker, so that the linker can identify the target from the file. Thus `__amigaos4__` symbol must be protected against stripping.

**Sobjs**

The marker symbol `DT_AMIGAOS_DYNVERSION` for sobjs files has a twofold functionality. First it is used to identify that the sobj file targets `ppc-amigaos`, because of the same reason as the `__amigaos4__` symbol for executable. A sobj file might even have an `__amigaos4__` symbol, to make sure that it is targeting `ppc-amigaos`, but per se the `__amigaos4__` is not needed for sobj files. The other proposes of the `DT_AMIGAOS_DYNVERSION` symbol is to tell the dynamic linker to which version of the sobj support for `ppc-amigaos` is adheres to. Currently it is version 2, the previous version was only used during development and isn't used today in real life systems.

#### Entry Symbol

The symbol `_start` symbol is used to determine the entry point of the executable. That is needed because on `ppc-amigaos`, all executable shares the same address space, thus the assumed entry point from the ELF header might be taken by another executable. At the same time this means that all executables are relocated during the loading by the dynamic linker.  Furthermore, the strip command isn't allowed to strip relocations and symbols which are needed by `ppc-amigaos`.

#### Small Data Model Symbol

The symbol `_SDA_BASE_` defines the small data area base. An address relative to which all data is accessed with 16bit offset. The address is stored in register r13. This symbol is only needed if the executable is compiled to use the small data model, which is enabled with `-msdata` for the gcc. It has the limitation that the data must fit within 64k. On the `ppc-amigaos` target even exits another addressing schema, named shortened baserel, which works in a similar way like the small data model, but doesn't have the limitation of that data must fit within 64k. Probably the small data model even work with sobjs file, and the baserel model might not. 
`_SDA_BASE_` should be defined by the linker script, so there shouldn't be any problem. Indeed, you don't need it when there are no small-data addressing modes in the whole executable. But usually you have that somewhere, for example in the startup-code, which always sets up small-data even when the main program doesn't use it.

#### PPC marker

There is even the symbol `__amigappc_` used. But I think this is a relict and can be removed, because that PPC is the target can easily be detected by the ELF Header

#### Additional information

Because strip by default removes all symbols from executables and object files, the marker symbols for these kinds of files must survive a strip, else they cannot be processed further and/or not started by the dynamic linker. This requirement isn't needed for sobjs, because the default strip don't remove symbols from sobjs files. 

**TODO:** Verify that this is correct???

### Relocation Types

#### PPC-AmigaOS Specific Relocations

For the `ppc-amigaos` specific address model baserel, target specific relocation are needed, which are based upon the `.data` section, see [Base Relative Address Model](#Base-Relative-Address-Model) fro more information:

`BREL relocation = Symbol + Addend - .data`

| Name                  | Value | FIELD    | Calculation of Addend                              |
| :-------------------- | :---- | :------- | :------------------------------------------------- |
| R_PPC_AMIGAOS_BREL    | `210` | `word32` | `val`                                              |
| R_PPC_AMIGAOS_BREL_LO | `211` | `half16` | `val & 0xFFFF`                                     |
| R_PPC_AMIGAOS_BREL_HI | `212` | `half16` | `(val >> 16) & 0xFFFF`                             |
| R_PPC_AMIGAOS_BREL_HA | `213` | `half16` | `{(val >> 16) & 0xFFFF) + {{val & 0x8000) >> 15 )` |

Surprisingly it uses the base address of the `.data` section and not the symbol `_DATA_BASE_`, which should be identical.
There is also a function in elf.library (CopyDataSegment()), which can be used to make new copies for BREL-addressing. This was meant to be useful in reentrant programs or shared libraries.

## Target specific

### Base Relative Address Model

Like the small data address model but uses the register `r2` to store the base address, and uses 32-bit offsets. The offsets are calculated relative to the start of the data segment, or symbol `_DATA_BASE_`.
It seems that this address model is broken, because the register `r2` has been abused for other features, which interferes with the base relative address model. Rumors are curculating that `r2` has been used for tasks stuff and/or sobjs support.

### Specific strip operation

The strip operations provided by binutils ld linker, like `--strip-all`, does remove all symbols from the output file. As mentioned in the chapter [Marker Symbols](#Marker-Symbols) this needs to adjust for the `ppc-amigaos` target. The mention symbols from the chapter must survive a strip run. Additional during a `--strip-all` run undefined symbols are kept too.

Furthermore binutils ld provides a `ppc-amigaos` target specific strip operation named `--strip_unneeded_rel_relocs`. This option can be explicitly called, which requires a `--strip-all` (adjusted for `ppc-amigaos`) run. If enabled as the name states, it removes unneeded relative relocations. Disabled by default.

From a comment:

> If the symbol refers to a stripped section, we still want to keep it, e.g., `_SDA_BASE_` 

**TODO:** We should perhaps output a warning or add another option to trigger this behavior.
**FIXME:** The section to which symbol refers must be adjusted as well.

#### Additional Information

The other present linker for `ppc-amigaos` is (vlink)[http://sun.hasenbraten.de/vlink/], which offers the follwoing strip operations:

```
-S strip all debugger symbols
-s strip all symbols (except protected ones)
-X strip local symbols that start with the letters 'L' or 'l', or with a dot
-x strip all local symbols
```

#### References:

 * [Comment #304 of thread 'gcc 9 and 10'](https://www.amigans.net/modules/newbb/viewtopic.php?post_id=142371#forumpost142371)

## Other Oddities

### Pointer Equality

The pointer equality between executables and sobjs is not working with current public dynamic linker (elf.library v53.30). The case for that is twofold. Firstly, the binutils, for the `ppc-amigaos` target <= v2.40, explicitly hinder this with the modification that the value of the function pointer symbol is not allowed to be zero. It must have a value calculated by the linker. Secondly, the dynamic linker cannot handle function pointer symbol having a value zero. It just uses the value found in the symbol entry. And a value of zero leads to a crash. Using a found non-zero value follows the ABI. But not being able to handle a zero value don't comply with the ABI. A zero value indicates that the dynamic linker must resolve the function pointer. Doing so by the dynamic linker guaranties that function pointer values are equal between executables and sobjs files.
The newer version on binutils and elf.library will comply more with the ABI and thus offer pointer equality.

### Keep Unused Section Symbol

The define `TARGET_KEEP_UNUSED_SECTION_SYMBOL` has been introduced by the new binutils and is defined to be `true`. The comment above states the line in `elf32-ppc.c:25` that it used by the Linux kernel and tells the assembler to generated even symbols which seems unused. If enabled it works for the `ppc-amigaos` target, but I don't understand if it has possible side effects.

### GOT Is Marked as Executable

The GOT is marked to be executable for the `ppc-amigaos` target. That is in common with the `ppc-vxworks` target, thus something which even is required by other targets. The explanation for the `pcc-vxworks` target is that the GOT can contained the `blrl` instruction. The only difference to `pcc-vxworks` target is, that its additional flag the section with `SEC_CODE`. Which somehow makes sense. I don't know if that is really the same reason for `ppc-amigaos` having it that way, or if there is another reason to have executable.

Frank Wille confirms that the GOT contains `blrl` instructions, but he isn't sure why, or whether it is ever called.

### Dynamic Sections Are Read-Only

If the target is `ppc-amigaos` all dynamic section in the out file are flagged being read only. It probably wouldn't hurt to make it read-only, as there is no reason to write to it. But writing to it would be an error anyway. It doesn't matter that much. NetBSD defines it writable.

### Eliminate Copy Relocations

Copy relocations are needed for the following scenario:
An executable is linked with a shared object and accesses (which also means writing!) data variables from this shared object. It cannot modify this data directly, because there will be conflicts with multiple processes linked to the same shared object in memory, all manipulating the shared object's data.

The technical solution is:

The linker allocates space in the executable's .bss (or .sbss) section for a copy of that data. It also generates an `R_COPY` relocation, so that initialized data is copied (by the dynamic linker) from the shared object on startup into the executable's .bss space.

For the target `ppc-amigaos` the elimination of copy relocations is disabled, to support that use case. It probably not wide used, or even used at all. So theoretically it could be enabled.

### Constructor List

The `ldctor_build_sets` method in `ldctor.c:198` is used by the linker to build the constructor list. For the target `ppc-amigaos` it is always performed to build the list, even for already defined symbols. Which might happen if it is called from collect, and thus the sets may already have been built.

Constructors must be relocatable for the `ppc-amigaos` target.

### Default linking

The default for linking for target `ppc-amigaos` is to link statically. Additional even relocatable files are marked as being executable, not just executables.


## Revision History

| Version | Date       | Author         | Comment                                                                                                                                                                                                                                                                         |
| :------ | :--------- | :------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.0     | 2023-10-20 | migthymax      | Initial approach to document the PPC 32bit AmigaOS ABI                                                                                                                                                                                                                          |
| 1.1     | 2023-10-29 | FlynnTheAvatar | Small typo and grammar fixes                                                                                                                                                                                                                                                    |
| 1.2     | 2023-10-29 | rjd324         | Small typo and grammar fixes                                                                                                                                                                                                                                                    |
| 1.3     | 2023-10-30 | migthymax      | Added local copies of external linked PDF                                                                                                                                                                                                                                       |
| 1.4     | 2023-10-31 | kas1e          | Updated part about start address                                                                                                                                                                                                                                                |
| 1.5     | 2023-11-03 | migthmax       | Updated BREL relocation values upon frank (vbcc) comments                                                                                                                                                                                                                       |
| 1.6     | 2023-11-05 | migthmax       | Updated Table of content, Small data model From frank (vbcc) comments:<br>- corrected targetOS meber to e_ident[EI_OSABI<br>- added information about `__amigaos4__` symbol<br>- added information about `_SDA_BASE_` symbol<br>- added information about Baserel & Relocations |
| 1.7     | 2023-12-25 | migthmax       | Updated Special Sections chapter regarding _.rodata_ section with information from Joerg and Oliver                                                                                                                                                                             |
| 1.8     | 2024-01-07 | migthmax       | Clarified some information with information from Frank Wille and fixed internal links                                                                                                                                                                                           |
