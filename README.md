# Getting gdb56300 to work:

First assemble your program with the -g flag to include debug information.

gdb56300 crashes with wine, but it runs okay if you run it with the winedebugger ```winedbg```

Simple wrapper can be used to get it to run.

```
justin@Z390:/usr/local/sbin$ cat gdb56300
#!/bin/bash
echo "continue" | winedbg "c:\Symphony-Studio\dsp56720-devtools\dist\gdb\gdb56300.exe"
```

You can load symbols from the assembled ".cld" program
Use the full path
```
(gdb) file /home/justin/dev/asm/CS4218/ECHO.cld
Reading symbols from z:\home\justin\dev\asm\cs4218\ECHO.cld...done.
```

```
(gdb) info sources
Source files for which symbols have been read in:

ada_init.asm, ECHO.asm, vectors.asm

Source files for which symbols will be read in on demand:


(gdb) info functions
All defined functions:

Non-debugging symbols:
0x00000188  echo
(gdb)
````


Get gdb to connect to openocd like so:

```
(gdb) target extended-remote localhost:3333
Remote debugging using localhost:3333
```

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
(gdb) continue
Continuing....
```

find location in code...
```
(gdb) where
#0 0x00000107 in endloop() at 56303_test.asm:43
```

A useful gdb command that has been created specifically for Symphony DSPs is a modify memory
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
