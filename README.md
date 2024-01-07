## GCC6809 User Manual

This is the user manual for the GCC port to the Motorola 6809 processor. It contains detailed information about how to set up and use the compiler and auxiliary tools.

This guide may not always reflect the latest version of GCC6809. Check the GCC6809 [homepage](https://web.archive.org/web/20090131183455/http://www.oddchange.com/gcc6809/) for additional information.

A description of the simulator is found [here](/SimReadMe.md).

### Installation

GCC6809 is now distributed as a patch against a specific version of the GCC sources. You’ll need to obtain a GCC source tarball, unpack it, then apply the patch. In order to build the compiler, you’ll of course need a working native compiler.

The sources can be obtained from ftp.gnu.org or any GNU [mirror](https://ftp.mirrorservice.org/sites/sourceware.org/pub/gcc/releases/gcc-4.6.4/). Look for a file named `gcc-4.6.4.tar.bz2` or similar. For convenience this file is in the `gcc` folder.

1. Extract the contents (using `gunzip` or `bunzip2`). For example, `bzcat gcc-4.6.4.tar.bz2 | tar xvf –`. This will create a new directory named `gcc-4.6.4` which has all of the pristine sources.

2.Apply the 6809 patch using the ‘patch’ command. You need to be in the newly created directory, and you need to use the `-p1` option to patch. For example: `cd gcc-4.6.4 && patch -p1 < ../gcc-4.6.4.patch`. You’ll see a few messages saying that some files are added/modified.

3. Fix up permissions on a few files. The files in binutils named `ar`, `as`, and `ld` all need to be executable. Use `chmod a+x` on each of them to fix this. Working-out how to do this automatically.

At this point, the sources are ready to build… see below for more.

GCC6809 only runs under GNU or POSIX style environments; notably, it will not work under Microsoft Windows.

Remember that gcc per se doesn’t include an assembler, linker, or any user libraries. The assembler and linker are provided along with the compiler tarball (they are in the `as-4.1.0` subdirectory and are a copy of the popular `asxxxx` cross-assembler). Building `gcc` can build and install these tools as well.

Libraries are not addressed here; it is possible to use GCC6809 with any libraries at all and provide all of your own support routines. A port of the newlib library is available for those who need one, although it is not well tested.

### Patch Contents

A top-level directory named `build-6809` contains some helper scripts for building and testing the compiler. If you’re familiar with building cross-compilers, you can safely ignore the files in this directory; however, building the compiler from here will be easier for most people. From here, you can just enter `make` (with optional arguments) to build everything. See “Building GCC” below for the available options.

When you build this way, another subdirectory named `m6809-unknown-none` (or similar) will be created; this is where all of the build artifacts will appear.

`as-4.1.0` contains the assembler, linker, and library manager. `binutils` contains the scripts that act as a frontend to them, using standard GNU syntax. For example, `aslink` is the linker but we provide a `ld` script which invokes it and translates command-line options to the GNU way. Not all options are supported, though.

The actual compiler files for the 6809 are all located under `gcc/config/m6809`. There are a few modifications to other files in the GCC distribution as well.

### About the Assembler/Linker

Three programs are built from the astools package in the `as-4.1.0` directory:

* `as6809` - the assembler
* `aslink` - the linker
* `aslib` - the library manager

You can build just these tools using the `make asm` command in the build directory.

### Building ‘binutils’

The frontend scripts in `binutils` are just shell scripts that parse and re-package the command-line options before calling the astools. Scripts are provided named `as`, `ld`, and `ar`.

Before building GCC, these scripts need to be installed in the same place that GCC will eventually go. This can be done by running `make binutils`. Remember to make sure that these files are executable.

### Building gcc

The helper Makefile can be used to configure, build, and install all of GCC in one step, just by typing `make`.

The Makefile will default to the correct version of GCC, although it can be fooled by defining `GCC_VERSION` in your environment prior to running make. This is an artifact of the older distribution method and should not be needed anymore.

By default, it will configure a default, C-only compiler build that will generate code for any 6809-based hardware. You can change the language frontends to be built by setting `GCC_LANGUAGES` in your environment before running `make`. You can also target a specific piece of supported harware using the `GCC_TARGET` variable.

When running on Linux, the build requires root privileges in order to install into `/usr/local` (or wherever you configured it to be installed). The makefile will normally attempt to run `sudo` before this step, except on host platforms that don’t support it.

The main files installed by GCC are:

* `/usr/local/m6809/bin/gcc` which is the frontend driver to the compiler
* `/usr/local/libexec/gcc/m6809/version/cc1` which is the actual compiler
* `/usr/local/libexec/gcc/m6809/version/collect2` a frontend to the linker
* `/usr/local/lib/gcc/m6809/version/libgcc.a` a library of helper functions that is used for certain complex expressions that don’t translate easily to 6809 instructions. 

The GCC6809 package also creates a copy of the gcc frontend driver named `gcc-`. This makes it easy to keep several different versions of gcc at once.

### Other `make` commands

Running just `make` will build and install the compiler, or pick up from where it left off if you interrupted it or changed anything. It will only rebuild what it has to.

The helper Makefile also supports the following commands:

`clean` : clean all of the object files, but not the output from configure
`distclean` : delete everything in src and build; this will force a complete rebuild
`everything`: clean, rebuild, and reinstall everything, including the assembler tools, the frontend scripts, and the compiler. This is the easiest way to build everything, although it takes a little longer.
`test` : run the GCC test suite. This requires that you downloaded a testsuite files and placed them in gcc-release. The environment variables that you can set before invoking make are:

```
AS_VERSION
BUILD_DEBUG_CFLAGS
GCC_LANGUAGES
GCC_VERSION
prefix
target_vendor
target_os
Data Types
```
An `int` is 16-bits, or 2-bytes wide. `short` or `char` can be used to refer to an 8-bit quantity. A plain `char` is unsigned.

A `long` is 32-bits, or 4-bytes wide. There is no support for `long long`.

Optionally, you can make integers 8-bits wide, by using the `-mint8` command-line option. This also shortens the size of `long` to 16-bits. It does not affect `short` or `char`. It is strongly recommended that you don’t do this unless you know what you are doing. When using this option, you can’t rely on the C library to work correctly, because the C standard explicitly does not permit this behavior. However, you can get more optimal code generation with it.

All pointers are 16-bits wide. A pointer is *not* necessarily the same size as `int`, if you use `-mint8`.

Floating point is supported although it is not thoroughly tested outside of the normal GCC testsuite. FP performance is weak because 32-bit math is weak.

### Calling Conventions

#### Arguments

GCC6809 supports several different calling conventions. You can select one via the `-mabi_version` option.

The default convention, called regs, will place the first 8-bit argument in the `B` register, the first 16-bit argument in the `X` register, and all other arguments on the stack in reverse order.

An alternate method, called stack, places all arguments on the stack. Note: a function that accepts a variable number of arguments is always treated this way, in any mode. This was the default in very early GCC6809 incarnations and can be specified via `-mabi_version=stack`.

No form of promotion is used on function arguments, as none is necessary.

The reason that `X` is used for 16-bits is that most 16-bit values tend to be pointers instead of simple data values. Using `X` right away allows access to all of the addressing modes on the 6809. Earlier versions of the compiler used `D`, and the generated code contained lots of `tfr d,x` instructions.

#### Return Values

An 8-bit return value is placed in B; a 16-bit return value is placed in `X`. Larger values, including structures, are returned by having the caller pass in a pointer to the location where the result would be copied.

#### Clobbering

The `B` and `X` registers are assumed to be clobbered by any function calls, since return values and argument values may be placed there.

The `Y` and `U` registers are assumed to be preserved across a function call. Thus, if a function wants to use those registers, it must be save/restore them internally. GCC does this automatically for you.

If you are writing assembly language macros inside C functions, you should understand this behavior. Do not assume that a value in registers D or X has the same value after making any function call. If you depend on the value of the `CC` or `DP` registers, you must deal with that explicitly.

#### Registers

The 6809 has a fairly small register set. This makes it tough for GCC to do a good job, because its internal algorithms assume the availability of plenty of registers. Very rarely, some complex expressions may cause the compiler to abort.

GCC does not use the `A` accumulator for temporaries. GCC treats `D` as a generic, 16-bit register, but renames it to `B` when only 8-bits of it are relevant. Even when no 16-bit math is required, GCC still cannot use `A` as a separate register from `B`. This is a limitation which, if removed, would bring some nice performance enhancements. The `A` register is used internally for some canned instruction sequences, but it cannot serve as a general-purpose 8-bit register.

Note that newer versions of GCC6809 are a little smarter with regards to the `A/B` split.

You can use the `A` register in assembly macros. This is useful for time-critical code in which using the register is faster than using temporaries on the stack. You must be sure that GCC does not use the `D` register while doing this, though.

The `U` and `Y` registers are used as general-purpose, 16-bit registers, and will be allocated for stack variables. Note that 8-bit local variables cannot be assigned to these variables. In some cases, you can get better performance for a local by extending its type (manually) from 8-bit to 16-bit to force allocation to one of these registers. `U` is preferred over `Y` since instructions using `Y` are all
one-byte longer, and thus take one cycle longer, too.

The `S` register refers to the program stack and is used to reference local variables, function arguments, and compiler-generated temporaries.

#### Soft Registers

If you have a need for lots of registers, due to complex calculations or large functions, GCC tends not to generate the best code. However, it can be improved by enabling “soft registers”. These are memory locations, accessible in the direct addressing mode, which appear to GCC as registers.

GCC6809 can use up to 4 soft registers; the memory locations are named `*m0` through `*m3`. Use the `-msoft-reg-count` option to enable their usage.

Most of the passes of GCC expect to have a large number of registers to work with. When it runs out, any remaining values must be “spilled” to the stack. This creates several problems in embedded environments. First, stack load/store instructions can be slow, because it requires indexed addressing. Second, systems may have little RAM. Third, in multiprocessing systems which need to save/restore the stack, keeping the stack smaller will produce faster code.

Soft registers help in all of these areas. Direct mode accesses are shorter in both length and execution time. Because they are global, and not relative to the stack frame, less overall memory is used. Finally, because GCC is unaware that these registers are actually in memory, it can still perform some optimizations on values in these locations which aren’t possible once values are spilled to the stack (just because of the way GCC is written).

Note that all soft registers are considered call-clobbered. GCC6809 will not save/restore any soft registers in the function prologue/epilogue, as it would be too slow. Consequently, values can’t be kept in soft registers across a function call.

#### Interrupts

Ordinary C functions can be used to implement interrupt handlers. These functions should be declared with the interrupt attribute to force emitting an `rti` instruction at function exit.

Because IRQ saves and restore all registers automatically, there is no need for GCC to do this on an IRQ handler. You should thus use the naked attribute on the IRQ function to optimize it. However, do not do this on the `FIRQ` handler.

The interrupt vector table can be declared as an ordinary structure of function pointers; use the section attribute to force them to be placed into a separate section, such as “vector”. This section should be mapped to address 0xFFF0 at link time. If you link against the simulator libraries, this is done automatically for you.

#### Command-Line Options

This section describes extra command-line arguments to the compiler that are specific to the 6809.

`-mint8`
Shorten `int` to just 8-bits. See above for full details. This option is not recommended.

`-mnodirect`
Inhibits placing variables in the direct section, even if they have been declared to be in the direct section. The default can be explicitly requested by using `-mdirect`.

`-mwpc`
Enables hardware acceleration on the WPC pinball platform. Currently, the only optimization being performed under WPC is to implement certain shift operations with the WPC hardware shift logic.

`-mfar-code-page=val`
Sets the code page for the module. When using the farcall feature, this tells the compiler which page that this module will be linked into. This is only used to optimize far calls to functions in the same code page, in which a farcall thunk is not required.

`-mcode-section=name`
Sets the name of the area/section to be used for code. This value can be overriden by declaring the function with a section attribute. The default value is `.text`.

`-mdata-section=name`
Sets the name of the area/section to be used for initialized data. This value can be overriden by declaring the variable with a section attribute. The default value is `.data`.

`-mbss-section=name`
Sets the name of the area/section to be used for uninitialized data. This value can be overriden by declaring the variable with a section attribute. The default value is `.bss`.

`-mabi-version=name`
Sets the calling convention to use (see above).

`-msoft-reg-count=name`
Sets the number of soft registers to use. The default is 0.

`-m6309`
Enables 6309 instructions. Not fully implemented.

#### Attributes

Attributes are a GCC extension that allows declarations to be annotated with additional properties outside of the C language specification. GCC 6809 defines some new attributes for features that are helpful on 6809 machines.

To apply an attribute to a function declaration/definition, use the following syntax:

`__attribute__((attr_name)) void f (...);`
If an attribute takes parameters, then use the following form:

`__attribute__((attr_name(arg))) void f (...);`

* `section`
Declares that a function or variable resides in a different section. This attribute takes a string argument, which is the name of the section.

By default, declarations are placed into section names that correspond to the default sections that the assembler understands. Use of a section attribute causes an `.area` directive to be emitted prior to the declaration to place it in a new section.

The section name `direct` is special. This indicates that the section will be linked into the zero page area, which can be accessed using shorter, faster instructions. It is up to you to make sure that the linker puts direct in the right place, and that the direct-page register `DP` is loaded correctly for this to work. Direct references are denoted in the assembly generated by using an asterisk in front of the object name.

* `far`
Denotes a function declaration (i.e. prototype) that should be called using the far calling convention.

Since the 6809 only provides a 64KB total address space, many 6809 implementations use some form of bank switching to allow for more code than fits into the address space at once. Each machine defines its own way of performing the bank switch.

To declare a function that lives in a bank other than the default, use the following syntax:

`__attribute__((far(page_value))) void f(...);`

Note: this attribute only belongs on the declaration (in a header file) and not the definition (in a C file). The generated code for the function itself is not any different; the differences are needed when other entities need to call this function.

`page_value` is a string that represents an 8-bit value. It can be a literal value, or a symbolic name that the assembler will understand. Normally you want to place a simple number here. This value somehow reflects the value of the bank switching register,
but GCC does not define its meaning.

When a function declared as `far` is called, GCC emits different code than just a simple jsr. Instead it emits the following:

```
jsr __far_call_handler
.dw #function_name
.db #function_page
```

`function_name` is the ordinary name of the function. `function_page` is the page value given in the function declaration’s `far`
attribute.

The `far` call handler is machine-specific and is responsible for figuring out how to call the function. It should save the current bank register, change it to the new value based on the page value, call the function, and then restore it to the previous value. It must also ensure that the registers on input to the actual subroutine are exactly what was in the registers prior to the `far` call handler being invoked. Actually, only the `B` and `X` registers need be correct, as `Y` and `U` are not live on function entry; however, those must be saved/restored if they are used within the far call handler itself.

Notice that the function name and page are located in memory immediately after the call to the handler. The handler must update the return address to return beyond those 3 bytes.

It is recommended that the far call handler be written in assembler, due to all of the strange requirements.

GCC6809 does not provide a default far call handler. Here is the one used on the FreeWPC platform as an example:

```
.area .bss
__far_call_address:
      .blkb 2

.area .text
.globl __far_call_handler
__far_call_handler:
pshs  b,u,x                 ; Save all registers used for parameters
ldu   5,s                   ; Get pointer to the parameters
ldx   ,u++                  ; Get the called function offset
ldb   ,u+                   ; Get the called function page
stu   5,s                   ; Update return address
stx   *__far_call_address   ; Move function offset to memory
lda   IO_BANK_REGISTER      ; Read current bank register value
stb   IO_BANK_REGISTER      ; Set new bank register value
puls  b,u,x                 ; Restore parameters
pshs  a                     ; Save bank switch value to be restored
jsr   [$__far_call_address]  ; Call function
puls  a                     ; Restore A
sta   IO_BANK_REGISTER      ; Restore bank register
```

Use the `-mfar-code-page` option to tell GCC what page the current module will be mapped to. A `far` call within the same code page (i.e. caller and callee are near each other) can be optimized to a normal call. This is optional, though.

The far call mechanism will fail to work with any function that requires stack arguments, since the farcall handler needs to use the stack for saving/restoring the bank number. GCC will give an error if you try to declare a function far when it would not work.

Finally, no support is given for helping with pointers-to-far-functions.

* `interrupt`
Specifies that a function is the entry point for an interrupt handler. On the 6809, this applies to both the `IRQ` and `FIRQ` vectors.

This attribute causes the `rts` instruction to be replaced with an `rti` instruction. It also prevents use of instructions such as `puls x,pc`.

* `naked`
Disable emitting prologue/epilogue code for the function. This causes no stack space to be allocated for local variables, and no registers to be saved/restored. Use only when necessary.

* `noreturn`
This is a standard GCC attribute. The 6809 backend understands this attribute and will emit `jmp` instructions instead of `jsr` when calling functions that don’t return.

* `Builtins`
Some 6809-specific opcodes can be accessed through C-like macros or functions. These are sometimes called intrinsics in other compilers.

```
swi – use __builtin_swi()
swi2 – use __builtin_swi2()
swi3 – use __builtin_swi3()
cwai – use __builtin_cwai()
sync – use __builtin_sync()
```