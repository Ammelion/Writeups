Hi, today we have a particularly hard lab over here
A seemingly simple binary bomb...

It has a total of six phases
Easier to hardest
lets take a look on how to solve it

## Some stuff to do beforehand:
1.) make a gdbCfg file that we'll refer to gdb like this
```display/10i $rip
display/x $rbp
display/x $rsp
display/x $rax
display/x $rbx
display/x $rcx
display/x $rdx
display/x $rdi
display/x $rsi
display/x $r8
display/x $r9
display/x $r12
display/x $r13
display/x $r14
display/10gx $rsp
b phase_1
start
```
2.) make an empty answers.txt file
```touch answers.txt
```
## Phase 1 ->
Entering the phase is pretty simple just initiate the program like
```gdb ./bomb -q -x ~/gdbCfg```

now you enter the program at the very first instruction:
type ni to go to the next instruction and repeatedly press enter until it asks a string

since we dont know the answer of this, type anything and press enter once

you will, from the disassembly see that we are now approching the function called "phase 1"
underneath this is the code that checks if your answer is correct or not

lets step INSIDE the function using the "si" command

lets interpret this
```
Dump of assembler code for function phase_1:
=> 0x00005555555555a7 <+0>:	endbr64
   0x00005555555555ab <+4>:	sub    rsp,0x8
   0x00005555555555af <+8>:	lea    rsi,[rip+0x1b9a]        # 0x555555557150
   0x00005555555555b6 <+15>:	call   0x555555555ad1 <strings_not_equal>
   0x00005555555555bb <+20>:	test   eax,eax
   0x00005555555555bd <+22>:	jne    0x5555555555c4 <phase_1+29>
   0x00005555555555bf <+24>:	add    rsp,0x8
   0x00005555555555c3 <+28>:	ret
   0x00005555555555c4 <+29>:	call   0x555555555be5 <explode_bomb>
   0x00005555555555c9 <+34>:	jmp    0x5555555555bf <phase_1+24>
End of assembler dump.
```
very interesting
the debugging symbols make this a lot easier
we can see a strings not equal check being done over here, and if it fails it jumps to the bomb
this implies that the solution is a hardcoded string, particularly on the 0x555555557150 memory location

lets inspect that memory location using
```
(gdb) x/s 0x555555557150
0x555555557150:	"I am just a renegade hockey mom."
```
cool, i guess thats a solution

## Phase 2 ->
for the phase next to it, we gotta do two things

First, add your answer to the answers.txt file
Then, in the gdbCfg file i mentioned , edit the phase 1 break to a phase 2 break
that's it

run the program again through gdb
use "c" to skip through the code to directly to the second input point

once there, write anything for the input part, and jump inside the phase two function using "si"
lets view the disassembly and get "horrified"
```
Dump of assembler code for function phase_2:
=> 0x00005555555555cb <+0>:	endbr64
   0x00005555555555cf <+4>:	push   rbp
   0x00005555555555d0 <+5>:	push   rbx
   0x00005555555555d1 <+6>:	sub    rsp,0x28
   0x00005555555555d5 <+10>:	mov    rax,QWORD PTR fs:0x28
   0x00005555555555de <+19>:	mov    QWORD PTR [rsp+0x18],rax
   0x00005555555555e3 <+24>:	xor    eax,eax
   0x00005555555555e5 <+26>:	mov    rsi,rsp
   0x00005555555555e8 <+29>:	call   0x555555555c11 <read_six_numbers>
   0x00005555555555ed <+34>:	cmp    DWORD PTR [rsp],0x1
   0x00005555555555f1 <+38>:	jne    0x5555555555fd <phase_2+50>
   0x00005555555555f3 <+40>:	mov    rbx,rsp
   0x00005555555555f6 <+43>:	lea    rbp,[rsp+0x14]
   0x00005555555555fb <+48>:	jmp    0x555555555612 <phase_2+71>
   0x00005555555555fd <+50>:	call   0x555555555be5 <explode_bomb>
   0x0000555555555602 <+55>:	jmp    0x5555555555f3 <phase_2+40>
   0x0000555555555604 <+57>:	call   0x555555555be5 <explode_bomb>
   0x0000555555555609 <+62>:	add    rbx,0x4
   0x000055555555560d <+66>:	cmp    rbx,rbp
   0x0000555555555610 <+69>:	je     0x55555555561d <phase_2+82>
   0x0000555555555612 <+71>:	mov    eax,DWORD PTR [rbx]
   0x0000555555555614 <+73>:	add    eax,eax
   0x0000555555555616 <+75>:	cmp    DWORD PTR [rbx+0x4],eax
   0x0000555555555619 <+78>:	je     0x555555555609 <phase_2+62>
   0x000055555555561b <+80>:	jmp    0x555555555604 <phase_2+57>
   0x000055555555561d <+82>:	mov    rax,QWORD PTR [rsp+0x18]
   0x0000555555555622 <+87>:	xor    rax,QWORD PTR fs:0x28
   0x000055555555562b <+96>:	jne    0x555555555634 <phase_2+105>
   0x000055555555562d <+98>:	add    rsp,0x28
   0x0000555555555631 <+102>:	pop    rbx
   0x0000555555555632 <+103>:	pop    rbp
   0x0000555555555633 <+104>:	ret
   0x0000555555555634 <+105>:	call   0x555555555220 <__stack_chk_fail@plt>
End of assembler dump.
```
much longer than before

