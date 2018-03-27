#### Struct:
A structure is a user defined data type in C/C++. A structure creates a data type that can be used to group items of possibly different types into a single type.

##### Structure padding
There is a way to minimize padding. The programmer should declare the structure members in their increasing/decreasing order of size. 

```typedef struct structd_tag
{
   double      d;
   int         s;
   char        c;
} structd_t;
```
`sizeof(structd_tag) = 16;`

compiler introduces alignment requirement to every structure. It will be as that of the largest member of the structure. i.e final size of structure should be multiple of largest data type size in structure. In above case 16 is a mulitple of 8.

#### Union:
Like Structures, union is a user defined data type. In union, all members share the same memory location. 

For example in the following C program, both x and y share the same location. If we change x, we can see the changes being reflected in y. Size of a union is taken according the size of largest member in union.

```
#include <stdio.h>
 
// Declaration of union is same as structures
union test
{
   int x, y;
};
```

#### What are applications of union?
TODO

#### Struct Hack (flexible array members)
What will be the size of following structure?
```
struct employee
{
    int     emp_id;
    int     name_len;
    char    name[0];
};
```
`4 + 4 + 0 = 8 bytes.`

And what about size of `“name[0]”`. In gcc, when we create an array of zero length, it is considered as array of incomplete type that’s why gcc reports its size as “0” bytes. This technique is known as “Stuct Hack”. When we create array of zero length inside structure, it must be (and only) last member of structure.
“Struct Hack” technique is used to create variable length member in a structure.

Let us see below memory allocation.

`struct employee *e = malloc(sizeof(*e) + sizeof(char) * 128); `
is equivalent to

```
struct employee
{
    int     emp_id;
    int     name_len;
    char    name[128]; /* character array of size 128 */
};
```
And below memory allocation

`struct employee *e = malloc(sizeof(*e) + sizeof(char) * 1024); `
is equivalent to
```
struct employee
{
    int     emp_id;
    int     name_len;
    char    name[1024]; /* character array of size 1024 */
};
```
Other advantage of this is, suppose if we want to write data, we can write whole data by using single “write()” call. e.g.

`write(fd, e, sizeof(*e) + name_len); /* write emp_id + name_len + name */ `


#### Bit Fields in C:
In C, we can specify size (in bits) of structure and union members. 
```
#include <stdio.h>
 
// A space optimized representation of date
struct date
{
   // d has value between 1 and 31, so 5 bits
   // are sufficient
   unsigned int d: 5;
 
   // m has value between 1 and 12, so 4 bits
   // are sufficient
   unsigned int m: 4;
 
   unsigned int y;
};
```

size of above structure is 8

1) A special unnamed bit field of size 0 is used to force alignment on next boundary. For example consider the following program.
unsigned int : 0;
2) We cannot have pointers to bit field members as they may not start at a byte boundary.
&date.m is not allowed
3) It is implementation defined to assign an out-of-range value to a bit field member.
4) In C++, we can have static members in a structure/class, but bit fields cannot be static.
 static unsigned int x: 5;(not allowed)
5) Array of bit fields is not allowed. For example, the below program fails in compilation.
 `unsigned int x[10]: 5;(not allowed )`
 
#### Storage Classes in C
1. auto 
2. extern
3. static
4. register 

#### Static in C
* Static variable and function
* Initialization of static variables in C
   static int i = 50;


#### Understanding “volatile” qualifier in C
The volatile keyword is intended to prevent the compiler from applying any optimizations on objects that can change in ways that cannot be determined by the compiler.
1) Global variables modified by an interrupt service routine outside the scope
2) Global variables within a multi-threaded application

Example:
```
const int local = 10;
int *ptr = (int*) &local;
printf("Initial value of local : %d \n", local);
*ptr = 100;
```

------------------------------------------------

#### Memory layout of c program:
TODO

Dynamic Memory allocation in C:
* malloc
* calloc
* realloc
* Use of realloc()
`void *realloc(void *ptr, size_t size);`

#### File management:
* fseek() vs rewind() in C
   `fseek()` sets the file position indicator of an input stream back to the beginning using rewind(). But there is no way to check whether the rewind() was successful.
   ```
   if(fseek(fp, 0L, SEEK_BEG) != 0){
   //handle error
   }
   ```
* EOF, getc() and feof() in C
getc() can return EOF if any error occurs. 
`foef()` is better choice.
```
if (feof(fp))
     printf("\n End of file reached.");
  else
     printf("\n Something went wrong.");
```

* fopen() for an existing file in write mode (wx mode like w)
* fgets() and gets() in C language
   For reading a string value with spaces, we can use either gets() or fgets() in C programming language. Here, we will see what is the difference between gets() and fgets().
   `fgets(char*, int n, FILE*);`
   `gets(char*);`
   There is no error array bound check in `gets()`;
   gets() is risky to use!
   it suffers from Buffer Overflow as gets() doesn’t do any array bound testing:
   
   * use fgets
   ```
   char str[MAX_LIMIT];
   fgets(str, MAX_LIMIT, stdin);
   ```

#### return statement vs exit() in main()
When exit(0) is used to exit from program, destructors for locally scoped non-static objects are not called. But destructors are called if return 0 is used.
Calling destructors is sometimes important, for example, if destructor has code to release resources like closing files.

   
#### little endian and big endian
TODO
   



#### Internal and external linkage: 
**external linkage** means the symbol (function or global variable) is accessible throughout your program and **internal linkage** means that it's only accessible in one translation unit.

You can explicitly control the linkage of a symbol by using the extern and static keywords. If the linkage isn't specified then the default linkage is extern for non-const symbols and static (internal) for const symbols.

Example:
```
// in namespace or global scope
int i; // extern by default
const int ci; // static by default
extern const int eci; // explicitly extern
static int si; // explicitly static

// the same goes for functions (but there are no const functions)
int foo(); // extern by default
static int bar(); // explicitly static
```




 
