# How to create a simple C shared object library

## Code

In this tutorial, we'll build the source code gradually. The final version of the code is in `examples/main.c`, `src/` and `inc/`.

To start, the code will be very simple, and use a function from a library that is not yet implemented:

```c
// examples/main.c
#include <stdio.h>

int main(int* argc, char** argv) {
    tutorial_print("Hello, world!");
    return 0;
}
```

## Compilation

When `gcc` is called, such as using `gcc -o a.out main.c`, it first compiles the code, creating an object file. This object file can be generated using the command

<!-- Compile to object code command -->

```
gcc -c examples/main.c
```

The `-c` flag tells gcc to compile and assemble, but not link. This command will generate a `main.o` file. To take a look at it's symbols, use

```
nm main.o
```

You'll see something like

```
0000000000000000 T main
                 U tutorial_print
```

This shows that there is an unresolved dependency to the tutorial_print function. This is also noted by the warning gcc gives us. The symbols that are unresolved at the compilation step will need to be added by the linker, that's why this causes a warning, and not an error.

Of course, before linking, we'll need to write and compile the library function:

```c
// src/libtutorial.c
#include <stdio.h>

void tutorial_print(const char* message) {
    printf("=== TUTORIAL PRINT: ===\n");
    printf("Message: %s\n", message);
    printf("=======================\n");
}
```

Aditionally, to remove the warning from the compilation of our `main.c` example, we can create a header file to include the definition of our library function in our main function:

```c
// inc/libctutorial.h
#include <stdio.h>

void tutorial_print(const char* message);
```

Now we update our source files to include this header:

```c
// src/libtutorial.c
#include "libctutorial.h"

void tutorial_print(const char* message) {
    printf("=== TUTORIAL PRINT: ===\n");
    printf("Message: %s\n", message);
    printf("=======================\n");
}

```

```c
// examples/main.c
#include "libctutorial.h"

int main(int* argc, char** argv) {
    tutorial_print("Hello, world!");
    return 0;
}

```

Now if we recompile our main program, we'll get no warning. However, we need to specify the path to the header file:

```
gcc -c examples/main.c -I inc/
```

To compile the library, use the following command:

```
gcc -o libtutorial.so -fpic -shared src/libtutorial.c
```

The `-fpic` flag tells the compiler to position-independent code (PIC) suitable for use in a shared library. The `-shared` flag tells the compiler to create a shared object.

> Note: the file name following the `-o` flag must be named `lib<library-name>.so`.

Again, it is possible to see that this library defines the `tutorial_print` function with the `nm` command:

```
nm libtutorial.so
```

Which contains this line in it's output:

```
0000000000001139 T tutorial_print
```

## Linking

After the compilation step, the linker will look at all the unresolved symbols in the object files and resolve them by looking for them in libraries. This can be done with the command

<!-- Link command -->

```
gcc -o main main.o -ltutorial -L.
```

In this command, `-l<library-name>` specifies the library name, and `-L<library-path>` specifies the path to the library (`.so` file).

This command produces an executable `main` file. However, upon execution, this program might throw an error:

```
$ ./main
./main: error while loading shared libraries: libtutorial.so: cannot open shared object file: No such file or directory
```

This happens because the shared object library needs to be loaded at run time, so it needs to be located at run time. One way to allow for the library to be located is to set the `LD_LIBRARY_PATH` environment variable to point to the library path:

```
$ export LD_LIBRARY_PATH=$(pwd):$LD_LIBRARY_PATH
$ ./main
=== TUTORIAL PRINT: ===
Message: Hello, world!
=======================
```
