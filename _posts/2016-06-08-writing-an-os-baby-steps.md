---
layout: post
title: "Writing an OS: Baby Steps"
---

![Baby Steps]

_(Taken from the excellent movie, [What About Bob?])_

This tutorial is intended to walk you through writing your own _very_ simple
operating system in assembly. I originally found the basis of this tutorial on the [OSDev wiki], but it didn't have any explanation to what was going on or why, so I decided to make this tutorial. We'll go over the basics of the booting process
and what tools you'll need to operate. 

Most popular operating systems like OSX, Linux, or Windows contain drivers and provide interfaces to the hardware, ensure a certain level of safety and security, keep the processes from fighting each other, and provide essential libraries for programs to use the computer. However, ours won't be nearly as complicated. :) By the end of this tutorial, you should have an operating system that prints a message to the screen!

I attempted to explain everything as simple as possible, but if I made any errors
or you'd like to suggest changes to my article, feel free to [make an issue] on
Github or email me.

# Table of Contents
* TOC
{:toc}

# Requirements

This tutorial assumes that you already have a Linux installation. If you don't
you can still follow along by [installing Ubuntu in a virtual machine]

You'll need `nasm`, `build-essentials`, and `qemu`.

`sudo apt-get install nasm build-essentials qemu`

* `nasm` is an assembler, which translates assembly code into binary code that the
  computer can directly execute, unlike code that involves _actual_ words.
* `build-essentials` installs many programs and compilers required for building other
  programs. We will use `make` mostly to automate building the operating system
  and running it.
* `qemu` is an easy to use virtual machine that emulates a computer, so that we
  don't have to accidentally screw up our computer or reboot constantly to
  develop our operating system. We can test the code out directly on the virtual
  machine that the qemu software provides.

# The Booting Process

When a computer is first powered on, it loads the BIOS (Basic Input Output System) from a special permanent flash memory chip on the motherboard. It functions as a rudimentary library that can access and modify hardware on the computer. The BIOS also performs a POST (Power On Self Test) check to make sure that all systems are running fine. It then locates the MBR (Master Boot Record, also known as boot sector) which is 512 bytes long and always found at the very start of the bootable media like a hard drive, floppy, dvd, or usb drive. After finding it, the BIOS executes the code in the MBR in Real Mode. Real Mode makes the processor use 16-bits even if the processor is 32-bits or 64-bits because of legacy reasons. This mode can only access 1MB of memory because of the smaller bit size. For our purposes, Real Mode is fine because it's the only mode which allows us to use BIOS functions and we don't have much code.

A byte is 8 bits, which are numbers that can be either 1 or 0 and are encoded as binary numbers. For example: 10010101 is a byte. Computers use bits and bytes to deal with instructions and data because that format is analogous to off and on. A computer is basically an incredibly dense electronics circuit, so using binary to program it works out well to map most directly to the hardware.

The MBR has many functions located inside it. It can hold the locations and information of different partitions for the hard drive and it also holds the code that the computer executes. Because the MBR is only 512 bytes and most operating systems don't fit into that size (Linux and Windows contain millions of lines of code), many operating systems use a bootloader that can load the operating system kernal code from different filesystems and execute it, while finishing the set up of the computer.

Our little operating system kernel, however, will fit inside the 512 bytes, so it doesn't need a bootloader to load more code from the disk.

What I'll do is show you some code and we'll deconstruct it bit by bit. Don't worry if you don't understand everything; this is meant to just give you some knowledge about how computers work. I'll also provide the commands and information so that you can follow along on your own computer.

# Finally, Some Code

Type this snippet into your favorite text editor as `boot.asm`:

{% highlight nasm linenos %}
; boot.asm
hang:
    jmp hang

    times 510-($-$$) db 0

	; Anything after a semicolon on that line is a comment

    db 0x55
    db 0xAA
{% endhighlight %}

* `hang:` is just a named marker in the code
* `jmp hang` means jump to the hang marker
	* This makes an infinte loop
* `times 510-($-$$) db 0` is nasm syntax for fill the rest of the bytes up with zeroes
	* We don't use `512` because the two `db` commands afterward store two bytes at the end
* `0x55` and `0xAA` are 'magic bytes' that tell the BIOS, "Yes, this is a executable MBR"

Enter this into the command line to assemble the file into a binary file that the computer can actually execute:

