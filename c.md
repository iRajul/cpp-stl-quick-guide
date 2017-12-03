1. Is it fine to write “void main()” or “main()” in C/C++?


2. Difference between “int main()” and “int main(void)” in C/C++?
    So the difference is, in C, int main() can be called with any number of arguments, but int main(void) can only be called without any argument. Although it doesn’t make any difference most of the times, using “int main(void)” is a recommended practice in C.


3.  Interesting Facts about Macros and Preprocessors in C
    In a C program, all lines that start with `#` are processed by preprocessor which is a special program invoked by the compiler. In a very basic term, preprocessor takes a C program and produces another C program without any `#`.

    Following are some interesting facts about preprocessors in C.
    1) When we use include directive,  the contents of included header file (after preprocessing) are copied to the current file. Angular brackets < and > instruct the preprocessor to look in the standard folder where all header files are held.  Double quotes “ and “ instruct the preprocessor to look into the current folder and if the file is not present in current folder, then in standard folder of all header files.
    2) When we use define for a constant, the preprocessor produces a C program where the defined constant is searched and matching tokens are replaced with the given expression. For example in the following program max is defined as 100.
    3) The macros can take function like arguments, the arguments are not checked for data type. For example, the following macro INCREMENT(x) can be used for x of any data type.
```
    #include <stdio.h>
    #define INCREMENT(x) ++x
    int main()
    {
        char *ptr = "GeeksQuiz";
        int x = 10;
        printf("%s  ", INCREMENT(ptr));
        printf("%d", INCREMENT(x));
        return 0;
    }
    // Output: eeksQuiz 11
```

4) The macro arguments are not evaluated before macro expansion. For example consider the following program
```
#define MULTIPLY(a, b) a*b
int main()
{
    // The macro is expended as 2 + 3 * 3 + 5, not as 5*8
    printf("%d", MULTIPLY(2+3, 3+5));
    return 0;
}
// Output: 16
```

5) The tokens passed to macros can be concatenated using operator `##` called Token-Pasting operator.
```
#include <stdio.h>
#define merge(a, b) a##b
int main()
{
    printf("%d ", merge(12, 34));
}
// Output: 1234
```

6) A token passed to macro can be converted to a sting literal by using `#` before it.
```
#include <stdio.h>
#define get(a) #a
int main()
{
    // GeeksQuiz is changed to "GeeksQuiz"
    printf("%s", get(GeeksQuiz));
}
// Output: GeeksQuiz
```


7) The macros can be written in multiple lines using ‘\’. The last line doesn’t need to have ‘\’.
```
#include <stdio.h>
#define PRINT(i, limit) while (i < limit) \
                        { \
                            printf("GeeksQuiz "); \
                            i++; \
                        }
int main()
{
    int i = 0;
    PRINT(i, 3);
    return 0;
}
// Output: GeeksQuiz  GeeksQuiz  GeeksQuiz
```

8) The macros with arguments should be avoided as they cause problems sometimes. And Inline functions should be preferred as there is type checking parameter evaluation in inline functions. From C99 onward, inline functions are supported by C language also.
For example consider the following program. From first look the output seems to be 1, but it produces 36 as output.
```
#define square(x) x*x
int main()
{
  int x = 36/square(6); // Expended as 36/6*6
  printf("%d", x);
  return 0;
}
// Output: 36
```
If we use inline functions, we get the expected output. Also the program given in point 4 above can be corrected using inline functions.
```
inline int square(int x) { return x*x; }
36/square(6);  36/6*6
```
9) Preprocessors also support if-else directives which are typically used for conditional compilation.
```
int main()
{
#if VERBOSE >= 2
  printf("Trace Message");
#endif
}
```

10) A header file may be included more than one time directly or indirectly, this leads to problems of redeclaration of same variables/functions. To avoid this problem, directives like defined, ifdef and ifndef are used.

11) There are some standard macros which can be used to print program file (__FILE__), Date of compilation (__DATE__), Time of compilation (__TIME__) and Line Number in C code (__LINE__)

* What’s difference between “array” and “&array” for “int array[5]” ?
    Basically, “array” is a “pointer to the first element of array” but “&array” is a “pointer to whole array of 5 int”. Since “array” is pointer to int, addition of 1 resulted in an address with increment of 4 (assuming int size in your machine is 4 bytes). Since “&array” is pointer to array of 5 ints, addition of 1 resulted in an address with increment of 4 x 5 = 20 = 0x14. 


* What is NULL pointer
* sizeof NULL
* sizeof void
* sizeof void*

