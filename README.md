# Getting gdb56300 to work:


gdb56300 is located in c:\Symphony-Studio\dsp56720-devtools\dist\gdb\

gdb56300 crashes/hangs with wine, but it runs okay if you run it with the winedebugger ```winedbg```

Simple wrapper can be used to get it to run.

```
justin@Z390:/usr/local/sbin$ cat gdb56300
#!/bin/bash
echo "continue" | winedbg "c:\Symphony-Studio\dsp56720-devtools\dist\gdb\gdb56300.exe"
```

First assemble your program with the -g flag to include debug information.

```
asm56300 -A -g -l MYPROG.lst -B MYPROG.asm
```

You can load symbols from the assembled ".cld" program
(Use the full path)
```
(gdb) file /home/justin/dev/asm/CS4218/MYPROG.cld
Reading symbols from z:\home\justin\dev\asm\cs4218\MYPROG.cld...done.
```

```
(gdb) info sources
Source files for which symbols have been read in:

ada_init.asm, MYPROG.asm, vectors.asm

Source files for which symbols will be read in on demand:


(gdb) info functions
All defined functions:

Non-debugging symbols:
0x00000188  echo
(gdb)
````
Edit: It appears asm56300 is not detecting symbols correctly, or at least not all of them. The ECHO.asm example it fails to acknowledge MAIN, begin etc...
See [1]

Get gdb to connect to openocd like so:

```
(gdb) target extended-remote localhost:3333
Remote debugging using localhost:3333
```
(Note: if you run file after connecting to openocd, the connection will be terminated, so use the order presented here)

monitor command can be used to send ocd commands

```
(gdb) monitor reset halt
halted: PC: 0x107
```

gdb "load" command can be used to upload the program through gdb via openocd. This appears to populate both the x and y memory regions correctly.

```
(gdb) load
Loading section .vectors, size 0x100 lma 0x20000000
Loading section .text, size 0x5a0 lma 0x20000100
Loading section .data, size 0x18 lma 0x200006a0
Start address 0x2000061c, load size 1720
Transfer rate: 22 KB/sec, 573 bytes/write.
```

continue command is used to continue program execution.
```
(gdb) continue
Continuing....
```

use ctrl + c to interrupt program execution and to resume gdb interaction.

```
Program received signal SIGINT, Interrupt.
0x00ff0025 in ?? ()
(gdb)
```

find location in code...
```
(gdb) where
#0 0x00000107 in endloop() at 56303_test.asm:43
```

A useful gdb command that has been created specifically for Symphony DSPs to modify memory
```
command M:
M <memspace>:<start_addr>[..<end_addr>] <data>
<memspace>: p | x | y | l
<start_addr>: start address, can be in hex by prefixing with 0x
<end_addr>: optionally end address to do a bulk memory modify of the same data word
<data>: The data word to write to memory, can be in hex by prefixing with 0x
```

Some debugging examples:

```
(gdb) break 56303_test.asm:40
(gdb) info break
Num Type           Disp Enb Address    What
1   breakpoint     keep y   0x00000101
(gdb)
(gdb) jump begin
Continuing at 0x101
(gdb) del break

(gdb) print $pc
$1 = 262
(gdb) print $r0
$2 = 279
(gdb) print $a0
$3 = 4421604
```

### [1] Symbols

gdb56300 appears to be ignoring certain symbols.
We can use the included cldlod.exe program to dump a 'LOD' format of the assembled cld file

The resulting output shows that the complete symbol table is intact.

ie: 
```
_SYMBOL P
vectors              I 000000
START                I 000100
main                 I 000100
sine_init            I 00010A
sine_loop_end        I 00012C
echo_loop            I 000132
ada_init             I 00013C
set_control          I 000153
```

# References:

Symphony-Studio windows installer.
https://www.nxp.com/products/processors-and-microcontrollers/additional-mpu-mcus-architectures/digital-signal-processors/symphony-studio-development-tools:SYMPH_STUDIO

User Manual:
https://www.nxp.com/docs/en/user-guide/DSPSTUDIOUG.pdf

dsp56k gcc + associated tools asm56300 cld*.exe etc
https://www-mipp.fnal.gov/TPC/DAQ/56KCCUM.pdf