{% highlight console %}
$ nasm -f bin boot.asm boot.bin
{% endhighlight %}

* `-f bin` ensures that nasm assembles it into the binary format instead of something like elf which is used for general purpose programs in Linux
* `boot.asm` is the source code assembly file that nasm is trying to assemble
* `boot.bin` is the output file name

Then we start qemu with this file:

{% highlight console %}
$ qemu-system-x86_64 -fda boot.bin
{% endhighlight %}

* `qemu-system-x86_64` is used because we want to use the version of qemu that provides a 64-bit computer
* `boot.bin` tells qemu to use boot.bin as bootable media

You should see this screen when qemu starts up, if your program was successful.

![Qemu Step One]

So, what this program does is make the computer go into an infinite loop and hang. Not too bad, right?

# Shortening the Build Process

That was a lot of commands you typed in the command line earlier just to get your operating system to run. Let's make it shorter. 

We'll do this with `make`, a program that is for setting up build toolchains for almost anytype of compilation.

First, let's make a file named `Makefile` and put this into it:

{% highlight make linenos %}
boot.bin:
    nasm -f bin boot.asm -o boot.bin

qemu: boot.bin
    qemu-system-x86_64 boot.bin

clean:
    rm *.bin
{% endhighlight %}

The values before the colons are names for the list of commands that come afterwards. This way you can type `make clean` and `make` will execute `rm *.bin` for you, which removes all of the assembled files.

The values that come after the colon are dependencies. So when you type `make qemu`, `make` will execute `boot.bin`'s commands (`nasm -f bin boot.asm -o boot.bin`) before it executes `qemu-system-x86_64 boot.bin`.

# Printing to the Screen

Change your `boot.asm` assembly file, so that it looks like this:

{% highlight nasm linenos %}
; boot.asm
mov ax, 0x07c0
mov ds, ax

mov ah, 0x0
mov al, 0x3
int 0x10

mov si, msg
mov ah, 0x0E

print_character_loop:
    lodsb

    or al, al
    jz hang

    int 0x10

    jmp print_character_loop

msg:
    db 'Hello, World!', 13, 10, 0

hang:
    jmp hang

    times 510-($-$$) db 0

    db 0x55
    db 0xAA
{% endhighlight %}

I know this looks daunting! We'll go through it line by line though.

I've separated the code by chunks that correspond with the surrounding
instructions. We'll go over each chunk and the instructions used, so
that we can figure out how the operating system goes about printing to the screen.

If you look at some of these instructions, they have certain pieces information behind them. These are called operands. In NASM syntax, the right operand is the source operand and the left one is the destination operand. The letter operands refer to registers in the CPU, which are special places that hold bits of information the computer can operate on.  

`mov` is an instruction that moves data around. It can move bytes from
  register to register and from locations in the code to registers.

Let's use this knowledge to figure out our first chunk:

{% highlight nasm linenos %}
mov ax, 0x07c0
mov ds, ax
{% endhighlight %}

In the first line, we move the value `0x07c0` into the register `ax`. `0x07c0` is
a hex value, which is a different number format and more convenient for assembly programmers to use than
raw binary numbers of `1`s and `0`s.

Then we copy the value from register `ax` to register `ds`. You might ask,
"Nick, why don't you just copy `0x07c0` directly into register `ds`?" Well,
`ds` is a very special register. It stands for data segment, and for some
reason known only to Intel developers, it can only have values transfered to it
from other general purpose registers.

# What is a segment?

Segmentation is a special feature of Real Mode, which Intel made to keep their
16-bit processor, and allow users to be able to access more memory. When a computer is 16-bit, only 16-bits are used in the addressing
lines, which call data storage devices with an addresses to get certain values stored
there. Because computers use binary, a 16-bit addressing scheme would mean that there are only 2^16
available slots in which to store a byte. Those crafty Intel developers
implemented segementation to allow the computer to access 20 bits worth of
addressing space. 

However, this comes at a cost of using the segmentation
feature. To keep 16-bit compatibility, the processor has to use two registers to
store the _segment_ and the _offset_. The register that holds the segment is
multiplied by `0x10`, which is what we humans use), to add another zero to the end of the hex number. Multiplying by `0x10` translates to adding four zeros to the end of the number in binary. This means that the 16 bit number is now 20 bits!
The offset is then added to the segment address to get the actual location.
So, when the offset is added to the segment, the address can change those four zero bits at the end to any number to access all the addresses in a 20-bit address space.

