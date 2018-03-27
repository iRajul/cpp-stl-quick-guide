#### Internal and external linkage via the static and extern keywords
	A variable with internal linkage is called an internal variable (or static variable). Variables with internal linkage can be used anywhere within the file they are defined in, but can not be referenced outside the file they exist in.

	A variable with external linkage is called an external variable. Variables with external linkage can be used both in the file they are defined in, as well as in other files.
	If we want to make a global variable internal (able to be used only within a single file), we can use the static keyword to do so:
	
```
static int g_x; // g_x is static, and can only be used within this file
```
	Similarly, if we want to make a global variable external (able to be used anywhere in our program), we can use the extern keyword to do so:
	

```
extern double g_y(9.8); // g_y is external, and can be used by other files
```

	By default, non-const variables declared outside of a block are assumed to be external. However, const variables declared outside of a block are assumed to be internal.

#### File scope vs. global scope

The terms “file scope” and “global scope” tend to cause confusion, and this is partly due to the way they are informally used. Technically, in C++, all global variables in C++ have “file scope”. However, informally, the term “file scope” is more often applied to file scope variables with internal linkage only, and “global scope” to file scope variables with external linkage.

Consider the following program:

```
global.cpp:
	

int g_x(2); // external linkage by default

main.cpp:
	

extern int g_x; // forward declaration for g_x -- g_x can be used beyond this point in this file

int main()

{

    std::cout << g_x; // should print 2

    return 0;

}
```
`g_x` has file scope within `global.cpp` -- it can not be directly seen outside of global.cpp. Note that even though it’s used in main.cpp, main.cpp isn’t seeing g_x, it’s seeing the forward declaration of g_x (which also has file scope). The linker is responsible for linking up the definition of g_x in global.cpp with the use of g_x in main.cpp.


#### Quiz
1) What’s the difference between a variable’s scope, duration, and linkage? What kind of scope, duration, and linkage do global variables have?

	Scope determines where a variable is accessible. Duration determines where a variable is created and destroyed. Linkage determines whether the variable can be exported to another file or not.

	Global variables have global scope (aka. file scope), which means they can be accessed from the point of declaration to the end of the file in which they are declared.

	Global variables have static duration, which means they are created when the program is started, and destroyed when it ends.

	Global variables can have either internal or external linkage, via the static and extern keywords respectively. 



2) What is a namespace?

* A namespace defines an area of code in which all identifiers are guaranteed to be unique. 

#### operator overloading:
	1. using friend
	2. normal function
	3. member function.


* Not everything can be overloaded as a friend function

* The `assignment (=), subscript ([]), function call (()), and member selection (->) operators` must be overloaded as member functions, because the language requires them to be.

* The following rules of thumb can help you determine which form is best for a given situation:
	* If you’re overloading assignment (=), subscript ([]), function call (()), or member selection (->), do so as a member function.
	* If you’re overloading a unary operator, do so as a member function.
	* If you’re overloading a binary operator that modifies its left operand (e.g. operator+=), do so as a member function if you can.
	* If you’re overloading a binary operator that does not modify its left operand (e.g. operator+), do so as a normal function or friend function.


* overlaod post and preincrement operator ++ and --

* dont call subscript operator on pointer to an object.
```
    IntList *list = new IntList;
    list [2] = 3; // error: this will assume we're accessing index 2 of an array of IntLists
```
* Direct, uniform and copy initialization. 

* conversion constructor which take atleast one parameter.


#### Function Overloading and float in C++: 
As per C++ standard, floating point literals (compile time constants) are treated as double unless explicitly specified by a suffix
 
 
#### Does overloading work with Inheritance?
Overloading doesn’t work for derived class in C++ programming language. There is no overload resolution between Base and Derived. The compiler looks into the scope of Derived.
 q

#### Size of array without sizeof operator
```
	int arr[6];
	int size = *(&arr + 1) - arr;
```

```
&arr ==> Pointer to an array of 6 elements.

(&arr + 1) ==> Address of 6 integers ahead as
               pointer type is pointer to array
               of 6 integers.

*(&arr + 1) ==> Same address as (&arr + 1), but 
                type of pointer is "int *".

*(&arr + 1) - arr ==> Since *(&arr + 1) points 
                   to the address 6 integers
                   ahead of arr, the difference
                   between two is 6.          
```

#### Inheritance and Composition
3 types of relationships: composition, aggregation, and association.
 
relationship - Dependencies
A dependency occurs when one object invokes another object’s functionality in order to accomplish some specific task. This is a weaker relationship than an association, but still, any change to the dependent object may break functionality in the caller. A dependency is always a unidirectional relationship.
example :  Our classes that use std::cout use it in order to accomplish the task of printing something to the console, but not otherwise.
 
 
Change access specifier of class:
you can change access specifier of base memeber the derived class would normally be able to access.

#### Multiple inheritance problem. 
1. Ambiguity in calling function with same name which are present in multiple base classes 
2. Diamond problem. (solution using virtual base.)

* Dont call virtual function from construtor and destructor.
* use override and final 
* object slicing and frakenobject. 
   ```
   int main()
   {
    Derived d1(5);
    Derived d2(6);
    Base &b = d2;
    b = d1; // this line is problematic
    return 0;
   }
   ```
   Only base class part get copied because `operator=` is not virtual by default.
* We cannot have references in vector . 

#### dynamic_cast
* dynamic_cast is not possible with protected and private inhertiance and class without virtual function.
* dynamic_cast with references throw a std::bad_cast instead of returning NULL;
* Difference b/w dynamic and static cast : https://stackoverflow.com/a/2254183/1622022


##### Reference to dynamic memory allocation :
int& foo = *(new int);

l-values are objects that have a defined memory address (such as variables), and persist beyond a single expression. r-values are temporary values that do not have a defined memory address, and only have expression scope.

References to r-values extend the lifetime of the referenced value
```
int somefcn()
{
    const int &ref = 2 + 3; // normally the result of 2+3 has expression scope and is destroyed at the end of this statement
    // but because the result is now bound to a reference to a const value...
    std::cout << ref; // we can use it here
} // and the lifetime of the r-value is extended to here, when the const reference dies
```
#### Some Quick Points

* because the void pointer does not know what type of object it is pointing to, it cannot be dereferenced directly! 
   `cout << *voidPtr << endl; // illegal: cannot dereference a void pointer`

* Function return types are not considered for uniqueness for overloading 

* all literal floating point values are doubles unless they have the ‘f’ suffix

#### How function calls are matched with overloaded functions

1) First, C++ tries to find an exact match. This is the case where the actual argument exactly matches the parameter type of one of the overloaded functions. For example:
```
void print(char *value);
void print(int value)
```

2) If no exact match is found, C++ tries to find a match through promotion.
Char, unsigned char, and short is promoted to an int.
Unsigned short can be promoted to int or unsigned int, depending on the size of an int
Float is promoted to double
Enum is promoted to int

* The default constructor allows us to create objects of the class, but does not do any initialization or assignment of values to the class members itself.

* you could initialize class variables in three ways: copy, direct, and via uniform initialization
   `Fraction six = Fraction(6);` 
   `Fraction six(6);`
   
* Initializing array members with member initializer lists
   `Something(): m_array {} // zero the member array`

* Member initializer lists allow us to initialize our members rather than assign values to them. This is the only way to initialize members that require values upon initialization, such as **const or reference members**, and it can be more performant than assigning values in the body of the constructor. Member initializer lists work both with fundamental types and members that are classes themselves.

* if we make each function return *this, we can chain the calls together


