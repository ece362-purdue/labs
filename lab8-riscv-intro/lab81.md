# Lab 8.1: Introduction to Basic Assembly

| Step | Associated Steps (and checkoff criteria)                                             | Points  |
|------|--------------------------------------------------------------------------------------|---------|
| 1    | [2. Assembly instruction: arithmetic](#2-assembly-instruction-arithmetic)            | 20      |
| 2    | [3. Assembly instruction: bit-wise/logical](#3-assembly-instruction-bit-wiselogical) | 30      |
| 3    | [4. Assembly instruction: memory](#4-assembly-instruction-memory)                    | 50      |
|      | Total                                                                                | 100     |

## Introduction

In this lab, we will briefly discuss the process of program generation to understand how C source files and assembly source files get translated into program that a machine (or a bare-metal simulator like QEMU) could understand.

We will also walk through several types of assembly instructions used with 32-bit RISC-V, specifically from the [RISC-V 32-bit Instruction Set Architecture](https://riscv.org/technical/specifications/) (ISA). In addition, some assembly directives will also be discussed to let you be prepared for the next two labs.

The ISA used in the lab is same as the one covered in the book 'Computer Organization and Design (RISC-V Edition) (2nd Edition)'. Hence the book serves as the reference for the lab.

A reference manual for the RISC-V ISA can be found [here](https://riscv.org/technical/specifications/). Refer to the 'Unprivileged' version, i.e., Volume 1.

## 1. A tutorial on program generation

### 1.1 Intro

You might have written some programs in C or other languages like JavaScript or Python before, just like the one below:

```C
// HelloWorld.c
#include <stdio.h>

int main (int argc, char * argv[]) {
    printf("Hello World!");
    int x = 1 + 2;
    return 0;
}
```

With a click of button in an IDE or some terminal commands, Voila! The program just works. Nevertheless, how exactly does the CPU recognize and execute the program?

### 1.2 ISA and Compiler

As human beings, we are also "compilers" in a sense - we can read the source code above and know this is prints the string `Hello World!` to the screen.

Unfortunately, CPUs cannot understand English (until and unless someone writes a compiler for the English language), even a small set of the English language, such as a programming language.  Instead, what the CPU understands is machine language, which is just a set of bits that map to some operations in hardware.

The set of bits that the CPU understands is called the **Instruction Set Architecture (ISA)**.

For instance, in the 32-bit RISC-V ISA, or simply RISCV32 (the one we are using) `0000000 01011 01100 000 01101 0110011` gets decoded as a 32-bit addition of the values in registers `x11` and `x12`, and saves the sum into `x13`. Recognition of these bits is hard-wired on the CPU chip and are the machine language representation of the entire instruction set architecture.

However, the bit strings are just `1` and `0`, so how did we get those from the program we just wrote?
Well you might already know the answer: through compilers and assemblers like those used by `gcc` or `clang`.

The compiler is programmed to understand specific programming languages and translate programs written in them, to the assembly language corresponding to the hardware platform.  For instance, if we compile the example above in RISCV32, it might become something like this:

```asm
li x11, 1
li x12, 2
add x13, x11, x12
sw x13, -20(sp)
```

The step of generating assembly from a high level language is done by the compiler.  The assembly then needs to be translated into the 1's and 0's that correspond to machine code the computer understands, which is done by an assembler.  You can also try seeing the machine instructions for your personal computer with the command `gcc -S [C source file]` if you are using Linux. 

The type of assembly generated depends on the architecture of your host computer - it might generate Intel's x86 assembly instead of RISC-V, for example.  When you run just `gcc` on your own machine, you're generating x86 assembly, allowing your program to run on your own computer.  But it is also possible to **cross-compile**, i.e. you have a compiler that was compiled to run on x86 machines, but when it compiles programs, it does so for RISC-V machines.  The `riscv64-unknown-elf-gcc` command is an example of such a cross-compiler.

All of these details are generally hidden from the application developer who just relies on the compiler/assembler toolchain to generate binaries that the computer can understand.

> [!Note]
> If you do try it with a modern compiler, the `int x = 1 + 2` part will likely just become:
> ```li a5, 3 ```
> ```sw a5, -20(s0)```
> This is because the compiler is actually smart enough to evaluate the value for you, saving time for the actual execution!

Using the same environment you set up on `eceprog`, or a lab machine, you can use the following instruction to generate the assembly file for the C program we gave you above.

```bash
riscv64-unknown-elf-gcc -S HelloWorld.c
```

You will now observe that a HelloWorld.s file is generated when you run this command. This file is the assembly language file which is seen by the Assembler.

> [!Note]
> The reason why even though we are using RISC-V 32-bit ISA, we have compiler which has 'riscv64' in its name is because it can compile both 32 and 64 bit RISCV. Note also that this compile won't readily be available on any Linux machine. It is already set up on your lab machines and `eceprog` for the purpose of the lab. 

### 1.3 Assembler

Converting assembly instructions to the bitstrings is handled by yet another program called an assembler.

The assembler will do the final translation from plain text assembly to bit strings:

As a reference, here is the HelloWorld.c main function in assembly:
```asm
...
.LC0:
        .string "Hello World!"
        .text
        .align  1
        .globl  main
        .type   main, @function
main:
        addi    sp,sp,-48
        sd      ra,40(sp)
        sd      s0,32(sp)
        addi    s0,sp,48
        mv      a5,a0
        sd      a1,-48(s0)
        sw      a5,-36(s0)
        lui     a5,%hi(.LC0)
        addi    a0,a5,%lo(.LC0)
        call    printf
      ...
```

Now, we invoke the assembler and check the file content:

```bash
# NOTE: This is just an example. You are not expected to
# understand these instructions or what it is doing
# yet.

# The following example can be run on eceprog or your lab machine.

# Assemble the helloworld assembly code into an object
# file called HelloWorld.o
riscv64-unknown-elf-gcc -c HelloWorld.s -o HelloWorld.o

# Examine the HelloWorld.o content
# Use objdump to disassemble the object file
# You could also use xxd to examine the object file
#     xxd: create a hexdump of the binary file
#     xxd HelloWorld.o

riscv64-unknown-elf-objdump -D HelloWorld.o > HelloWorldObjectFile.txt

```
The following are the contents of the text file generated.

```text
...
00000000000101a2 <main>:
   101a2:       7179                    add     sp,sp,-48
   101a4:       f406                    sd      ra,40(sp)
   101a6:       f022                    sd      s0,32(sp)
   101a8:       1800                    add     s0,sp,48
   101aa:       87aa                    mv      a5,a0
   101ac:       fcb43823                sd      a1,-48(s0)
   101b0:       fcf42e23                sw      a5,-36(s0)
   101b4:       67f1                    lui     a5,0x1c
   101b6:       79078513                add     a0,a5,1936 # 1c790 <__clzdi2+0x48>
   101ba:       14c000ef                jal     10306 <printf>
   101be:       478d                    li      a5,3
   101c0:       fef42623                sw      a5,-20(s0)
   101c4:       4781                    li      a5,0
   101c6:       853e                    mv      a0,a5
   101c8:       70a2                    ld      ra,40(sp)
   101ca:       7402                    ld      s0,32(sp)
   101cc:       6145                    add     sp,sp,48
   101ce:       8082                    ret
...
```

<!-- SHRGAUR -->
<!-- We can see that at the start, there is a 32-bit word `ffc3 00d1`, which is `sub sp, sp, #48`, followed by `fd7b 02a9`, corresponding to the next line `stp x29, x30, [sp, #32]`, which is the start of our main function. -->

### 1.4 Linker

Generally, you cannot just run the generated object file on a computer running an OS, although you might able to run it on a "bare-metal" CPU.  There are some additional tasks that need to be done to create an executable file and since we have to use the `stdio.h` header file, we will have to link to it so that the program know where to find `printf()`.  To do so, simply use `riscv64-unknown-elf-gcc HelloWorld.o -o HelloWorld`, which will automatically handle all the tasks of invoking the compiler, assembler, and a final tool called a linker that will create the final executable binary.

```shell
# Again you can use the riscv gcc compiler in order to generate an executable.
riscv64-unknown-elf-gcc HelloWorld.o -o HelloWorld
./HelloWorld
> Hello World! 
```

> [!NOTE]
> *Wait, I thought you said I \*couldn't\* run RISC-V programs on my computer, but I just did! Explain yourself!*
> Run `file HelloWorld`.  You'll see the following output:  
> 
> `HelloWorld: ELF 64-bit LSB executable, UCB RISC-V, RVC, double-float ABI, version 1 (SYSV), statically linked, not stripped`
> 
> This indicates that yes, you did just compile a RISC-V binary.  Yes, it did just execute, but no, it did not do so on your x86 processor.  It did so using the QEMU emulator we've installed for you when you run `module load riscv`.  This is how you can run RISC-V binaries on your computer in the same manner as any x86-compiled program, but it is not running on your computer's hardware.

If you have multiple source files, the linker will manage merging these object files together into a single executable.

We actually use this feature to in the following exercises to link the autograder object file with your solutions. This way the grader could run your code and see if the results are correct.

### 1.5 Compiler Summary

The following figure provides an overview of the flow necessary to generate an executable from
the source code you are familiar with.

```text
┌─────────────┐  Compiler   ┌──────────┐  Assembler  ┌─────────────┐
│ Source Code ├────────────►│ Assembly ├────────────►│ Object File │
└─────────────┘             └──────────┘             └──────┬──────┘
                                          Linker            │
                                 ┌──────────────────────────┘
                                 │
                          ┌──────▼───────┐
                          │  Executable  │
                          └──────────────┘
```

## 2. Assembly instruction: Arithmetic

In this sections, we will discuss the arithmetic instructions of RISCV32 ISA and write some code with them.

### 2.0 Set-up for Lab

The set-up for the lab has already been provided to you. Make sure you finished the [setup](./lab80.md) portion of the lab before you continue further with Lab 8.1.

### 2.1 Example

Fill in the subroutine body for `q2_1_example` with instructions that will set the value of `x10` to `x10 + x11 + x12 + x13`. For instance, if you set the values x10 through x13 like this:

<!-- ```asm
mov x0, #1
mov x1, #2
mov x2, #4
mov x3, #8 
``` -->  

```asm
li x10, 1
li x11, 2
li x12, 4
li x13, 8
```
You should expect that the x10 register will contain the value 15 when execution reaches the nop following the subroutine invocation. It does not matter what values are left in x11, x12, and x13 after the return from example. Remember to use only the registers x10 through x13 when you write your instructions.

#### Solution for the example

The operation cannot be implemented with a single instruction. You must compose multiple instructions to produce the result.

<!-- ```asm
.global q2_1_example
q2_1_example:
    /* Enter your code after this comment */
    
    add x1, x0, x1 // now, x1 = x0 + x1
    add x1, x1, x2 // now, x1 = x0 + x1 + x2
    add x1, x1, x3 // finally, x1 = x0 + x1 + x2 + x3
    mov x0, x1     // put the result into x0
    
    /* Enter your code above this comment */
    ret lr
``` -->

```asm
.global q2_1_example
q2_1_example:
    /* Enter your code after this comment */
    
    add x11, x10, x11 // now, x11 = x10 + x11
    add x11, x11, x12 // now, x11 = x10 + x11 + x12
    add x11, x11, x13 // finally, x11 = x10 + x11 + x12 + x13
    add x10, x11, 0 // Move the results to x10

    /* Enter your code above this comment */
    ret
```

You should copy this into the example subroutine in the file, and trace through the execution with the debugger to make sure you understand how it works.

#### Another solution for the example

There are usually many ways to write the same high-level operation in assembly language. The fewer instructions you can use, the faster the code will run to completion. Here is another solution for the previous problem that has fewer instructions:

<!-- ```asm
.global q2_1_example
q2_1_example:
    /* Enter your code after this comment */
    
    add x0, x0, x1 // now, x0 = x0 + x1
    add x2, x2, x3 // now, x2 = x2 + x3
    add x0, x0, x2 // finally, x0 = (x0 + x1) + (x2 + x3)

    /* Enter your code above this comment */
    ret lr
``` -->

```asm
.global q2_1_example
q2_1_example:
    /* Enter your code after this comment */
    
    add x10, x10, x11 // now, x10 = x10 + x11
    add x12, x12, x13 // now, x12 = x11 + x12
    add x10, x10, x12 // finally, x10 = x10 + x11 + x12 + x13

    /* Enter your code above this comment */
    ret
```

Since some registers are reused, it may be a little more difficult to understand.  You should study it to discover how it works. For this lab's exercises, it does not matter how slowly your solution works (within reason).  What does matter is that no registers other than x10, x11, x12, or x13 are
modified by the code.

> [!Note]
> For the purpose of this lab we refer to the registers with their x name. You can write your solution using the ABI names of the registers as well. To get the ABI names, see page 137, Table 25.1 in the reference manual.  TL:DR; x10 - x17 maps to a0-a7.

### 2.2 Calculating the discriminant of a quadratic equation

For a quadratic equation $ax^2 + bx + c = 0$, its discriminant, commonly represented by the Greek symbol Delta, determines if the equation has any real roots:

$$
\begin{align*}
  \Delta &= b^2-4ac
\end{align*}
$$

If $\Delta = 0$, the equation has two identical real roots; if $\Delta > 0$, two distinct real roots exist; if $\Delta < 0$, no real roots exist.

Use the assembly function `q2_2_delta` to write the assembly necessary to compute the discrminiant of a quadratic equation with the coefficents given in registers `x10-x12`.  You will need to put the final result back in `x10` when the function return.  Specifically, the coefficients and registers mapping is:

```C
// For the equation ax^2 + bx + c = 0
a: reg x10
b: reg x11
c: reg x12

// Final result need to be put in register x0 
delta: reg x10
```

After completing the problem, you could build and run the `lab` executable. The autograder will grade your function by compared its result with the C equivalent function on random inputs.

> Hint: You might find the instructions `sub`, `li`, `mul` to be useful. A reference manual to the RISCV32 can be found [here](https://riscv.org/technical/specifications/). Use the unprivileged version of the manual. 

> For a quicker reference, you can check out the Reference Data card we'll post on Piazza.  You may want to bookmark that one.

### 2.3 Calculating the dot product of two 2-D vectors

For two vectors in $\mathbb{R}^2$ space, $\vec{A} = (a_1, a_2)$ and $\vec{B} = (b_1, b_2)$, the dot product between them is commonly defined as:

$$
\begin{align*}
  \vec{A} \cdot \vec{B} = a_1b_1 + a_2b_2 
\end{align*}
$$

In this problem, you will fill up the assembly function `q2_3_dot_product` to compute the dot product of two 2D integer vectors with their components specified in registers `x10-x13`. Similar to previous problem, you will need to put the result back in `x10` when the function return. The mapping of the arguments and registers mapping is:

```C
// For two vector A, B
// A = (a1, a2)
// B = (b1, b2)
a1: reg x10
a2: reg x11
b1: reg x12
b2: reg x13

// Final result
a1b1 + a2b2: reg x10
```

After completing the problem, you could build and run the lab1 executable. The autograder will grade your function by compared its result with the C equivalent function on random inputs.

> [!NOTE]
> You might find the instructions `add` and `mul` to be useful. 

## 3. Assembly instruction: bit-wise/logical

In this section, we will use some of the logical/bit-wise instructions of RISC-V ISA and write some code with them.

### 3.1 Taking one byte from a 32-bit word

Often in embedded programming, you will need to extract one byte from a word and perform some operations on it.
In this problem, you will need to extract the MSB (most significant byte, the byte to the leftmost)
and the LSB (least significant byte, the byte at the rightmost position) of a **32-bit** word.
There are two functions you will need to implement: `q3_1_MSB` amd `q3_1_LSB`.
Both functions will be passed in with the word in register `x10`, and you will put the result back in `x10`.

For example: Suppose the given word is `0x11223344AABBCCDD`, its MSB will be `0x11` and its LSB will be `0xDD`.

> [!Note]
> Be careful to remember that the B in LSB and MSB is **byte**, not bit!
> 
> Hint: You will need to use logical shift operation and bit-wise AND operation to extract one byte from a 32-bit word. You might find the instructions `srli` and `and` useful. Also, you might also want to consider loading a register with value `0xFF`.

### 3.2 Turning a flag on/off

In embedded programming, usually you will need to configure the peripherals associated with the microcontroller. For instance, you might need to enable certain pins on the development board or configure the clock frequency of the CPU. A typical way to enable or disable them is by setting or resetting certain bits in control registers, which can hold a 32-bit or 64-bit value, with each bit corresponding to a different function. You can imagine each bit as a tiny switch to control one part of the microcontroller.

Another common name for these individual bits (tiny switches) is *flag*, which is a value with only one bit set to 1 and the rest to 0, like `0x1000000` or `0b0001 0000 0000 0000`.  

Keep in mind that the flag is not the actual bit position that you need to shift to - it is the already-shifted value.  Therefore, if you needed to turn on bit 17 in a specific control register, the flag value given to you will be `1 << 17`.

In this problem, you will implement 3 functions to:

1. Set the bit of a value `x10` corresponding to a flag `x11` to `1` (function `q3_2_flag_set`)
2. Set the bit of a value `x10` corresponding to a flag `x11` to `0` (function `q3_2_flag_reset`)
3. Toggle the bit of a value `x10` corresponding to a flag `x11` (function `q3_2_flag_toggle`)

For all three of them, the value will be passed in `x10` and the flag will be passed in `x11`. You will again save the modified value to `x10`.

> Hint: Instruction `or`,`xor`, `xori`, and `and` might come in handy. Also note that there is no special instruction for inversion in RISC-V. The developer made the decision to use xori with 0xFFFFFFFF to get the inverse instead in order to keep with the three-operand format of the instructions.

### 3.3 Swapping the LSB and MSB of a 32-bit word

> [!Note]
> You could reuse what you have in section 3.1 to extract LSB and MSB.

In this problem you will implement the function `q3_3_swap_byte` that swaps the LSB and MSB of a 32-bit word passed in from `x10` and stores the swapped word in `x10`.

You should only swap the LSB and MSB and leave the rest of the bytes intact.

> [!TIP]
> `slli` shifts to the left and `srli` shifts to the right, and both will pad the corresponding side with zeroes.  You also might want to review what you have done for [*3.2 Turning a flag on/off*](#32-turning-a-flag-onoff).

## 4. Assembly instruction: memory

So far, you have only been manipulating the register values. However, as you might already know, there are simply not enough registers to hold all the variables within a program. This is where memory steps in.

With memory, we could store the values in our registers to memory and leave space to perform other calculations. When we are done, we could just loaded the values back from memory and continue doing our work. You can think of memory as a gigantic hash table or array with `address` as the key or index:

```C
// Pseudo-code for memory store and load

// Storing to memory, or writing
Mem[addr] = variable

// Loading from memory, or reading
variable = Mem[addr]
```

In this section, we will work with memory instructions to perform some simple stores and loads, but first, we will have a crash course on assembly directives that helps programs know where a variable is in memory and what size does it has.

### 4.0 Crash course on assembly directives

In this lab, you will only work with the `.global` and `.asciz` directives. You can safely ignore the others.

1. `.global SYMBOL`: make the symbol `SYMBOL` visible to other object files during linking, which could either be a variable or a function.
2. `.asciz`/`.string "SOME_STRING"`: specify a null-terminating C-string
   1. Null-terminating means that the assembler will append a `\0` char to the end of the string.
  
From a high-level point of view, these directives provide some meta-info beyond the actual instructions that both the [assembler](#13-assembler) and [linker](#14-linker) can utilize to generate the executable.

### 4.1 Uppercase formatter
In this problem, you will implement the assembly function `q4_1_toupper` that will take in an address of a lowercase letter and save its uppercase form to the same address like the C program below:

```C
// Assume we have this function implemented
void toupper(char *c) {...}

char chr = 'a';
char *string = "hello";

toupper(&chr);
toupper(string);

// After the above function calls, we will have
  chr = 'A'
  string = "Hello"
```

The address will be passed in register `x10`, you will need to read the value at the address in `x10`, modify it, and then store back to the same address.

In addition, the autograder will use your function to modify the string at label `lowercase_string` and print it out! You could also change the string to whatever you like and observe the effect (autograder won't use this string for grading).

> [!Note]
> You can safely assume that the character at the address passed in will always be lowercase.   
> 
> Hint #1: You would want to look at instructions `lb`, `li`, `sub` and `sb` for this task.  
> 
> Hint #2: Check out the ASCII table (try `man ascii` on a Linux or Mac terminal) and find the relation between lowercase and uppercase letter!

### 4.2 Swapping two integers

> Swap: <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;take part in an exchange of <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-- from Oxford dictionary

In this section, you will implement the assembly function `q4_2_swap` that will take two 32-bit integers addresses and swap the content, similar to the C program below:

```C
void swap(int32_t *a, int32_t *b) {...}

int32_t x = 362;
int32_t y = 270;

swap(&x, &y);

// After the above function call
x = 270
y = 362
```

The two addresses will be passed in register `x10` and `x11` respectively.

### 4.3 String cutter

> Sometimes, strings are just too long.

In this problem, you will implement the assembly function `q4_3_cutter` that will take two arguments, a C character string starting address and a cut position, passed in from `x10` and `x11`. The function will be similar to the C program below:

```C
void cutter(char * str, uint32_t pos) {
  *(str + pos) = '\0'
}

char string[50] = "Hello World!";
cutter(string, 5);
printf(string);

// After the above function call, printf will print
// 'Hello' instead of 'Hello World!' to the console
// since the bit at position 5 gets replaced with the 
// null character '\0'.
```

> Hint: You do not need to load the string.

### 4.4 Endianness: which side are you with?

> From *Gulliver's Travels* by **Jonathan Swift**
> 
> It is allowed on all hands, that the primitive way of breaking eggs before we eat them, was upon the larger end: but his present Majesty's grandfather, while he was a boy, going to eat an egg, and breaking it according to the ancient practice, happened to cut one of his fingers. Whereupon the Emperor his father published an edict, commanding all his subjects, upon great penalties, to break the smaller end of their eggs.
>
> [reference](https://www.ling.upenn.edu/courses/Spring_2003/ling538/Lecnotes/ADfn1.htm)

Although there is not a physical egg within a computer, we did find a way to start a "war" on which end should we break the egg: the Big-Endian or the Little-Endian.

Before we actually explain these two vague terms, let's consider how does a computer store a 32-bit integer inside memory. 

We all know that (or know by now) that modern computer store data at byte (8 bits) granularity inside memory.  However, for a 32-bit integer like `0x11223344`, it has `4` bytes, in what order should we store it?  There are `4! = 24` ways to do so, and we could just store it randomly.  But to make our lives easier, and to make the computer's life easier, we should probably store them in consecutive order starting at a memory address.

Therefore, we store one byte at address `x`, and then store the next byte in order at `x + 1`, and continue until we finish all `4` bytes.  However, there exist two ways to do this - we store the MSB first, or the LSB first.

|address  |   `x`  | `x + 1` | `x + 2` | `x + 3` |
|:--      |  :--:  |  :--:   |  :--:   |  :--:   |
|LSB first| `0x44` | `0x33`  | `0x22`  | `0x11`  |
|MSB first| `0x11` | `0x22`  | `0x33`  | `0x44`  |

> [!Note]
> MSB refers to most significant byte and LSB refers to least significant byte, check [3.1](#31-taking-one-byte-from-a-32-bit-word) for more explanation on them.

If we store the MSB first, it is Big-Endian; if we store the LSB first, it is Little-Endian.

<!-- Now back to `section 4.1`, using `ldr` instead of `ldrb` will work is because when we read using `ldr`, it will load 8 bytes (or 4, depends on whether you use `xn` or `wn`) to the register. Since the architecture we use is `ARMv8-A`, which is Little-Endian by default, the LSB of the register will hold the character at the memory address, which is the one we want to modify. If you convert the lower case character to upper case by modifying it at the bottom 8 bits of the register, you will see that `ldr` and `str` will work. -->

In this part you will need to implement the function `q4_4_cvt_endian` that will take in a 32-bit integer and reverse its endianness. That is, if it is little-endian, we want it to be big-endian, and vice-versa.

Again like the previous problem, the integer will be passed by reference with its address passed in register `x10`. You will have to store the reversed endianness integer back the same address.

> Hint: You might find yourself copy pasting a lot, which is normal, as we won't cover loops until next lab. 

<hr>

*The following tidbit is more relevant to computer architecture if you're curious.*

It may seem tedious to do this step without loops, but implementing a function like this is actually a common compiler optimization called [loop-unrolling](https://en.wikipedia.org/wiki/Loop_unrolling) that is actually faster compared to a loop.  

Loops are typically implemented with instructions called **branches**.  These are instructions that tell the CPU to jump to a different part of the program, either with no conditions, or based on the result of a condition (branch-if-equal, branch-if-less-than, etc.).

Modern CPUs are implemented with a component called **branch predictors**.  These try to predict the next instruction to execute when it arrives at the beginning of an if/else structure, marked by a **branch** instruction.  The CPU may start loading the instructions at the part of the program indicated by the predictor, even while the branch/compare instruction itself is being executed.  

With a correct prediction, a CPU can execute instructions much faster than if it had to wait on the result of the branch instruction.  

However, if the prediction is wrong, the CPU will have to "flush its pipeline" of instructions and start over.  This is called a branch mispredict, and can reduce the efficiency of a program.  Loop unrolling reduces the number of branches and therefore reduces the number of branch mispredictions.  This is why compilers will often unroll loops for you if the limits of the loop are known at compile-time.