up until +29 we are setting up canary protection and reading six numbers, nothing much i guess
+34 says rsp gets compared to 1 otherwise get bombed, so the first number is 1
+62 to +78 forms a simple loop of doubling the current number and comparing to the next, simple stuff
since the total numbers are supposed to be six and we compare doubles, we can conclude that
```
1 2 4 8 16 32
```
is the next solution, classic geometric progression
onto the next!

## Phase 3 ->
verrrry logical, if you are smart enough, you might not even have to calculate anything for this one
lets see the disassembly first
```
Dump of assembler code for function phase_3:
=> 0x0000555555555639 <+0>:	endbr64
   0x000055555555563d <+4>:	sub    rsp,0x18
   0x0000555555555641 <+8>:	mov    rax,QWORD PTR fs:0x28
   0x000055555555564a <+17>:	mov    QWORD PTR [rsp+0x8],rax
   0x000055555555564f <+22>:	xor    eax,eax
   0x0000555555555651 <+24>:	lea    rcx,[rsp+0x4]
   0x0000555555555656 <+29>:	mov    rdx,rsp
   0x0000555555555659 <+32>:	lea    rsi,[rip+0x1caf]        # 0x55555555730f
   0x0000555555555660 <+39>:	call   0x5555555552c0 <__isoc99_sscanf@plt>
   0x0000555555555665 <+44>:	cmp    eax,0x1
   0x0000555555555668 <+47>:	jle    0x555555555688 <phase_3+79>
   0x000055555555566a <+49>:	cmp    DWORD PTR [rsp],0x7
   0x000055555555566e <+53>:	ja     0x555555555704 <phase_3+203>
   0x0000555555555674 <+59>:	mov    eax,DWORD PTR [rsp]
   0x0000555555555677 <+62>:	lea    rdx,[rip+0x1b22]        # 0x5555555571a0
   0x000055555555567e <+69>:	movsxd rax,DWORD PTR [rdx+rax*4]
   0x0000555555555682 <+73>:	add    rax,rdx
   0x0000555555555685 <+76>:	notrack jmp rax
   0x0000555555555688 <+79>:	call   0x555555555be5 <explode_bomb>
   0x000055555555568d <+84>:	jmp    0x55555555566a <phase_3+49>
   0x000055555555568f <+86>:	mov    eax,0x274
   0x0000555555555694 <+91>:	sub    eax,0x24c
   0x0000555555555699 <+96>:	add    eax,0x2b0
   0x000055555555569e <+101>:	sub    eax,0x7e
   0x00005555555556a1 <+104>:	add    eax,0x7e
   0x00005555555556a4 <+107>:	sub    eax,0x7e
   0x00005555555556a7 <+110>:	add    eax,0x7e
   0x00005555555556aa <+113>:	sub    eax,0x7e
   0x00005555555556ad <+116>:	cmp    DWORD PTR [rsp],0x5
   0x00005555555556b1 <+120>:	jg     0x5555555556b9 <phase_3+128>
   0x00005555555556b3 <+122>:	cmp    DWORD PTR [rsp+0x4],eax
   0x00005555555556b7 <+126>:	je     0x5555555556be <phase_3+133>
   0x00005555555556b9 <+128>:	call   0x555555555be5 <explode_bomb>
   0x00005555555556be <+133>:	mov    rax,QWORD PTR [rsp+0x8]
   0x00005555555556c3 <+138>:	xor    rax,QWORD PTR fs:0x28
   0x00005555555556cc <+147>:	jne    0x555555555710 <phase_3+215>
   0x00005555555556ce <+149>:	add    rsp,0x18
   0x00005555555556d2 <+153>:	ret
   0x00005555555556d3 <+154>:	mov    eax,0x0
   0x00005555555556d8 <+159>:	jmp    0x555555555694 <phase_3+91>
   0x00005555555556da <+161>:	mov    eax,0x0
   0x00005555555556df <+166>:	jmp    0x555555555699 <phase_3+96>
   0x00005555555556e1 <+168>:	mov    eax,0x0
   0x00005555555556e6 <+173>:	jmp    0x55555555569e <phase_3+101>
   0x00005555555556e8 <+175>:	mov    eax,0x0
   0x00005555555556ed <+180>:	jmp    0x5555555556a1 <phase_3+104>
   0x00005555555556ef <+182>:	mov    eax,0x0
   0x00005555555556f4 <+187>:	jmp    0x5555555556a4 <phase_3+107>
   0x00005555555556f6 <+189>:	mov    eax,0x0
   0x00005555555556fb <+194>:	jmp    0x5555555556a7 <phase_3+110>
   0x00005555555556fd <+196>:	mov    eax,0x0
   0x0000555555555702 <+201>:	jmp    0x5555555556aa <phase_3+113>
   0x0000555555555704 <+203>:	call   0x555555555be5 <explode_bomb>
   0x0000555555555709 <+208>:	mov    eax,0x0
   0x000055555555570e <+213>:	jmp    0x5555555556ad <phase_3+116>
   0x0000555555555710 <+215>:	call   0x555555555220 <__stack_chk_fail@plt>
```
do you see a pattern here? the code length keeps on increasing :(
anyways

+39 shows a sscanf, hmm that's a clue that can show us what data it asks for this time
lets read that address
```
(gdb) x/s 0x55555555730f
0x55555555730f:	"%d %d"
```
so it accepts two numbers? that's simple... i hope
+49 tells me that it cannot be greater than 7, alright
+59 to +76 shows that this code will jump to different places and depends upon the value of the first number i give which is kind of like switch cases in c
BUT WAIT
+116 shortens the number of cases to 5!
so five cases actually, alright

so basically whatever the answer is, is pressent at eax at +122 !
so we will move to the next instruction in eax until +122 and check the eax register with
```
(gdb) info registers eax
eax            0x25a               602
```
eax is 602 if we entered the input 0
there we go

phase 3 answer is 
```0 602```

## Phase 4 ->

do the usual steps , type any input and jump into phase 4's disassembly
```
(gdb) disas
Dump of assembler code for function phase_4:
=> 0x000055555555574b <+0>:	endbr64
   0x000055555555574f <+4>:	sub    rsp,0x18
   0x0000555555555753 <+8>:	mov    rax,QWORD PTR fs:0x28
   0x000055555555575c <+17>:	mov    QWORD PTR [rsp+0x8],rax
   0x0000555555555761 <+22>:	xor    eax,eax
   0x0000555555555763 <+24>:	lea    rcx,[rsp+0x4]
   0x0000555555555768 <+29>:	mov    rdx,rsp
   0x000055555555576b <+32>:	lea    rsi,[rip+0x1b9d]        # 0x55555555730f
   0x0000555555555772 <+39>:	call   0x5555555552c0 <__isoc99_sscanf@plt>
   0x0000555555555777 <+44>:	cmp    eax,0x2
   0x000055555555577a <+47>:	jne    0x555555555782 <phase_4+55>
   0x000055555555577c <+49>:	cmp    DWORD PTR [rsp],0xe
   0x0000555555555780 <+53>:	jbe    0x555555555787 <phase_4+60>
   0x0000555555555782 <+55>:	call   0x555555555be5 <explode_bomb>
   0x0000555555555787 <+60>:	mov    edx,0xe
   0x000055555555578c <+65>:	mov    esi,0x0
   0x0000555555555791 <+70>:	mov    edi,DWORD PTR [rsp]
   0x0000555555555794 <+73>:	call   0x555555555715 <func4>
   0x0000555555555799 <+78>:	cmp    eax,0xa
   0x000055555555579c <+81>:	jne    0x5555555557a5 <phase_4+90>
   0x000055555555579e <+83>:	cmp    DWORD PTR [rsp+0x4],0xa
   0x00005555555557a3 <+88>:	je     0x5555555557aa <phase_4+95>
   0x00005555555557a5 <+90>:	call   0x555555555be5 <explode_bomb>
   0x00005555555557aa <+95>:	mov    rax,QWORD PTR [rsp+0x8]
   0x00005555555557af <+100>:	xor    rax,QWORD PTR fs:0x28
   0x00005555555557b8 <+109>:	jne    0x5555555557bf <phase_4+116>
   0x00005555555557ba <+111>:	add    rsp,0x18
   0x00005555555557be <+115>:	ret
   0x00005555555557bf <+116>:	call   0x555555555220 <__stack_chk_fail@plt>
End of assembler dump.
```

short code? nah we can already see at +73 there is a call to some func4... hmm
before that lets see what this part has to offer

till +47 is usual stuff, we are reading 2 numbers
+49 says the first number should be below 0xe which is 14 basically
we setup edx, esi, edi to 0xe, 0 and to the first number respectively and enter func4

BUT WAIT

after func4 at +78 we see eax MUST be 0xa, THIS MEANS THAT WHATEVER THIS FUNCTION SPITS MUST BE 10!!

now disassembling func4 with
```disas func4```

```
(gdb) disas func4
Dump of assembler code for function func4:
   0x0000555555555715 <+0>:	endbr64
   0x0000555555555719 <+4>:	push   rbx
   0x000055555555571a <+5>:	mov    eax,edx
   0x000055555555571c <+7>:	sub    eax,esi
   0x000055555555571e <+9>:	mov    ebx,eax
   0x0000555555555720 <+11>:	shr    ebx,0x1f
   0x0000555555555723 <+14>:	add    ebx,eax
   0x0000555555555725 <+16>:	sar    ebx,1
   0x0000555555555727 <+18>:	add    ebx,esi
   0x0000555555555729 <+20>:	cmp    ebx,edi
   0x000055555555572b <+22>:	jg     0x555555555733 <func4+30>
   0x000055555555572d <+24>:	jl     0x55555555573f <func4+42>
   0x000055555555572f <+26>:	mov    eax,ebx
   0x0000555555555731 <+28>:	pop    rbx
   0x0000555555555732 <+29>:	ret
   0x0000555555555733 <+30>:	lea    edx,[rbx-0x1]
   0x0000555555555736 <+33>:	call   0x555555555715 <func4>
   0x000055555555573b <+38>:	add    ebx,eax
   0x000055555555573d <+40>:	jmp    0x55555555572f <func4+26>
   0x000055555555573f <+42>:	lea    esi,[rbx+0x1]
   0x0000555555555742 <+45>:	call   0x555555555715 <func4>
   0x0000555555555747 <+50>:	add    ebx,eax
   0x0000555555555749 <+52>:	jmp    0x55555555572f <func4+26>
End of assembler dump.
```

this function calls itself again and again, recursive stuff
the starting looks very much like calculating the mid of a number that will be 0xe in the first iteration

this is a binary search function, but i can see a modification
+38 and +50 shows me that this will add the mid of the number and call the function again, additive loop
otherwise if target comes out equal to the number do NOTHING but return the number

The answer we need is 10

by just simply trying numbers from 0 
we see that at 3...

Iteration one
func4(3,0,14)
3 less than mid (which is 7)

so 7+func4(3,0,6)
3 is actually the mid!

rollback and add
3+7=10

soooooo ding ding
the third solution is 
```
3 10 (you can write a python script for this but i am just lazy)
```
## Phase 5

Nowwww lets see what phase 5 has to offer

```(gdb) disas
Dump of assembler code for function phase_5:
=> 0x00005555555557c4 <+0>:	endbr64
   0x00005555555557c8 <+4>:	sub    rsp,0x18
   0x00005555555557cc <+8>:	mov    rax,QWORD PTR fs:0x28
   0x00005555555557d5 <+17>:	mov    QWORD PTR [rsp+0x8],rax
   0x00005555555557da <+22>:	xor    eax,eax
   0x00005555555557dc <+24>:	lea    rcx,[rsp+0x4]
   0x00005555555557e1 <+29>:	mov    rdx,rsp
   0x00005555555557e4 <+32>:	lea    rsi,[rip+0x1b24]        # 0x55555555730f
   0x00005555555557eb <+39>:	call   0x5555555552c0 <__isoc99_sscanf@plt>
   0x00005555555557f0 <+44>:	cmp    eax,0x1
   0x00005555555557f3 <+47>:	jle    0x55555555584f <phase_5+139>
   0x00005555555557f5 <+49>:	mov    eax,DWORD PTR [rsp]
   0x00005555555557f8 <+52>:	and    eax,0xf
   0x00005555555557fb <+55>:	mov    DWORD PTR [rsp],eax
   0x00005555555557fe <+58>:	cmp    eax,0xf
   0x0000555555555801 <+61>:	je     0x555555555835 <phase_5+113>
   0x0000555555555803 <+63>:	mov    ecx,0x0
   0x0000555555555808 <+68>:	mov    edx,0x0
   0x000055555555580d <+73>:	lea    rsi,[rip+0x19ac]        # 0x5555555571c0 <array.3471>
   0x0000555555555814 <+80>:	add    edx,0x1
   0x0000555555555817 <+83>:	cdqe
   0x0000555555555819 <+85>:	mov    eax,DWORD PTR [rsi+rax*4]
   0x000055555555581c <+88>:	add    ecx,eax
   0x000055555555581e <+90>:	cmp    eax,0xf
   0x0000555555555821 <+93>:	jne    0x555555555814 <phase_5+80>
   0x0000555555555823 <+95>:	mov    DWORD PTR [rsp],0xf
   0x000055555555582a <+102>:	cmp    edx,0xf
   0x000055555555582d <+105>:	jne    0x555555555835 <phase_5+113>
   0x000055555555582f <+107>:	cmp    DWORD PTR [rsp+0x4],ecx
   0x0000555555555833 <+111>:	je     0x55555555583a <phase_5+118>
   0x0000555555555835 <+113>:	call   0x555555555be5 <explode_bomb>
   0x000055555555583a <+118>:	mov    rax,QWORD PTR [rsp+0x8]
   0x000055555555583f <+123>:	xor    rax,QWORD PTR fs:0x28
   0x0000555555555848 <+132>:	jne    0x555555555856 <phase_5+146>
   0x000055555555584a <+134>:	add    rsp,0x18
   0x000055555555584e <+138>:	ret
   0x000055555555584f <+139>:	call   0x555555555be5 <explode_bomb>
   0x0000555555555854 <+144>:	jmp    0x5555555557f5 <phase_5+49>
   0x0000555555555856 <+146>:	call   0x555555555220 <__stack_chk_fail@plt>
End of assembler dump.
```
Smaller code, probably some scare factor wrapped within, lets dive!

examining 0x55555555730f gives that this also asks for two numbers, cool
+49 to +61 is gonna bomb, if we set the first number as 15, basically only 1 to 14 numbers are allowed as first input
```
(gdb) x/20dw 0x5555555571c0
0x5555555571c0 <array.3471>:	10	2	14	7
0x5555555571d0 <array.3471+16>:	8	12	15	11
0x5555555571e0 <array.3471+32>:	0	4	1	13
0x5555555571f0 <array.3471+48>:	3	9	6	5
```
this is the array reffered at +73
after banging my head for a day or two i find:
that this is nothing but an array traversal 15 times from the indexed number until on the last iteration the stopping number is 15 too
aka the 6th index, then sum that all up to get the second number

alright!
so if we start from the 5th index, we can reach the end goal of 15 with a total sum of 115

answer:
```5 115```


## Phase 6
For the finale we have -> 

```(gdb) disas
Dump of assembler code for function phase_6:
=> 0x000055555555585b <+0>:	endbr64
   0x000055555555585f <+4>:	push   r14
   0x0000555555555861 <+6>:	push   r13
   0x0000555555555863 <+8>:	push   r12
   0x0000555555555865 <+10>:	push   rbp
   0x0000555555555866 <+11>:	push   rbx
   0x0000555555555867 <+12>:	sub    rsp,0x60
   0x000055555555586b <+16>:	mov    rax,QWORD PTR fs:0x28
   0x0000555555555874 <+25>:	mov    QWORD PTR [rsp+0x58],rax
   0x0000555555555879 <+30>:	xor    eax,eax
   0x000055555555587b <+32>:	mov    r13,rsp
   0x000055555555587e <+35>:	mov    rsi,r13
   0x0000555555555881 <+38>:	call   0x555555555c11 <read_six_numbers>
   0x0000555555555886 <+43>:	mov    r14d,0x1
   0x000055555555588c <+49>:	mov    r12,rsp
   0x000055555555588f <+52>:	jmp    0x5555555558b9 <phase_6+94>
   0x0000555555555891 <+54>:	call   0x555555555be5 <explode_bomb>
   0x0000555555555896 <+59>:	jmp    0x5555555558c8 <phase_6+109>
   0x0000555555555898 <+61>:	add    rbx,0x1
   0x000055555555589c <+65>:	cmp    ebx,0x5
   0x000055555555589f <+68>:	jg     0x5555555558b1 <phase_6+86>
   0x00005555555558a1 <+70>:	mov    eax,DWORD PTR [r12+rbx*4]
   0x00005555555558a5 <+74>:	cmp    DWORD PTR [rbp+0x0],eax
   0x00005555555558a8 <+77>:	jne    0x555555555898 <phase_6+61>
   0x00005555555558aa <+79>:	call   0x555555555be5 <explode_bomb>
   0x00005555555558af <+84>:	jmp    0x555555555898 <phase_6+61>
   0x00005555555558b1 <+86>:	add    r14,0x1
   0x00005555555558b5 <+90>:	add    r13,0x4
   0x00005555555558b9 <+94>:	mov    rbp,r13
   0x00005555555558bc <+97>:	mov    eax,DWORD PTR [r13+0x0]
   0x00005555555558c0 <+101>:	sub    eax,0x1
   0x00005555555558c3 <+104>:	cmp    eax,0x5
   0x00005555555558c6 <+107>:	ja     0x555555555891 <phase_6+54>
   0x00005555555558c8 <+109>:	cmp    r14d,0x5
   0x00005555555558cc <+113>:	jg     0x5555555558d3 <phase_6+120>
   0x00005555555558ce <+115>:	mov    rbx,r14
   0x00005555555558d1 <+118>:	jmp    0x5555555558a1 <phase_6+70>
   0x00005555555558d3 <+120>:	mov    esi,0x0
   0x00005555555558d8 <+125>:	mov    ecx,DWORD PTR [rsp+rsi*4]
   0x00005555555558db <+128>:	mov    eax,0x1
   0x00005555555558e0 <+133>:	lea    rdx,[rip+0x3919]        # 0x555555559200 <node1>
   0x00005555555558e7 <+140>:	cmp    ecx,0x1
   0x00005555555558ea <+143>:	jle    0x5555555558f7 <phase_6+156>
   0x00005555555558ec <+145>:	mov    rdx,QWORD PTR [rdx+0x8]
   0x00005555555558f0 <+149>:	add    eax,0x1
   0x00005555555558f3 <+152>:	cmp    eax,ecx
   0x00005555555558f5 <+154>:	jne    0x5555555558ec <phase_6+145>
   0x00005555555558f7 <+156>:	mov    QWORD PTR [rsp+rsi*8+0x20],rdx
   0x00005555555558fc <+161>:	add    rsi,0x1
   0x0000555555555900 <+165>:	cmp    rsi,0x6
   0x0000555555555904 <+169>:	jne    0x5555555558d8 <phase_6+125>
   0x0000555555555906 <+171>:	mov    rbx,QWORD PTR [rsp+0x20]
   0x000055555555590b <+176>:	mov    rax,QWORD PTR [rsp+0x28]
   0x0000555555555910 <+181>:	mov    QWORD PTR [rbx+0x8],rax
   0x0000555555555914 <+185>:	mov    rdx,QWORD PTR [rsp+0x30]
   0x0000555555555919 <+190>:	mov    QWORD PTR [rax+0x8],rdx
   0x000055555555591d <+194>:	mov    rax,QWORD PTR [rsp+0x38]
   0x0000555555555922 <+199>:	mov    QWORD PTR [rdx+0x8],rax
   0x0000555555555926 <+203>:	mov    rdx,QWORD PTR [rsp+0x40]
   0x000055555555592b <+208>:	mov    QWORD PTR [rax+0x8],rdx
   0x000055555555592f <+212>:	mov    rax,QWORD PTR [rsp+0x48]
   0x0000555555555934 <+217>:	mov    QWORD PTR [rdx+0x8],rax
   0x0000555555555938 <+221>:	mov    QWORD PTR [rax+0x8],0x0
   0x0000555555555940 <+229>:	mov    ebp,0x5
   0x0000555555555945 <+234>:	jmp    0x555555555950 <phase_6+245>
   0x0000555555555947 <+236>:	mov    rbx,QWORD PTR [rbx+0x8]
   0x000055555555594b <+240>:	sub    ebp,0x1
   0x000055555555594e <+243>:	je     0x555555555961 <phase_6+262>
   0x0000555555555950 <+245>:	mov    rax,QWORD PTR [rbx+0x8]
   0x0000555555555954 <+249>:	mov    eax,DWORD PTR [rax]
   0x0000555555555956 <+251>:	cmp    DWORD PTR [rbx],eax
   0x0000555555555958 <+253>:	jge    0x555555555947 <phase_6+236>
   0x000055555555595a <+255>:	call   0x555555555be5 <explode_bomb>
   0x000055555555595f <+260>:	jmp    0x555555555947 <phase_6+236>
   0x0000555555555961 <+262>:	mov    rax,QWORD PTR [rsp+0x58]
   0x0000555555555966 <+267>:	xor    rax,QWORD PTR fs:0x28
   0x000055555555596f <+276>:	jne    0x55555555597e <phase_6+291>
   0x0000555555555971 <+278>:	add    rsp,0x60
   0x0000555555555975 <+282>:	pop    rbx
   0x0000555555555976 <+283>:	pop    rbp
   0x0000555555555977 <+284>:	pop    r12
   0x0000555555555979 <+286>:	pop    r13
   0x000055555555597b <+288>:	pop    r14
   0x000055555555597d <+290>:	ret
   0x000055555555597e <+291>:	call   0x555555555220 <__stack_chk_fail@plt>
End of assembler dump.
```
a very good attempt to make us regret our journey, unfortunately we wont :)

