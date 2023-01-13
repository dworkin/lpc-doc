NOTE:
This file is a work in progress, so when in doubt look at the source code.
It should give you the general idea, but it still needs a lot of love.

# The LPC Reference Manual

 Copyright © 1995 - 1997 Dworkin B.V.  This document has been released
 into the public domain.

## 1. Introduction

### 1.1. Purpose

This document is a formal description of the LPC programming language.
The LPC programming language is derived from C and named after its primary
creator, Lars Pensjö.  Several dialects of LPC exist; the programming
language described here is the one used in *Dworkin's Game Driver*
(DGD) version 1.7.

### 1.2. History

## 2. Environment

## 3. Language

In the syntax notation of this document, syntactic categories are
indicated by *italic* type.

### 3.1. Lexical elements

### 3.1.1. Tokens

A *token* is the minimal lexical element of the language.  The
categories of tokens are:
*[identifiers](#identifiers)*,
*[keywords](#keywords)*,
*[constants](#constants)*,
*[operators](#operators)* and
*[punctuators](#punctuators)*.  Tokens are separated by
white space: blanks, horizontal and vertical tabs, newlines, carriage returns,
form feeds and comments.

### 3.1.2. Comments

A comment is a sequence of characters starting with the characters `/*`
and terminated by the characters `*/`, or a sequence of characters that
starts with // and terminated by a new line.  All characters in between are
part of the comment.  Comments count as whitespace separating tokens, and
may not be nested.  The sequence `/*` does not start a comment if it
is part of a string.  **Note: hydra does not support // style comments.**

Examples:

```
	/* This is 
		comment */
	// This is another comment
```

### 3.1.3. Identifiers {#identifiers}

An identifier is an arbitrarily long sequence of letters, digits and
underscores.  To be a valid identifier in LPC, the first character 
must be a letter or underscore.  Only the first 1023 characters of an
identifier are significant.  Note letters are case sensitive, *N5* is not
the same as *n5\"*.

### 3.1.4. Keywords {#keywords}

The following identifiers are reserved for use as keywords in the language:

`atomic`
`break`
`case`
`catch`
`continue`
`default`
`do`
`else`
`float`
`for`
`function`\*
`goto`\*
`if`
`inherit`
`int`
`mapping`
`mixed`
`new`
`nil`
`nomask`
`object`
`operator`
`private`
`return`
`rlimits`
`static`
`string`
`switch`
`try`
`varargs`
`void`
`while`

**Note: dgd does not support goto currently.**  
**Note: function is only available if compiled with the -DCLOSURES option.**

### 3.1.5. Constants {#constants}

A *constant* is either an
*[integer constant](#integer-constants)*,
*[floating constant](#floating-constants)*,
*[string constant](#string-constants)* or
*[character constant](#character-constants)*.

### 3.1.5.1. Integer Constants {#integer-constants}

An *integer constant* is a *decimal constant*, *octal constant*
or *hexadecimal constant*.

A sequence of digits is taken to be a *decimal constant* (base ten)
unless it begins with `0`.  A sequence of digits starting with `0`
and not including `8` or `9` is an *octal constant*
(base 8).  A sequence of digits and letters in the range of `A - F`,
preceded by `0x` or `0X` is a
*hexadecimal constant*. (base 16, where `A` through `F`
have the decimal values 10 through 15).  **Note: the letters can be upper or lower case in this instance.**

*Integer constants* are represented by 32 bit 2's complement arithmetic,
and range from -2147483648 through 2147483647.  It is an error to specify a
*decimal constant* that is too large to be represented.  *Octal constants*
and *hexadecimal constants*, when too large, are
truncated to the lower 32 bits.

For instance, the number forty-seven can be written as 47, 057 or 0x2f.
There is a problem particular to LPCd with regard to -2147483648.  Since
LPCd, unlike C, has no unsigned type, the decimal version of this number
(which is scanned as - 2147483648) is the negation of a number too large
to be represented.  - and 2147483648 cannot be read as a single token,
because then 1-2147483648 would be scanned as two consecutive integers.
This number must be specified by an octal or hexadecimal constant.

### 3.1.5.2. Floating constants {#floating-constants}

A *floating constant* consists of an integer part, a decimal point, a
fraction part, and an exponent part, consisting of an `e` or `E`
and an optionally signed integer exponent.  Either the integer part or the
fraction part, but not both, may be missing.  Either the decimal point and
fraction part, or the exponent part, but not both, may be missing.

*Floating constants* represent a floating point number in the range of
\-1.79769313485E308 through 1.79769313485E308, the smallest possible number
being -2.22507385851E-308.  It is an error to specify a *floating constant*
that is too large to be represented.  Numbers which are too small are
flushed to zero.

### 3.1.5.3. String constants {#string-constants}

A *string constant* is a sequence of zero or more characters enclosed
in double quotes.  All characters, with the exception of newline, can be used in
a *string constant*. A backslash character `\` introduces an
escape sequence, which consists of at least 2 and at most 4 characters
(including the backslash), and which is translated into a single character in
the string.  The following escape sequences can be used:

```
	\a = 007 (bell)			\o
	\b = 010 (backspace)		\oo
	\f = 014 (form feed)		\ooo
	\n = 012 (newline)		\xh
	\r = 015 (carriage return)	\xhh
	\t = 011 (horizontal tab)	\c
	\v = 013 (vertical tab)
```

The value of `\a`, `\b`, `\f`, `\n`, `\r`,
`\t` and `\v` is the octal value shown.  `\o`,
`\oo` and `\ooo` constitute an integer of at most 3 octal
digits, the octal value of which specifies a single character.  `\xh` and
`\xhh` constitute an integer of at most 2 hexadecimal
digits, the hexadecimal value of which specifies a single character.  All other
escape sequences `\c` specify the character `c` (any character
except `a`, `b`, `f`, `n`, `r`, `t`,
`v`, `x`, or a digit), which itself is not interpreted in any
special way.

### 3.1.5.4. Character constants {#character-constants}

A *character constant* consists of a single character or escape sequence,
enclosed in single quotes.  All characters except newline can be used.
Escape sequences are handled as with *[string constants](#string-constants)*.

### 3.1.6. Operators {#operators}

The following are operators:

```
	[   ]   (   )   -> <-
	++  --  +   -   ~   !   ... catch new
	*   /   %   <<  >>  <   >   <=  >=  ==  !=  &   ^   |   &&  ||
	?   :
	=   *=  /=  %=  +=  -=  <<= >>= &=  ^=  |=
	,
```

### 3.1.7. Punctuators {#punctuators}

Punctuators are like a '.' in a sentence, there are a variety of punctuators
but they are only used in key places.  For example when your defining a 
variable, you might type: **int x, y;**. The ';' is a punctuator that
says this is the end of this statement.  The ',' says I have another int
that I want to define.  The following are some of the punctuators you will see: 

```
	, ; :
```

We'll introduce them as we need them, because they need context to make sense.

### 3.2. Expressions

### 3.3. Constant expressions

### 3.4. Declarations

A *Declaration* can be a
*[variable declaration](#variable-declaration)*,
*[array declaration](#array-declaration)*,
*[function declaration](#function-declaration)*.

### 3.4.1. Variable declaration {#variable-declaration}

A variable declaration has the following form:
*[TYPE](#types)* Identifier;  
If you have multiple variables of the same type, you can separate them by a
coma to keep them on the same line:  TYPE Identifier1, Identifier2;
**Note: DGD does not allow assignment in the declaration statement. (int x=5; is not valid DGD LPC)**

Examples:

```
	int x;
	mixed bing;
	float a, b;
```

### 3.4.2. Types {#types}

The following types are supported:

*[nil](#typenil)*,
*[int](#typeint)*,
*[float](#typefloat)*,
*[string](#typestring)*,
*[object](#typeobj)*,
*[mapping](#typemap)*,
*[mixed](#typemixed)*,
*[void](#typevoid)*

Types can also have *[Type Modifiers](#type-modifiers)*
which will change how the variable or function works.

### 3.4.2.1 Types: nil {#typenil}

Nil is a special value, its not actually a type but it is the value
of a uninialized variable.  So you can see if a variable is undefined by
seeing if it is equal to nil.  Example:

```
/* Note this does not work for type int, but does for other types: string, mapping, etc... */
string x;
if (x == nil) write("You need to set x"); 
```

### 3.4.2.2 Types: int {#typeint}

Declares the variable to be an integer, for a function it says the
function will return an integer. See 
*[integer constant](#integer-constants)* for valid integers.

### 3.4.2.3 Types: float {#typefloat}

Declares the variable to be a float, for a function it says the
function will return a float. See
*[floating constant](#floating-constants)* for valid floats.

### 3.4.2.4 Types: string {#typestring}

Declares the variable to be a string, for a function it says the
function will return a string. See
*[string constant](#string-constants)* for valid strings.

### 3.4.2.5 Types: object {#typeobj}

Declares the variable to be an object.  The driver deals with objects as a
basic type.  If you create a room, a monster, a player, or a sword, all of 
these are considered objects to the driver.  For a function it says the 
function will return an object.

### 3.4.2.6 Types: mapping {#typemap}

A mapping is similar to an *[array](#array-declaration)*,
execpt you get to name your indices.
It is also sometimes called an associative array or a hash.
Think of it as a key/value pair.  Example:

```
	mapping location;
	location['fred'] = 'home';
	write("Fred is " + location['fred'] + ".\n");
```

### 3.4.2.7 Types: mixed {#typemixed}

The mixed type is used when a function will take multiple types, you can
use the typeof function to find out what type the variable is currently
using.  A good example of using a mixed type would be if a function
returns a string or an int depending on what gets passed in.  You want to
avoid using the mixed type to just get around type checking, limit it to
cases where its truely useful to have multiple types.

### 3.4.2.8 Types: void {#typevoid}

The void type is only valid for functions.  It says that the function
does not return anything.  You can also use void to say that a function has
no *[paramaeters](#parameters)* passed into it. Examples:

```
void do_stuff(int x) { write("I did stuff " + x + " times.\n"); }
int do_stuff2(void) { write("I did stuff.\n"); }
```

### Type Modifiers {#type-modifiers}

Type modifiers appear before a type and provide special instructions
to the driver for how handle the special type.  A type can have multiple
modifiers associated with it.
There are the following type modifiers: 
*[private](#private)*,
*[static](#static)*,
*[nomask](#nomask)*,
*[atomic](#atomic)*

### 3.4.3. Type Modifiers: private {#private}

The private type modifier is valid for both functions and variables.
When a variable or fuction is marked as private,  it becomes only accessable 
by the object in which it is defined.  Not even child objects that inherit
this object will be able to access them.  You can use private functions
and variables to insure that others access your data the way
you want them to access it.

### 3.4.3. Type Modifers: static {#static}

The static type modifier behaves differently for variables and functions.

For variables that are declared as static, the variable will be defined
globally for that object, and it will not be saved when save_object is called.

For functions that are declared as static, the function will only be available
to the object and it's children.

### 3.4.3. Type Modifers: nomask {#nomask}

The nomask type modifier is only valid for functions, it
makes it so that you can not redefine a function.  To put it another way,
a child object can not have a function with the same name, it will use 
the parents version of the function.

### 3.4.3. Type Modifers: atomic {#atomic}

The atomic type modifier is only valid for functions, it
makes it so that if an error occurs durring the function, the
driver will roll everything back to before the function was called.
It's a saftey net of sorts.  If its modifying an object and half way
through it has a problem, the object will get reset to its state
before the function call.  Atomic functions have a couple of drawbacks.
They are forbidden from working with files in any way(renaming, writing, 
reading, etc...).  They also have some overhead, they take twice as many
ticks as they would if they were not declared atomic.

### 3.4.3. Array declaration {#array-declaration}

An array is an extension of a basic type in lpc, which turns it into an 
ordered list.  You can denote an array in the following way:
basic_type \*Identifier;

### 3.4.4. Function declaration {#function-declaration}

A function declaration can be a 
*[function prototype](#function-prototype)*, or a
*[function definition](#function-definition)*.

### 3.4.4.1. Function prototype {#function-prototype}

A prototype is used to reserve a name for a function, and specify the
number of parameters you need to give it as well as define what the function
will return, without defining the actual function.  It allows the driver
to put the function in the symbol table even though it does not know what the 
function is yet.  It is basically saying \"hey I'm going to make a function
that looks like this later\".

A function prototype has the form: type Identifier(paramaters);

### 3.4.4.2. Function definition {#function-definition}

A function definition has the form: type Identifier(paramaters) { statements }

### 3.5. Statements

There are lots of different types of statements: Assignments, function calls,
function definitions, conditionals, loops, and other statements that affect
the flow of a program.

### Conditionals

### Loops

### 3.6. Inheritance

Inheritance is one of the key building blocks of lpc.  The mudlib defines
some standard objects.  For example, maybe your mudlib has: 
/std/object.c, /std/room.c or /std/monster.c.
You can use these building blocks by inheriting them.  Inherit statements
must come before any other code, including #include preprocessor statements.

The inherit syntax can have the following syntax:

```
[priviate] inherit "FILENAME";
[priviate] inierit Identifier "FILENAME";
```

You can use an Identifer if your inheriting from multiple objects and want to
explicitly call functions from one of them.  The private option will make it
so that only this object will have access to the variables and functions
inherited by the inherit statement.  You will not be able to access them
from a call_other function, or if you inherit this object, you will not
have access to the privately inherited stuff.

Examples:

```
	inherit "/std/room.c";
	void create( void ) {
		::create();
		set_short("The void");
		set_long("A vast nothingness.");
	}
```

or you can also do something like this:

```
	inherit room "/std/room.c";
	inherit ob "/std/object.c";
	void create( void ) {
		room::create();
		ob::create();
		set_short("The void");
		set_long("A vast nothingness.");
	}

```

Because we have inherited our standard room, we get all of the code
in our room file for free.  Because our room code has a function for
set_long, we do not need to define set_long, we can just call it.
This makes programming in lpc much more readable and maintainable.
If we want to we can override our set_long function by defining a new
function with the same name.  In the above example, we have wrote
over the create function.  With a new function that sets a short and
long description for our room.  the :: operator is used to call a function
that is defined at a higher level.  So ::create(); says first call
the create function that was defined before this create function.
So its important to call ::create(); so that we do not have to
duplicate all of the code in /std/room.c's create function.
We can just call ::create(); and then after that make any additional
modifications we want to make.  If we don't care about the previously
defined create function we can remove the ::create(); call.

### 3.7. Preprocessing directives

Preprocessor commands start with a #, these commands, get run before compile
time.  They are used to optionally compile bits of code and or to instruct
the compiler how to do something.  Many times they are used to make things
more portable.  The following preprocessor directives are recognized: 
*[include](#preinclude)*,
*[define](#predefine)*,
*[undef](#preundef)*,
*[ifdef](#preifdef)*,
*[ifndef](#preifndef)*,
*[if](#preif)*,
*[endif](#preifdef)*,
*[else](#preelse)*,
*[elif](#preelif)*,
*[pragma](#prepragma)*,
*[line](#preline)*,
*[error](#preerror)*

### 3.7.1. #include {#preinclude}

You use #include to include a header in a file.  A header file in general
usually has special definitions and function prototypes.  For example if you
were writing a NETWORK header, you might have a definition for a 
default network port, and function prototypes for connect, disconnect etc..
Header files are kind of like an API for the software you write.
They make it simple to change important settings and explain how you use
a library.  In general they should not include code in the header file.
The syntax for an include looks like this:

```
#include <file.h>
#include "filename.h"
```

In general, the first case is for a standard header in the default search path
and ""'s are used to include your own header.  It starts by searching the
current directory.
Examples:

```
#include <std.h>
#include "room.h"
```

### 3.7.2. #define {#predefine}

You use #define lots of times in a header file (filename.h)
to make it easy to change certain values.  The syntax for
a define looks like this: 

```
#define Identifier VALUE
#define Identifier
```

If VALUE is omitted, it defines the Identifier but your not sure
what the value is.  It may be nil, or it may be 1, or it may be something 
else depending on implementation of the driver.

Examples:

```
#define MAX_STR 255
#define DICTIONARY_SIZE 2000
#define DEFAULT_PORT 4000
#define DEBUGGING_ON
```

### 3.7.3. #undef {#preundef}

You use #undef to undefine a previously defined item.  The syntax
for undef is:

```
#undef Identifier
```

Examples:

```
#undef MAX_STR
#undef DEFAULT_PORT
```

### 3.7.4. #ifdef, #endif {#preifdef}

Ifdef is used to instruct the driver to optionally evaluate some code.
While we are explaining ifdef we also need to introduce #endif since it
is needed to complete an ifdef statement.
\#ifdef has the following syntax:

```
#ifdef Identifier
(insert your code here)
#endif
```

So if whatever Identifier you have specified is defined to some value 
(with an #define, or it can also be passed to the driver at runtime)
Then it will evaluate the code until it hits a #endif  If there is no
\#endif the driver will throw an error.  Note: #endif is used in multiple
places to designate then end of a block of code, its sometimes hard to keep
track of it all so its good to add a comment saying this #endif goes with
the Identifier you specify.  Note, ifdef does not actually check the
value of your Identifier, it just checks to see if it is defined.
Something can be defined as 0 for example and ifdef will still say
yes its defined.  If you want to check the value of something that is
defined, you will want to use a more complicated #if statement, which
you can find more info on later in the document.

Example:

```
#ifdef DEBUG
	write("Were in debugging mode.\n");
#endif // DEBUG
```

### 3.7.5. #ifndef {#preifndef}

Ifndef works the same way as ifdef but it only gets called if the Identifier
you specify is not defined.

Example:

```
#ifndef DEBUG
	#define DEBUG 1
#endif
```

### 3.7.6. #if, #else, #elif {#preif}

An if statement is used for more complicated tests, you can test
the value of a defined item, or for variations etc...  It works
similar to the if conditional statement but at the preprocessor level.

Examples:

```
#if defined(DEBUG) || defined(VERBOSE)
	write("Welcome to VERBOSE MODE!\n");
#endif

#if (3 == DEBUG)
	write("Were in really verbose MODE.\n");
#elif (2 == DEBUG)
	write("Were in verbose MODE.\n");
#elif (1 == DEBUG)
	write("were in semi verbose MODE.\n");
#else
	write("Were in quiet MODE.\n");
#endif
```

### 3.7.7. #pragma {#prepragma}

You use #pragma to pass a flag to the driver.  In general
This is used for platform specific code. DGD currently ignores
\#pragma statements.  The syntax for Pragma is:

```
#pragma STRING
```

Note: the STRING is determined by the DRIVER its not something you can
just make up.  If the DRIVER does not recognize a STRING it will just ignore
it and or maybe send a warning.

### 3.7.8. #line {#preline}

Used to override the driver's automatic detection of line# and or filename.
I think this is suppose
to be used to get around bugs, you probably want to avoid this unless
you find a situation where you really need it.  If your familiar with
FLEX, BISON and or YACC, you know that sometimes your code is generated
on the fly and you could use #line to tell people the real file and
position in the file they are interested in looking at for debugging purposes, 
instead of the generated code which only exists for a short time.
(If you don't know what I'm explaining, you probably do not need to 
worry about it.)

Examples:

```
#line 22 "myfile.c"
#line "myfile.c"
#line 22
```

### 3.7.9. #error {#preerror}

Call an error message, This is usually based on some condition that is
external to the #error command,

Example:

```
#ifdef APSTUDIO_INVOKED
        #error this file is not editable by Microsoft Visual C++
#endif
```