A segmented address is generally referred to in this format
`segment:offset`. 

# Back to Our Code

{% highlight nasm linenos %}
mov ax, 0x07c0
mov ds, ax
{% endhighlight %}

Now you should see that this chunk of code loads `0x07c0` into register
`ds`, or data segment, for segment addressing. But, why would we need this?
Some BIOS functions require the location of the code stored into the `ds` register to access it later. The code is located at `0x07c0:0x0000` always because that's where the BIOS loads the MBR every time.

Here's another chunk:

{% highlight nasm linenos %}
mov ah, 0x0
mov al, 0x3
int 0x10
{% endhighlight %}

Here we encounter `int`, another new instruction. It stands for interrupt
and it interrupts the CPU and calls a certain piece of
code referred to as an interrupt handler. Usually that interrupt handler uses pieces of code in registers to do 
operations like taking the character in a register and printing it to the screen. In our case, the interrupts are already mapped to functions that the BIOS
has setup for us earlier.

`int` instructions only take one operand and that refers to the interrupt
number. `0x10` is the BIOS interrupt that manages video services like writing
characters to the screen, clearing it, setting the video mode and size, among
other functions depending on the value stored in `ah`. The numbers that affect
operation of the BIOS functions are usually
just chosen without rhyme or reason, so don't be afraid to look up information
about the BIOS interrupt you're using online.

* `0x0` in register `ah` refers to setting the video mode and size. In the second line, we use
* `0x3` in register `al` tells `int 0x10` that the video size should be 80 characters by 25
characters.

These lines get us set up for the character printing loop:

{% highlight nasm linenos %}
mov si, msg
mov ah, 0x0E
{% endhighlight %}

In the first line, we move the address of our message we want to print into the
`si` register. We use `si` because the `lodsb` instruction uses the
segemented address `ds:si` to load a byte from that location.

`mov ah, 0x0E` uses the `ah` register again. It tells the `0x10` interrupt we
use in the loop to allow us to utilize the interrupt for printing characters to
the screen. This is referred to as teletype output because it emulates the
functionality of a teletypewriter, which is similar to how a typewriter
operates, but able to send the text to a computer or printer.

Finally, we're at the part where we actually print
characters to the screen:
{% highlight nasm linenos %}
print_character_loop:
    lodsb

    or al, al
    jz hang

    int 0x10

    jmp print_character_loop
{% endhighlight %}

The `lodsb` instruction loads a byte from the segmented address `ds:si` and moves the `si`
  register onto the next byte. We want to load bytes from the `msg:`
  location because that's where our characters for the message we want to
  print are stored. Conveniently
  enough, each byte can store one English letter or some punctuation under the
  ASCII standard (American Standard Code for Information
  Interchange).

`or al, al` performs an `or` of the `al` register against itself. An OR
instruction compares each bit of the first operand against the
corresponding bit of the other operand. Then if either of the bits are 1, the
destination operand's bit in that location is changed to 1. 

So if we were comparing `101` and `010`, we would look at the first bits `1` and `0`. 

* Since one of the bits is one, The first bit is kept as 1 in the destination
operand. 
* Then we look at the second bits, `0` and `1`. There's another 1, so
we change the second bit of the destination operand to `1`. 
* Finally, we
compare `1` and `0` from the third bits and see that there is a one, so we
change the third bit also to `1`. 

Our final destination operand is `111`. 

The string terminates with a NULL character, which is a zero byte in binary
under the ASCII standard. A `0` compared against a `0` would be a zero.
So we use the `or` instruction to check if the string has ended. 

{% highlight nasm linenos %}
    or al, al
    jz hang
{% endhighlight %}

So, why are these two instructions together?

Well, there's a neat thing that the processor does. If the return value from
an operation is `0`, the processor will set the zero flag. And, if the zero
flag is set, the `jz` instruction executes. `jz` stands for jump if zero. So if
the string has ended, the processor will jump to the operand, which is `hang:`'s
location to hang the processor instead of looking for more characters that
aren't there in the string.

Now, we're finally at the last parts of the loop:

{% highlight nasm linenos %}
     int 0x10
 
     jmp print_character_loop
{% endhighlight %}

If you remember from earlier, `0x0E` is in the `ah` register. This means that we
can use `int 0x10` for printing characters to the screen that are stored in
`al`. 

