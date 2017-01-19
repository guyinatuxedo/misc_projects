Challenges can be found at "/levels". We seet hat there is an elf named "level01". Let's try to run it.

```
level1@io:/levels$ ./level01
Enter the 3 digit passcode to enter: 426
```

Ok so it'a asking us for a 3 digit passcode. Let's look at the assembly using gdb.

```
level1@io:/levels$ gdb ./level01
```

One wall of text later...

```
(gdb) set disassembly-flavor intel
(gdb) disas main
Dump of assembler code for function main:
   0x08048080 <+0>:	push   0x8049128
   0x08048085 <+5>:	call   0x804810f
   0x0804808a <+10>:	call   0x804809f
   0x0804808f <+15>:	cmp    eax,0x10f
   0x08048094 <+20>:	je     0x80480dc
   0x0804809a <+26>:	call   0x8048103
End of assembler dump.
```

I set the syntax to intel, since it is easier to understand and it defaults to att. Looking at here we can see a cmp function that compares the eax register to the hex value 0x10f. Let's see what it is equal to using python, then try and input the value.

```
level1@io:/levels$ python
Python 2.7.9 (default, Aug 13 2016, 16:41:35) 
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 0x10f
271
>>> exit()
```

As you can see, the hex value 0x10f is equivalent to the decimal 271, which meets the format the elf asked for. Let's try it now.

```
level1@io:/levels$ ./level01
Enter the 3 digit passcode to enter: 271
Congrats you found it, now read the password for level2 from /home/level2/.pass
sh-4.3$ whoami
level2
sh-4.3$ cat /home/level2/.pass
XNWFtWKWHhaaXoKI
```

And just like that, we pwned the program.







