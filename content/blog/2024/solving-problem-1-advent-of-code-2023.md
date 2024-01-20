---
title: Solving problem 1 of Advent of Code 2023 in x86 Assembly
author: Phineas Jensen
date: 2024-01-20
description: "Reminiscing on a favorite class—A bright idea—A simple plan for a simple problem—Choosing an assembler and syntax—A tour of the program"
tags:
  - assembly
---

A few years ago, I took a systems programming class which briefly covered x86 assembly in a project which involved using GDB to defuse a series of "bombs" (essentially figuring out what input to pass to break the code and jump to certain areas). That was one of my favorite classes I've taken in my undergrad program, and I really enjoyed working with assembly language, but I hadn't really touched it since then, until the beginning of this year.

I knew I wouldn't have time to do the complete Advent of Code challenge (I never have, and time was even more limited this year), so I thought I might only solve one or two problems, but give myself an extra challenge. That made it the perfect opportunity to try to solve one of the problems in x86 assembly, right? This blog post stands as a journal entry and guide to my solution. I hope you find it interesting and/or helpful!

## The Problem and the Plan

I opened up the [Advent of Code day 1](https://adventofcode.com/2023/day/1) to read the problem description:

>The newly-improved calibration document consists of lines of text; each line originally contained a specific calibration value that the Elves now need to recover. On each line, the calibration value can be found by combining the first digit and the last digit (in that order) to form a single two-digit number.
> 
>For example:
>
> ```text
> 1abc2
> pqr3stu8vwx
> a1b2c3d4e5f
> treb7uchet
> ```
> 
>In this example, the calibration values of these four lines are 12, 38, 15, and 77. Adding these together produces 142.
> 
>Consider your entire calibration document. What is the sum of all of the calibration values?

It's a pretty simple problem. Find the first and last digit on each line, combine them into a two-digit number, and return the sum of each of those. Right away, I could imagine a solution that would work well with the primitives of assembly language:

1. Get the input into memory, knowing the address of the beginning and the end
2. Choose registers to act as the first and last digits for each line
3. Iterate through a line, updating those registers when digits are found
4. Stop at a newline and convert the two digits into a number, then add those to a final result register
5. Repeat 3-4 until the end of input
6. Convert the result to a string and print it out using a syscall

## Assemblers and Syntax Styles

With that plan in mind, I got started figuring out how to write assembly from scratch. After a few searches online and skimming a few blog posts and Wikipedia pages, I realized and was reminded that "assembly language" isn't as straightforward a concept as I thought.

First of all, there are a number of assemblers available, with each supporting different features and syntax to define things like data sections and references to memory addresses. It didn't take me long to run into [NASM](https://en.wikipedia.org/wiki/Netwide_Assembler) (the Netwide Assembler), [Open Watcom Assembler](https://en.wikipedia.org/wiki/Open_Watcom_Assembler), [FASM](https://en.wikipedia.org/wiki/FASM), and (my choice for this project), the [GNU Assembler](https://en.wikipedia.org/wiki/GNU_Assembler) (or `gas` or `as` for short).

I still don't know a lot about the differences between these and I may have made the "wrong" choice, but I chose the GNU Assembler because it seemed most familiar, since most of my exposure to assembly comes from using GDB (another GNU project). It was also already installed as part of the build tools on my system.

There are also two [different syntaxes](https://en.wikipedia.org/wiki/X86_assembly_language#Syntax) of x86 assembly: Intel and AT&T. I chose AT&T for two reasons: First, I like that the source of an instruction comes before the destination, as in `movl $5, %eax`. That reads naturally to me as "move the number 5 into the `eax` register". Intel would write that as `mov eax, 5`, which translates in my head as the more clunky[^1] "move into the eax register the value 5". Second, GNU Assembler defaults to AT&T syntax, and I figured I'd start with defaults.

In retrospect, I think I might have preferred Intel syntax. I vaguely remember one of my professors telling the class to "do yourself a favor and configure GDB to use Intel syntax", and I can see some of the benefits now:

- Intel syntax doesn't need `$` for immediate values and `%` for registers.
- The lack of variant instructions for different sizes of data (e.g. `movq` vs `movl` in AT&T are just `mov` in Intel). It was a bit annoying to have to change both a register name and the instruction name when switching from a 64-bit to 32-bit operation, when I could have just changed the register name with Intel syntax.
- Online resources for Intel syntax seem more plentiful and/or easier to find. I had to translate those mentally to really understand them.

Oh well! I got the job done with AT&T syntax, so it's not a big deal.
## The Program

To start, we need to set up the memory we will use to read the input text and print the result:

```gas
.data
result: .ascii "         "
result_end:
input: .ascii "1abc2
pqr3stu8vwx
a1b2c3d4e5f
treb7uchet
"
input_end:
```

Rather than reading from standard input, we're just embedding the problem input directly into the program's data section, using the [`.data`](https://sourceware.org/binutils/docs/as/Data.html) directive (the [`elf(5)`](https://manpages.debian.org/stretch/manpages/elf.5.en.html) man page says the `.data` section "holds initialized data that contribute to the program's memory image."). Our output string `result` we initialize as a bunch of spaces using the [`.ascii`](https://sourceware.org/binutils/docs-2.41/as.html#Ascii) directive (enough to store the entire result, which shouldn't be very long), and our input string in `input` the same way. We have to make sure the input ends with a newline, because `\n` is how we tell we've hit the end of a line.

We also add `result_end` and `input_end` labels after each of these strings so we know where they end as well, which will make it easy to stop iterating over the input and write the output string in reverse order. As each label refers to a memory address and with the data in between ordered contiguously, the `_end` ones refer to the address immediately after the previous chunk's end.

Next, we have to set up the section that holds the executable instructions and entry point of the program:

```gas
.text
.globl _start
_start:
```

[`.text`](https://sourceware.org/binutils/docs-2.41/as.html#Text) is a directive that indicates the start of the section containing executable instructions[^2], while [`.globl`](https://sourceware.org/binutils/docs-2.41/as.html#Global) (or `.global`, they both do the same thing) makes the `_start` label visible to the linker. Though I haven't yet found somewhere that lays it out explicitly, I assume that the linker would know by convention that `_start` is the entry point for the program.

Now that we've got a section for our code defined, we can put the first instructions in. For this section, I started it with a few initializations:

```gas
_start:
	movq $input, %r8  # Address of current character of input string is stored in %r8
    movq $0, %r15     # Result value is stored in %r15
    jmp reset_numbers
    
reset_numbers:     # Initialize first & last digits and the first-digit-found flag
    movb $0, %r11b # first digit
    movb $0, %r12b # last digit
    movb $0, %r13b # first digit found flag
```

I chose these registers at random, and that's another place where my knowledge of assembly is lacking. I know that there are conventions for how the CPU registers are used, but I don't know what those conventions are. For a problem this small, it doesn't matter.

For the initialization of the registers holding the first and last digit of each line (as well as a register to keep track of whether we've found the first digit yet at all), I used a label to name the block `reset_numbers`. We'll be resetting these numbers for each line of input, so we need an address to jump back to.

Immediately following that block is the main body of the loop (which I cleverly named `loop`) that will iterate through each character of the input:

```gas
loop:
    cmpb $0x30, (%r8) # Skip character if less than ascii '0'
    jl inc

    cmpb $0x39, (%r8) # Skip character if greater than ascii '9'
    jg inc

    movb (%r8), %r10b # Copy current character into r10
    subb $0x30, %r10b # convert from ascii to plain integer

    cmpb $0x0, %r13b  # If the first digit has been found
    jne last_digit    # jump directly to updating the last digit

    movb %r10b, %r11b # Update the first number for the line (%r11b)
    incb %r13b        # and mark the first number as found (%r13b)

last_digit:
    movb %r10b, %r12b    # Update the last digit found
```

The first four instructions here just compare the character at the address stored in `%r8` (using parentheses around the register name as a sort of pointer-dereference[^3]) to the ASCII value for 0 and 9. If you're a little rusty on ASCII, take a gander at [`man ascii`](https://manpages.debian.org/stretch/manpages/ascii.7.en.html) and notice that the 10 digits are placed contiguously in that encoding. If we take the value for the ASCII character '0' (which happens to be `0x30`) and subtract it from each of the ASCII digit characters, we get their integer values (e.g. `0x34 '4' - 0x30 '0' = 0x04).

If the current character falls *outside* that range, we know that it's not a digit, so we can skip to the next letter, so we jump to the `inc` label, which we will write below. `inc` will move the loop forward, breaking out of it if we've reached the end of the input.

If the current character is a digit, then we can copy it into another register (I'm going with `%r10` for this one) and once again subtract `0x30` from it to get the integer value of the digit. Then we check if we've found our first digit for the line (by checking if `%r13` is non-zero). If the line's first digit *has* been found, we skip the next two instructions by jumping to the `last_digit` label. Otherwise, we execute those instructions which update our first digit register `%r11b` and our first-digit-found register `%r13b` before continuing on to execute what comes after the `last_digit` label; that is, updating the last digit register `%r12b`.

With that character handled, we can move onto the next one as the program continues into the `inc` block, which looks like this:

```gas
inc:
    incq %r8             # Move character pointer
    cmpq $input_end, %r8 # Check if we're at the end of the input
    jge print_result     # and jump to printing the result if so

    cmpb $0x0a, (%r8)    # Check if the current character is a newline,
    je add_to_result     # and add to final result if so

    jmp loop             # Jump to beginning of the main loop
```

The first instruction here is pretty obvious: move the character pointer forward by incrementing it once. Next, although we check if we're at the end of the input by comparing to the `input_end` address we set up earlier. If we are at the end of input, we jump to the `print_result` label, which I'll describe below.

Otherwise, we check if the current character is a newline (ASCII `0x0a`). If it is, we can jump to a block I labeled `add_to_result` which takes those first and last digits from the line we have and adds them to our final result value. I'll discuss that next.

If we don't jump to `add_to_result`, we instead jump back to `loop` to continue on with the next character.

Here's `add_to_result`:

```gas
add_to_result:
    imulq $10, %r11      # Multiply first digit by 10
    addq %r11, %r15      # Add first digit to result
    addq %r12, %r15      # Add last digit to result
    incq %r8             # Move character pointer forward
    cmpq $input_end, %r8 # Check if we're at the end of the input
    jl reset_numbers     # and jump to printing the result if so
```

Since we're working with 2-digit numbers, it's fairly simple to combine them into one value we can add to our total result: Multiply the first digit by 10 and add it to the second digit. Or, in this case, multiply the first digit by 10 and add it to the result, then add the second digit to the result. Same effect in either case.

After that, we can increment our character pointer again, check if we've reached the end of input, like in the `inc` block, and jump back to `reset_numbers` if we're ready to process another line.

If we did reach the end of the input either in this block or earlier in `inc`, then we move on to the `print_result` block:

```gas
print_result:
    movq $result_end, %r14 # Keep output pointer (end of output location) in %r14

    movq %r15, %rax  # Store result (dividend) in %rax
    movq $10, %rcx   # Store divisor in %rcx

print_loop:
    movq $0, %rdx    # Reset first 64 bits of dividend to 0
    divq %rcx        # Divide!

    addq $0x30, %rdx # Remainder is stored in %rdx, so add 0x30 to make it ascii
    movb %dl, (%r14) # Put character at output pointer location

    cmpq $0, %rax    # If the result of division was 0
    je print         # jump to print the result

    decq %r14        # Move character pointer left
    jmp print_loop   # Go back to the start of the loop
```

This was probably the hardest part of the program for me, as I had to consider how to convert an integer to a string in assembly and I didn't really know how to allocate or write memory at all. For the memory problem, I decided to just create a section in the `.data` segment (as described above) that I could write to.

Converting an integer to a string is tricky because the basic process I came up with (divide by 10, store the remainder as a character in the string and repeat, dividing the quotient, until the result is 0) returns the digits of the number in reverse order:

> 142 / 10 = 14 with remainder of 2
> 14 / 10 = 1 with remainder of 4
> 1 / 10 = 0 with remainder of 1
> 
> We've got our digits 1, 4, and 2, but they're in reverse order :(

But if we just write these digits to our output string *backwards*, that's not an issue. Luckily, we've got a nice `result_end` address; we can just start there. So the first instruction here stores `result_end` in `%r14` as our output character pointer. After that, we store our dividend (starting with the problem's final answer) in `%rax` and our divisor (10) in `%rcx`.

`divq` is probably the second most fun instruction in this program. It expects its dividend to be stored in `%rdx` *and* `%rax` (meaning you can divide 128-bit numbers), and the divisor is the instruction's only direct argument. Then, it stores the quotient in `%rax` and the remainder in `%rdx`. The nice thing about that is that you can just divide the quotient again by clearing `%rdx` and calling the instruction again (if you don't clear `%rdx`, the remainder would drastically embiggen the result, so `142 / 10` would put `14` in `%rax` and `2` in `%rdx`, making the dividend to the next call be `0x0200000000000000e` instead of `0xe`).

After calling `divq`, we add `0x30` to our remainder to convert it into an ASCII character, then we use `movb` (**not** another size like `movq`) to store that character at the address pointed to by our output pointer `%r14`. I messed things up here a few times by trying to use `movq`. Eventually, I realized that that was writing an entire 64 bits which messed up the output. I was only trying to write one byte anyway, so `movb` makes more sense.

After that, we have `cmpq $0, %rax` check if our quotient is 0, meaning we've divided our way to the last digit and we're ready to print the result. If so, we jump to `print`, which I'll show below. If not, we decrement `%r14`, moving the output character pointer back once, and jump back to the beginning of `print_loop`.

Here's `print`:

```
print:
    movq $0x04, %rax       # 4 is write()
    movq $0x1, %rbx        # File descriptor 1 (stdout)
    movq %r14, %rcx        # Pointer to result string
    movq $result_end, %rdx # Store address of result string as output length
    sub %r14, %rdx         # and subtract the beginning of the string
    incq %rdx              # and add 1 to get the actual length of the string
    int $0x80              # Send the syscall interrupt
```

What we're doing here is setting up the arguments for the `int 0x80`[^4] instruction, which executes a syscall:

- The index of the `write()` syscall in `%rax` 
- The file descriptor for STDOUT (which is 1) to write to in `%rbx`
- The start of the string to write (which is our current result character pointer) in `%rcx`
- The length of the string we're writing in `%rdx`

We calculate the length of the string by taking the end address, subtracting the beginning, and adding 1. After that, we can call `int 0x80` again which will write the result, and then we can exit:

```gas
end:
    movl $1, %eax # 1 is exit()
    movl $0, %ebx # Exit status 0
    int $0x80     # Send the syscall interrupt
```

We exit by again using a syscall, but this time it's `exit()`. Since the program should have succeeded if we've reached this point, we set the first argument as the number 0, meaning no error, and then interrupt.

## Building and executing

I've stored the entire program in a file called `day1.asm`. Assembling this program into an executable requires two steps:

1. Use the GNU Assembler to generate a "relocatable file"[^5], which has the instructions and data assembled into a binary format, but isn't a fully executable file.
2. Use `ld`, the linker, to convert that file into an executable.

We can set that up in a simple Makefile to make things easier:

```makefile
day1: day1.asm
		as -g day1.asm -o day1.o
		ld day1.o -o day1

clean:
        rm *.o day1
```

The `-g` flag adds debugging info, which was very helpful as I used `gdb` to debug a bunch of parts of this program.

With this set up, we can build and execute our program to get our final result:

```bash-session
$ make
as -g day1.asm -o day1.o
day1.asm: Assembler messages:
day1.asm:86: Warning: unterminated string; newline inserted
day1.asm:87: Warning: unterminated string; newline inserted
day1.asm:88: Warning: unterminated string; newline inserted
day1.asm:89: Warning: unterminated string; newline inserted
ld day1.o -o day1
$ ./day1
142
```

And there we have it, a solution for day one of the Advent of Code in x86 assembly!
## Resources

- [Brown CS033's x86 Assembly cheat sheet](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf) wasn't how I figured out how to use `divq`, but I wish it was. It's great.
- [A Beginner's Guide to x86 Assembly, Part 1](https://derek.gurchik.com/2017/beginners-assembly-part1) by Derick Gurchik is probably a much better resource than this blog post, and it's where I (re-)learned how system calls work in assembly.
- The [X86 Assembly](https://en.wikibooks.org/wiki/X86_Assembly) Wikibook. I didn't use this much but found it very helpful for answering some incidental questions I had.
## Further Questions

- Do registers need to be initialized with some number, or do they default to 0 or some other value?
- Why exactly is the linker necessary? What's different from the `.o` object and the final binary?
- What is an effective address? Is that syntax an assembly language thing, or a GNU Assembler thing? How do the offset and scale parts work? Why does Intel syntax make it look so much better?
- How do you allocate memory? What does it even *mean* to allocate memory?
- How does the ELF format work? What would this look like on Windows?
- How is the `_start`  symbol used? Does the linker know that's the program's entry point? Is that an ELF convention? An OS convention?

[^1]: Another reason to use Intel syntax occurred to me as I edited this post. A lot of instructions have only a destination argument, which makes the destination consistently the first argument in Intel syntax. Although now I'm realizing there are also some instructions which only have a source argument, which throws a wrench in that argument. Ultimately, I guess both can work and can be gotten used to.
[^2]: It's not clear why exactly this section is called `.text`, when it seems like it's got a lot more going on than just text. A [few](https://stackoverflow.com/questions/1282506/where-did-the-text-segment-get-its-name) [explanations](https://softwareengineering.stackexchange.com/questions/171565/why-is-the-code-section-called-a-text-section) seem to float around and are all plausible, but ultimately we can just chalk it up to convention.
[^3]: I believe that's called an effective address, and the syntax can get more complicated. Honestly, I have a very tenuous grasp on that, hence it being a subject in the [[#Further Questions]] section.
[^4]: Turns out `int 0x80` is the old 32-bit x86 way of interrupting for a syscall, while x86-64 has a dedicated `syscall` instruction. The fun part is that they use different registers for arguments and different numbers to indicate syscalls. I haven't really found a good source on what those syscall numbers are yet.
[^5]: A sort of assembled, but not "fully executable binary; it's more like a "partial" executable that lacks certain information", as ChatGPT explained it.