* `int 0x10` prints the character to the screen
* `jmp print_character_loop` jumps to the start of the loop to print another
  character if the string has not ended

In summary, the loop:

1. Loads a character and moves the address to the next character
2. Checks if the character is zero and if it is, makes the processor hang
3. If the character isn't zero, it prints the character
4. And goes to the start of the loop

We've just got the final chunk left:

{% highlight nasm linenos %}
msg:
    db 'Hello, World!', 13, 10, 0
{% endhighlight %}

The `db` command says store these values at the `msg:` address. What the comma
means is also store these decimal numbers as bytes too.

* `13` translates to carriage return or `\n` in ASCII. This moves our cursor to
  the next line
* `10` translates to new line or `\r` in ASCII. This moves our cursor to the
  start of the line
* `0` tells our program that this is the end of the string

Both of these values are legacies left over from teletypewriters.

This operating system we made wasn't _that_ terrible when we broke it up, right?
Programming an OS is not something to be taken lightly (we haven't even gotten
to the hard parts yet :D) as you can see. Eventually we'd be able to use a higher level language like C or Rust, but that's for another tutorial. I'll leave some exercies and readings for you if
you'd like to learn more.

# Exercises

1. Print something else to the screen

2. Add two numbers together

3. Sum up the numbers from 1-100 and print the solution to the screen

4. Print the contents of some memory address

5. Try and read something from the disk using the BIOS

6. Get keypresses from the BIOS

# Further Reading

*Useful Information for Extending Our Operating System*

* [Bare Bones - OSDev Wiki](http://wiki.osdev.org/Bare_Bones) - This goes over
  using C or C++ to make your operating system kernel and boot it under an
  already made bootloader like GRUB, but like the article title says, this is not a hand-holding tutorial
* [Category:Babystep - OSDev Wiki](http://wiki.osdev.org/Category:Babystep) -
  The first two steps along with many other tutorials are what I based this
  tutorial on
* [Makefiles - Mrbook's Stuff](http://mrbook.org/blog/tutorials/make/) is a
  great example tutorial on how to use `Makefiles` 
* [BIOS interrupt call -
  Wikipedia](https://en.wikipedia.org/wiki/BIOS_interrupt_call) - A relatively good list on BIOS interrupts you can use

*Different Operating Systems*

* [A minimal x86 kernel - Writing an OS in
  Rust](http://os.phil-opp.com/multiboot-kernel.html) - These tutorials are excellent and very high quality, but in my opinion assume you have a certain amount of knowledge
* [MikeOS - simple x86 assembly langauge operating
  system](http://mikeos.sourceforge.net/) - "MikeOS is an operating system for x86 PCs, written in assembly language. It is a learning tool to show how simple 16-bit, real-mode OSes work, with well-commented code and extensive documentation."

*More OS Theory*

* [Memory Translation and Segmentation - Gustavo
  Duarte](http://duartes.org/gustavo/blog/post/memory-translation-and-segmentation/) - Goes into greater depth about segementation
* [The Kernel Boot Process - Gustavo
  Duarte](http://duartes.org/gustavo/blog/post/kernel-boot-process/) - If you want to look a little into how the Linux kernel boots, or any operating system kernel, really, this is for you
* [How Computers Boot Up - Gustavo
  Duarte](http://duartes.org/gustavo/blog/post/how-computers-boot-up/) - More indepth information about how computer boot
* [The Master Boot Record (MBR) and What it Does](http://www.dewassoc.com/kbase/hard_drives/master_boot_record.htm) - This tutorial goes into greater depth on the MBR and what the computer does with it
* [X86
Assembly/Bootloaders](https://en.wikibooks.org/wiki/X86_Assembly/Bootloaders) -
This is a good theory tutorial on how bootloaders work

[Baby Steps]: http://127.0.0.1:4000/images/babysteps/book.jpg
[What About Bob?]: http://www.imdb.com/title/tt0103241/
[OSDev wiki]: http://wiki.osdev.org/Babystep1
[make an issue]: https://github.com/TutorialsByNick/TutorialsByNick.github.io/issues
[installing Ubuntu in a virtual machine]: {% post_url 2016-05-25-how-to-install-ubuntu-in-a-virtual-machine-on-windows %}
[Qemu Step One]: http://127.0.0.1:4000/images/babysteps/qemu-step-one.png
