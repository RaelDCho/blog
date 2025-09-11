## Bbbbloat (PicoCTF)

**Challenge**: Can you get the flag?\
Reverse engineer this [binary](https://artifacts.picoctf.net/c/47/bbbbloat).

**Challenge Instructions**: Do the usual - download in your desired directory through `wget "https://artifacts.picoctf.net/c/47/bbbbloat"`\
Make it an executable using `chmod +x <file-name>`

**Solution**: 

After downloading the binary, I analysed what sort of file it was through the `file` command.

![file](images/file.png)

It seems to be a 64-bit PIE ELF, that is dynamically linked, and stripped.

Some things to note:
- PIE: PIE stands for Position Independent Executable. This means that the executable can be loaded at any address.
- ASLR: ASLR stands for Address Space Layout Randomisation. ASLR randomises memory addresses by an application.
- Stripped: Stripped binaries are executable files that has had its symbols removed.

These factors have all been implemented for the security of the binary, and will make it a touch harder to cracking it.

After finding out the type of file, I use the `strings` command to see if the flag has been loaded as a string, and can be picked up.

![strings](images/strings.png)

Doesn't look like it. However, there are some interesting lines such as: `What's my favorite number?` and `Sorry, that's not it!`

I assume these are strings that are a spit out depending on user input.

Let's get to debugging this file.

Obviously something like `disas main` won't work, because the functions have been stripped.

![info functions](images/info_functions.png)

If we check the functions through `info functions`, we can see that only functions from external libraries.

There are many ways of completing this puzzle. But one thing to keep in mind is that, although PIE and ASLR will randomise the address so that it's difficult to find the `main` function, it will always be at an offset.

If I call the `info file` command:

![info file](images/info_file.png)

We can see the entry point to be at `0x1160` (inline with the `.text` section, because that is the code section).

If we examine the instructions from that address through `x/20i 0x1160`, we will be in the `_start` function:

![examine instructions](images/examine_instructions.png)

The line at the address `0x1188` is calling the `main` function, so the address that is loaded into `rdi` must be the address of `main`.

As noted before, `main` will always be at an offset from the entry point of the program.

If we calculate the offset from `main` minus the entry point of the program, we will be able to find out the address of `main` everytime we run the program.

**`0x1188` - `0x1160` = `0x28`**

Now, if we want, we can run a new instance of this, but know where we to find the main function, as long as we can find the entry point of the program.

I examine the first 20 instructions of main through `x/20i 0x1307` to see if there are any lines worth noting. 

![examine main](images/examine_main.png)

There isn't much until I do `x/150i 0x1307`, and see that the program calls for user input through `scanf()`, and is stored in a local variable at `rbp-0x40`.

FYI, the string that is being loaded in to the register `rdi` is at the address `0x2020`, which when we run `x/s 0x2020` shows that it is `%d` (noting that it is inputting an integer).

When we go down further down the program, we can see that the local variable at `rbp-0x40` is loaded into the lower 4 bytes of `rax` (`eax`), and compared against the value of `0x86187`.

![scanf](images/scanf.png)

After the `cmp` instruction, there is a `jne` (jump not equal) instruction to the address `0x1583`, where a string is loaded into the register `rdi`.

![cmp](images/cmp.png)

The string at this - `0x2023` - is loaded into the register `rdi`, and then there is a `puts()` call. When examining that hex as a string (`x/s 0x2023`) it turns out to be: `Sorry, that's not it!`.

The program then does the usual stack check and `return 0`.

![return](images/return.png)

This proves that our input when running the program has to be equal to `0x86187`. When we print this out as a decimal through `print /d 0x86187`, we get the result `549255`.

Now, we can clear all breakpoints and run the program as normal to get:

![cut the bloat](images/cut_the_bloat.png)

**`cut the bloat`**