till +38 we get six numbers as input
next we find that the input will only take unique numbers below or equal to 5.

next we find that 0x555555559200 is called "node 1"
perhaps a reference to a data structure like a linked list
this means some sensitive data is present here
lets give it a shot
```
(gdb) x/2gx 0x555555559200
0x555555559200 <node1>:	0x0000000100000212	0x0000555555559210
(gdb) x/2gx 0x0000555555559210
0x555555559210 <node2>:	0x00000002000001c2	0x0000555555559220
(gdb) x/2gx 0x0000555555559220
0x555555559220 <node3>:	0x0000000300000215	0x0000555555559230
(gdb) x/2gx 0x0000555555559230
0x555555559230 <node4>:	0x0000000400000393	0x0000555555559240
(gdb) x/2gx 0x0000555555559240
0x555555559240 <node5>:	0x00000005000003a7	0x0000555555559110
(gdb) x/2gx 0x0000555555559110
0x555555559110 <node6>:	0x0000000600000200	0x0000000000000000
```
six nodes, neatly arranged with the last one ending with a null , SWEET!

now!
convert all the eax first parts to data in the linked list (32 bytes)
we get
```
Node 1 -->530
Node 2 -->450
Node 3 -->533
Node 4 -->915
Node 5 -->935
Node 6 -->512
```
now, lets take a look as the disassembly again
so the point is from +171 to +221 we can see that we are "reassembling" the linked list to make sure all the data points are in descending order

lets do that
```5(935)  > 4(915) > 3(533) > 1(530) > 6(512) > 2(450)```

so finally the six unique numbers that are needed to be arranged as follows
```
=> 5 4 3 1 6 2
```

and thus we've solved all the phases of the Binary Bomb Lab!
Special thanks to Xeno and Ost2 for providing so much knowledge and fun stuff to tinker with
