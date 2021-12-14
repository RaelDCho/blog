# Session Two Challenges
---
## Tools
* Disassembler - used Cutter

# Control Flow 1

Key point
* 4 values are checked
  * argc
  * and the values provided by the user

### Analysis

<!-- insert picture of first block -->
![0x68a](~/Images/HackadayTutorials/0x68a.JPG)

The program begins with the stack being set up in:

```
push rbp
mov rbp, rsp
sub rsp, 0x20
```

<!-- maybe insert a theoretical image of the stack -->

The old `rbp` is pushed onto the stack.

`rbp` is then moved to `rsp` to show the start of the current stack.

And then `0x20` bytes of memory is allocated for the stack (which includes things like the buffer).

The next few lines of instructions:
```
mov dword [ac], edi
mov dword [av], rsi
cmp dword [ac], 3
je 0x6b5
```

Basically store the `argc` inside variable `ac` and `argv` inside variable `av`.

These can be seen in the variables section along with their location on the stack (or how to reference them
through `rbp`).

In this example specifically:

```
var const char **av @ rbp-0x20
var uint64_t ac @ rbp-0x14
arg int argc @ rdi
arg char **argv @rsi
```

Through this we can see that `av` is an array of "strings" and `av` is a 64-bit unsigned integer.

Going back to the instructions before.
The arguments are stored in their respective variables.

Simply put, the instruction `mov dword [ac], edi` shows that the lower 32-bits of `rdi` or argument `argc` is stored
inside the variable `ac`.

Or more detailed - the value `rbp-0x14` (whatever that is) is treated as an address and the argument `argc` is stored
at that location.

This is the same for `av`.

The third line shows that `ac` is actually compared (`cmp`) with 3 - basically, 3 arguments have to be provided or the
program will spit out some string about two values being required and then will return using code `-1`.


Alternatively, if 3 arguments were provided it will move to location `0x6b5` -

<!-- insert 0x6b5 image -->
![0x6b5](~/Images/HackadayTutorials/0x6b5.JPG)

This block beings with the instructions:

```
mov rax, qword [av]
add rax, 8
mov rax, qword [rax]
mov rdi, rax
call atoi
mov dword [var_ch], eax
```

In these lines, the value(s) of `av` will be stored inside register `rax`.
8 will then be added (`add`) to the register `rax`. This is to increment `rax` by 1 word (64-bit system, 64 bits = 8 bytes).

This means that it will be accessing the value after the executable name. That value will then be moved (`mov`) into
register `rax` which will then be stored inside register `rdi` to use as an argument for the function `atoi()`.

<!-- man page image for atoi -->

The man page shows us that `atoi()` converts a string to an integer.

The final line shows the output of `atoi()` (typically in register `eax` for x64 - but not always) being stored
inside variable `var_ch`.

These instructions are then similarly followed by:

```
mov rax, qword [av]
add rax, 0x10
mov rax, qword [rax]
mov rdi, rax
call atoi
mov dword [var_8h], eax
```

Differences:

The third value is accessed instead of the second through the instruction `add rax, 0x10`.

`0x10` is 16 in hexadecimal notation, and by adding that to register `rax`, it is incrementing it by 2 words.

The second difference is that it stores the output inside variable `var_8h`.


The next set of instructions show:

```
mov edx, dword [var_ch]
mov eax, dword [var_8h]
add eax, edx
mov dword [var_4h], eax
mov eax, dword [var_ch]
cmp eax, dword [var_8h]
jg 0x707
```


The first two lines move `var_ch` and `var_8h` inside registers `edx` and `eax` respectively.

They are then added and then stored inside register `eax` which is then stored inside variable `var_4h` - this value will be
referenced in a later set of instructions.

The value in variable `var_ch` is then stored inside register `eax`. Which is then compared (`cmp`) with value inside `var_8h`.
This will then decide whether it will jump to 0x707 if `eax` is greater than `var_8h` (jg).

Simply put, the two values that are provided by the user will be added and stored in the variable `var_4h`. The same two values are then compared to see which is the larger value. If the value inside `var_8h` (the second argument provided) is larger, the executable will exit with return code `-1`. If `var_ch` is larger, the executable will continue onto the address `0x707`.

<!-- 0x707 block picture -->
![0x707](~/Images/HackadayTutorials/0x707.JPG)

The next set of instructions shows an instruction that has not yet been seen before in `shl dword [var_8h], 1`.

Basically, this instruction shifts the bits in the variable `var_8h`.


This value is then moved into the `eax` register (lower 32-bits of `rax`) and then compared with variable `var_ch`.

If `eax` is less than `var_ch` the executable will exit with return code `-1`, else, it will move to `0x725`.

<!-- 0x725 block picture -->
![0x725](~/Images/HackadayTutorials/0x725.JPG)


The final set of instructions consists of the value that was calculated before in `0x6b5` being used to compare to the value `0x63` or 99 in hexadecimal.

So, going from top to bottom - moves the value of `var_4h` inside register `eax` (lower 32-bits of `rax` register). That value is then substituted by the value in `var_ch` (or the first argument provided). Which is then finally compared with `0x63` (or 99 in hexadecimal).

This would mean that the third argument would be compared with `0x63`.
If the value is less `0x63`, it will quit with return code `-1`, otherwise, it will be successful and quit with return code `0`.

### Key takeaway points
* First, user needs to provide 3 arguments (including executable name).
* Secondly, the second argument needs to be greater than the third argument.
* Thirdly, the third argument shifted left by 1-bit has to be greater or equal to the second argument.
* And finally, the third argument has to be greater than `0x63` (99).

### Crack

So, to crack this we need to take all the points above into account. There can be various answers with these mathematical ones.

An example:
`./control-flow-1 199 100`

In the format:
`executable arg1 arg2`

These values tick all the boxes above.

# Loop 1
