# Arm Toolchain: Map Files

## Basic Functionality
Map files are created by a directive to the linker.

```
arm-none-eabi-ld main.o -T linker.ld --cref -Map main.map -o main.elf
```

The output of the linking process is a Map file.  The notable elements in the map file are starting addresses, lengths of variables and sections, and symbols exported for startup code.

Below you can see the variables from the main.c file (global_constant => .rodata) and (global_variable => .bss) their addresses and lengths.

## Use Case

Locate the addresses for code and verify the section that code is allocated to.
For example, the vector table must be located at a specific address, the variable holding the table can be located in the map file.  From there, it will be known if the memory for the vector table must be relocated.

## Source Listing
``` C
#include<stdint.h>

const uint32_t global_constant = 3;

uint32_t global_variable = 0;

int main(void)
{
    uint32_t result = 0;
    global_variable = 10;
    for(uint32_t i = 0; i<global_constant; i++)
    {
    	global_variable += global_variable;
    }
    result = global_variable;
    return result;
}
```

## Map File Listing for Example Snippet
```

Linker script and memory map

LOAD main.o
                0x10002000        _stack_start = (ORIGIN (CCRAM) + LENGTH (CCRAM))

.text           0x08000000        0x48
 *(.isr_vector)
 *(.text)
 .text          0x08000000        0x44 main.o
                0x08000000        main
 *(.rodata)
 .rodata        0x08000044        0x4 main.o
                0x08000044        global_constant
                0x08000048        . = ALIGN (0x4)
                0x08000048        _end_text = .
                0x08000048        _la_data = LOADADDR (.data)

.glue_7         0x08000048        0x0
 .glue_7        0x08000048        0x0 linker stubs

.glue_7t        0x08000048        0x0
 .glue_7t       0x08000048        0x0 linker stubs

.vfp11_veneer   0x08000048        0x0
 .vfp11_veneer  0x08000048        0x0 linker stubs

.v4_bx          0x08000048        0x0
 .v4_bx         0x08000048        0x0 linker stubs

.iplt           0x08000048        0x0
 .iplt          0x08000048        0x0 main.o

.rel.dyn        0x08000048        0x0
 .rel.iplt      0x08000048        0x0 main.o

.data           0x20000000        0x0 load address 0x08000048
                0x20000000        _start_data = .
 *(.data)
 .data          0x20000000        0x0 main.o
                0x20000000        _end_data = .
                0x20000000        . = ALIGN (0x4)

.igot.plt       0x20000000        0x0 load address 0x08000048
 .igot.plt      0x20000000        0x0 main.o

.bss            0x20000000        0x4 load address 0x08000048
                0x20000000        _start_bss = .
 *(.bss)
 .bss           0x20000000        0x4 main.o
                0x20000000        global_variable
                0x20000004        . = ALIGN (0x4)
                0x20000004        _end_bss = .
OUTPUT(main.elf elf32-littlearm)
LOAD linker stubs

.debug_info     0x00000000       0xe9
 .debug_info    0x00000000       0xe9 main.o

.debug_abbrev   0x00000000       0x9a
 .debug_abbrev  0x00000000       0x9a main.o

.debug_aranges  0x00000000       0x20
 .debug_aranges
                0x00000000       0x20 main.o

.debug_line     0x00000000       0x165
 .debug_line    0x00000000       0x165 main.o

.debug_str      0x00000000       0x10e
 .debug_str     0x00000000       0x10e main.o
                                 0x14b (size before relaxing)

.comment        0x00000000       0x45
 .comment       0x00000000       0x45 main.o
                                 0x46 (size before relaxing)

.ARM.attributes
                0x00000000       0x2e
 .ARM.attributes
                0x00000000       0x2e main.o

.debug_frame    0x00000000       0x38
 .debug_frame   0x00000000       0x38 main.o

Cross Reference Table

Symbol                           File
global_constant                  main.o
global_variable                  main.o
main                             main.o

```