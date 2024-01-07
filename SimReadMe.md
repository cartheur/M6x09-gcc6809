## 6809 Simulator

GCC6809 has been tested in a variety of ways. An important tool has been this 6809 simulator, which was originally written by Arto Salmi but has been modified and enhanced greatly to support 6809 development.

The simulator has not been online for some time, but it is so useful to me, I felt it was time to publish it again. Download the version 0.91 sources here.

The 6809 simulator has recently undergone a major rewrite in many areas. It is still being worked on heavily, and further releases are expected. However, feedback is welcome at this point for other ways to make it more useful to more people.

Additional details are described [here](https://github.com/cartheur/M6809-I-SBC).


### Features
Cycle accurate simulation of all 6809 opcodes, except for cwai and sync.
Builtin debugger similar to gdb which lets you step through code.
Capable of loading "linker map" files for symbolic debugging.
Designed to emulate a number of different machine architectures. Current machines range from simple, which has a single 64KB RAM device, to wpc, which emulates most of the WPC pinball ASIC. Adding support for other machines like CoCo or the like would be fairly easy.
Fairly efficient simulation that runs about 20x faster than realtime.
Supports machines that have bank switching hardware, where there is more addressable memory than can fit into 64KB at once.
A more detailed user manual will be written once the interface becomes more stable.

Compiling the Code
The simulator can compile on any UNIX-like platform, such as Linux or Cygwin for Windows. autoconf is used to create the build environment. The usual configure, make, make install sequence should work just fine.

The executable is named m6809-run.

SIMPLE
The Simple Architecture was created to allow for minimal testing of the compiler. The C library, newlib, assumes this architecture today. All executables are just S-record files that contains the full memory image of the program, including the vector table at address 0xFFF0. The simulator starts from the reset handler at 0xFFFE and proceeds from there.

Three special addresses are reserved for input, output, and exit:

Writing to 0xFF00 causes a character to be printed to standard output.
Reading from 0xFF02 causes a character to be read from standard input.
Writing to 0xFF01 causes the simulator to exit back to the UNIX shell, passing back the value written as the return code.
The C library already knows about these functions, if your target program calls getchar(), putchar(), or exit().