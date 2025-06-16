## Welcome to my first writeup!

This is a writeup on the RTFM&WTFI Lab by OpenSecurityTraining 2. Special thanks to Xeno for conducting this!

Basically our AIM is to somehow covert this set of assembly instructions into their raw, executable bytes:

```asm
mov $0xAABBCCDD, %eax
sahf
jz mylabel
and $0x31337, %eax
mylabel:
ret
```

soooo its time to read the Glorious manual that is the [Software Developers Manual for Intel 64 and IA-32 Architectures](https://ost2images.s3.amazonaws.com/Arch2001/CourseMaterials/325462-sdm-vol-1-2abcd-3abcd.pdf)

lets move through the assembly line by line

```asm
mov $0xAABBCCDD, %eax
```

i see a transfer of a 32-bit immediate value to the eax register, which is ALSO a 32 bit register soooo if we refer to the manual

![MOV instruction reference](/images/1.png)

This is the closest of a match we can get, lets go ahead and write the bytes as follows:

```asm
.byte 0xB8, 0xDD, 0xCC, 0xBB, 0xAA
```

```asm
sahf
```

from the manual we see that it copies the upper 8 bits of the AX register, also known as the AH register and into the lower bytes of the EFLAGS register affecting the status flags

![SAHF instruction reference](/images/2.png)

The result therefore for this one is trivial :p

```asm
.byte 0x9E
```

```asm
jz mylabel
```

Since the processor wont remember mere labels like these we need to use a little bit of thinking over here...... from the manual we see that it takes exactly the number of bytes we have to jump, lets see how many bytes each instruction takes

![Jump instruction reference](/images/3.png)

The next instruction in the program is an AND instruction which takes 5 bytes so:

```asm
.byte 0x74, 0x05
```

```asm
and $0x31337, %eax
```

this will, as the name says will perform an and instruction, if we refer to the manual to check for its opcode we get this row

```asm
.byte 0x25, 0x37, 0x13, 0x03, 0x00
```

![AND instruction reference](/images/4.png)

```asm
ret
```

perform a return, its simple, we have to do a near return so

```asm
.byte 0xC3
```

![RET instruction reference](/images/5.png)

Now to write the following as inline assembly in GCC we do the following:

```c
#include <stdio.h>

__attribute__((naked)) void asm_func() {
    __asm__ volatile (
        ".byte 0xB8, 0xDD, 0xCC, 0xBB, 0xAA\n"
        ".byte 0x9E\n"
        ".byte 0x74, 0x05\n"
        ".byte 0x25, 0x37, 0x13, 0x03, 0x00\n"
        ".byte 0xC3\n"
    );
}

int main() {
    asm_func();
    return 0;
}
```

That’s all for this Lab, thanks for reading!
