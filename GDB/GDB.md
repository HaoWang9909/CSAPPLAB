# GDB

## `Learning Materials`

---

- Learning Materials
    - Material 1
      
        https://www.sourceware.org/gdb/
        
    - Material 2
      
        https://nasdaqgodzilla.github.io/2022/04/22/GDB教程-命令速查手册/
        

### Cue

- What is GDB and what is its primary purpose?
- What operating systems and programming languages does GDB support?
- How do you install GDB on Ubuntu?
- What are some essential GDB commands and their functions?
- How does GDB help in debugging programs that have crashed?

### Notes

---

# Introduction to GDB

## Overview

- **Website**: [GDB Official Site](https://www.sourceware.org/gdb/)
- **Definition**: GDB, the GNU Project debugger, is a tool that allows developers to inspect what a program is doing while it executes, or to analyze what a program was doing at the moment it crashed.
- **Functionality**: GDB helps in debugging by allowing you to:
    - Start a program under specific conditions.
    - Pause the execution based on specified conditions.
    - Inspect the state of a program after it has stopped.
    - Modify the program state to test different fixes.
- **Execution Environment**: It works with programs running natively on the same machine, on a remote machine, or on a simulator. GDB is compatible with most UNIX systems, Microsoft Windows, and macOS.

## Language Support

GDB supports debugging for a variety of programming languages, including:

- Ada, Assembly, C, C++, D, Fortran, Go, Objective-C, OpenCL, Modula-2, Pascal, Rust.

## Installation

To install GDB on an Ubuntu system, use the following commands:

```bash
sudo apt update
sudo apt install gdb

```

To verify the installation, check the GDB version:

```bash
gdb --version

```

## Basic Usage

### Compilation for Debugging

To debug effectively with GDB, compile your C programs with the `-g` option to include debugging information:

```bash
gcc -g test.c -o test

```

### Starting GDB

```bash
gdb ./test

```

### Common GDB Commands

| Command | Description |
| --- | --- |
| `layout next` | ***find the best layout*** |
| `r` | Run the program |
| `quit` | Exit GDB |
| `man gdb` | Open the manual for GDB |
| `b` | make a break point (use *0x to break at the line address) |
| `list` | List source code |
| `n` | Execute next program line |
| `d` | Delete a breakpoint |
| `p` | Print variable or expression |
| `step` | Step into a function |
| `disas` | disassemble |
| `finish` | if you enter a function, jump out of it |
| `display/5i $pc` | show 5 lines of the program right now |
| `p **(int**)($rsp+0x8)` |  |
| `x/nfu address` | exmaine |

### Practical Example

Setting and viewing breakpoints:

```bash
b main   # Set a breakpoint at the start of main function
b 9      # Set a breakpoint at line 9
info b   # Display all breakpoints

```

### Advanced Features

- **Logging**: Enable logging to capture session data.
  
    ```bash
    set logging on
    
    ```
    
- **Watchpoints**: Monitor changes to variables.
  
    ```bash
    watch <variable_name>
    
    ```
    

### Debugging Core Dumps

When a program crashes, it might generate a core dump. To debug with a core dump, use:

```bash
gdb -c [core dump file] [executable]

```

## Example Code

Here's a simple C program example for testing with GDB:

```cpp
#include <stdio.h>

int main() {
    int arr[] = {1, 2, 3, 4};
    int size = sizeof(arr) / sizeof(arr[0]);

    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\\n");

    return 0;
}

```

## GDB a running program

```bash
# Attach GDB to a running process
gdb -p [process ID]
```

### Summary

- GDB is a versatile tool used by developers to debug programs by allowing them to run in a controlled testing environment where they can stop, inspect, modify, and manipulate the program execution in real-time. It supports a wide range of programming languages and runs on most operating systems, making it a critical tool for software development and troubleshooting. Key functionalities include setting breakpoints, watching variables, stepping through code, and analyzing core dumps to solve and understand bugs in programs.

---