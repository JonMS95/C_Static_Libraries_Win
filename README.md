# Static Libraries in C (for Windows)

While working with the C/C++ programming language, it's common for developers to work with some files called libraries.
These files are pieces of code that have already been compiled and that are ready to be linked to our main code.

There are two kind of libraries in the C/C++ standard: **static** and **dynamic** libraries:
- **Static libraries** are the type of libraries that are attached to the main executable file at linking process. The file is part of the final executable file, so
the code is faster. As a con, the code is a bit heavier, as the library file is included.
- On the other hand, **dynamic libraries** are attached at runtime, it's to say, they are somehow related to the main executable file after the executable file has already been generated. As a pro, more than one file could be using these files at the same time, but they will not be part of the final executable file, making it
slightly slower.

In this short tutorial, we are only going to cover the static kind of libraries. We will leave the dynamic ones for another time.

## Static libraries

Let's say we want to do some arithmetic operation, such as addition or subtraction, but for any reason, we don't know yet which of them we want to finally perform.
Anyway, we are 100% sure that we want to do an operation that takes two integer operands and returns a result (as an integer too). Thus, we aleady know that our
generic function's prototype should look like this:

`
int function(int a, int b);
`

If we want to pass parameters to the final executable file by command line, we should then add dome input parameters to the main function in our *main.c* file.
Also, there should be some reference to the function it's going to be calling if the correct number of arguments are being introduced on the command line when
calling the executable file. Therefore, the function doesn't need to be necessarily defined, but the function prototype does [^1], so the next two files are being created:

[^1]: Note that all the fucntions that are going to be introduced in the executable when linking the files have to follow the same pattern, it's to say, they must all have the same function prototype.

### functionPrototype.h

```
#ifndef FUNCTION_PROTOTYPE_H
#define FUNCTION_PROTOTYPE_H

int function(int a, int b);

#endif
```

### main.c

```
#include <stdlib.h>
#include <stdio.h>
#include "functionPrototype.h"

#define ARGUMENT_NUMBER_MESSAGE "Only two arguments should be passed on the command line!\r\n"

int main(int argc, char** argv)
{
    if(argc == 3)
    {
        int x = atoi(argv[1]);
        int y = atoi(argv[2]);
        int z = function(x, y);

        printf("%d\r\n", z);
    }
    else
    {
        printf(ARGUMENT_NUMBER_MESSAGE);
    }

    return 0;
}
```

At this point, the *main.c* can already be compiled, so we can execute the following command:

**gcc -c main.c -o main.o**

We can forget about the object (.o) file we have just created temporarily. We are now going to generate the functions as well as their libraries.

## Generating the libraries

At the beginning of this tutorial, we said that addition and subtraction operations were willing to be performed. Let's define the source code for both of them:

### sumFunction.c

```
int function(int a, int b)
{
    return (a + b);
}
```

### subtractFunction.c

```
int function(int a, int b)
{
    return (a - b);
}
```

Note that both functions have the same function prototype.

Now, we should first compile both of the recently created files, as the librray needs those to be generated:

**gcc -c sumFunction.c -o sumFunction.o**

**gcc -c subtractFunction.c -o subtractFunction.o**

We are now ready to generate the library files. Let's execute the following lines:

**ar rcs libsumFunction.lib sumFunction.o**

**ar rcs libsubtractFunction.lib subtractFunction.o**

It’s worth paying some additional attention to the statement that are part of the command above:
- **ar** stand for archive, as it’s what we are going to generate.
- **rcs** is the combination of three parameters:
  - **r**: replace. It means it’s going to replace any other files that exist within the current folder with the same name.
  - **c**: create. We are going to create an archive that doesn’t already exist.
  - **s**: index (no idea why it’s represented with the “s” character). It means that an index of symbols is going to be generated in order for other files to easily find them.
- **libFileName.lib**: well, it’s actually called libsumFunction.lib or libsubtractFunction.lib in this case, but what we are trying to tell here, is that it does always have to follow the described pattern[^2].
- **fileName.o**: it’s sumFunction.o or subtractFunction.o in this case, but same as before, the purpose is to understand the general case: a reference to the source code we are going to generate the library from.

[^2]: It will get specially relevant at linking time. On top of that, note that the extension of the file is *.lib* for Windows operating systems files, whereas it's *.a* for Linux OS.

## Link process and executable files generation

Until this point, we have already generated the *main.o* files as well as the library files we are going to use. Okay, so let's generate the final executable files:

**gcc main.o -L. -lsumFunction -o sumProg**

**gcc main.o -L. -lsubtractFunction -o subtractProg**

Again, some parameters need a detailed explanation for a better understandability:
- **main.o**: it’s the object file that’s been generated first using main.c as source code.
- **-Ldir**: it’s -L. in this case. This flag tells the linker that flags may be in the current “.” directory. If for example, the library existed in a “testDir” directory, the command would be written as “-LtestDir”.
- **-lfileName**: it’s -lsumFunction in this case. Note that in order to reference the library, we have removed the “lib” substring from the beginning, as well as the “.lib” extension. Also, check that the extension is suitable for the OS we are working at, so it should be .lib for Windows, or .a for Linux.
- **-o**: it tells which the output is going to be.
- **prog**: or sumProg in our case, is the name of the executable file that’s the output of the command.

Once we’ve done all of this, the executable file may be used by writting some command line parameters, so (once we are working with the command line) if we type:

**sumProg.exe 6 4**

**subtractProg.exe 6 4**

The output will be the sum or subtraction of the provided numbers (10 for the addition, 2 for the subtraction). As we only allow two numbers to be passed as input parameters apart from the name of the executable, another message will be displayed in that case, telling the user that only two additional parameters can be used with the current program[^3].
[^3]: keep in mind that the name of the executable file is also an input parameter.
