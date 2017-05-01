### Cracking the lost password
>Can you figure out the password? The binary file should work in 64-bit Linuxes. http://sec-mooc-1.cs.helsinki.fi/password_3/ac964116-534f-488c-8cbe-3c4326d5b6d1/password_3

While writing this I noticed that `file` says the binary is statically linked but I think that's a side effect of it being packed. Jusb3 and I talked about this challenge after we had both completed the CTF and he mentioned a tool called `upx` which I didn't know about before.

```
root@kali:~/temp/pw3# file password_3 
password_3: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, stripped
root@kali:~/temp/pw3# ldd password_3
	not a dynamic executable
root@kali:~/temp/pw3# upx -d password_3
...
root@kali:~/temp/pw3# file password_3 
password_3: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=e85338b69149e7af3dc13f6e04884b3b0f338116, not stripped
root@kali:~/temp/pw3# ldd password_3 
	linux-vdso.so.1 (0x00007ffd373a3000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7d44478000)
	/lib64/ld-linux-x86-64.so.2 (0x0000556557645000)
```

After unpacking you can just use `strings` to get the flag out but interestingly you can now see that it actually is dynamically linked. The solutions below were done on the original unpacked binary. 

#### Solution 1:
The binary is stripped and at the time I didn't know how to reverse engineer it. But since all the previous challenges of the same type used strcmp to compare the passwords I decided to hook in to it and see if it was also used here. Below is the code for a very simple strcmp which prints out the parameters and then returns 0.
```c
#define _GNU_SOURCE
#include <stdio.h>

int strcmp(const char* s1, const char* s2) {
    printf("Hooked - \ns1: %s\ns2: %s\n", s1, s2);
    return 0;
}
````

Compile the above as a shared object library which can then be linked during runtime:
```
root@kali:~/temp/pw3# gcc -shared -fPIC hook.c -o hook.so
```
Link the library and run the challenge program:
```
root@kali:~/temp/pw3# LD_PRELOAD=./hook.so ./password_3
Enter the password :AAA
Hooked -
s1: AAA
s2: CorrectPasswrdAAA

You entered correct password

```
The flag was `CorrectPasswrdAAA`.


#### Solution 2:
This solution also hooks in to a library function but instead we just dump memory and look for the string there.

First we need to leak some memory addresses and we can do that by taking advantage of format strings like you would do in a [format string exploit](https://www.exploit-db.com/docs/28476.pdf).

```c
#define _GNU_SOURCE
#include <stdio.h>

int puts(const char *str){
    printf("hooked - %s", str);
    printf("%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p");
}
```
Compile and run just like in solution 1:
```
root@kali:~/temp/pw3# gcc -shared -fPIC leak.c -o leak.so
root@kali:~/temp/pw3# LD_PRELOAD=./leak.so ./password_3
Enter the password :AAAA
hooked -
You entered incorrect password0x7f32391326d3.0x7f3239114760.0x7fffffd7.0x400820.0x28.(nil).0x400820.0x7fff158614d0.0x400730.0x41414141.(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil)
```
You can see two memory addresses here that are quite close to each other, `0x400730` and `0x400820`. Let's dump all memory between those two addresses:
```c
#define _GNU_SOURCE
#include <stdio.h>

int puts(const char *str){

    printf("hooked - %s", str);
//    printf("%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p");

    int i = 0x400730;
    for (; i <= 0x400820; i++) {
        char *ptr = i;
        printf("%c", *ptr);
    }
}
```
Run it:
```
root@kali:~/temp/pw3# gcc -shared -fPIC leak.c -o leak.so
root@kali:~/temp/pw3# LD_PRELOAD=./leak.so ./password_3
Enter the password :AAA
hooked -
You entered incorrect password¸HuødH34%(tèçýÿÿÉÃDAWAVAÿAUATL%®& UH-®& SIöIÕL)åHHÁýèoýÿÿHít 1ÛLêLöDÿAÿÜHÃH9ëuêH[]A\A]A^A_Ðf.óÃHHÃEnter the password :%sCorrectPasswrdAAA
You entered correct password
```
And in the dump we find `CorrectPasswrdAAA`.
