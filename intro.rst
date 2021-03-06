Welcome to Python
=================

About This Tutorial
-------------------

This tutorial is geared at teaching modern Python (and useful libraries) to people that are already somewhat familiar with programming in another language.

Aiming for a pretty wide range of skill levels in a tutorial like this is challenging, but this tutorial adopts a few strategies to help:

* One of the most important skills in being a developer is learning through reading code & documentation.  This tutorial will often guide you to external resources to dive deeper.
* While there will be code examples and some exercises, make sure you occassionally start up Python and type in code and reason why something happened.  That, combined with understanding how to find and use documentation is what will make you a strong developer.
* Often shorter books & tutorials skimp on core concepts in favor of demonstrating syntax, etc.  I'm going to take the opposite approach and try to explain some fo the core concepts being addressed.  This is hopefully useful for beginners to learn from and for more experienced developers to relate what Python is doing to concepts they're already familiar with.


What Is Python?
---------------

If you read Wikipedia or a typical Python introduction you'll see it described as an **intepreted** & **dynamically typed** language.

You'll also sometimes hear languages defined in terms of being **object-oriented**, **functional**, or **procedural**. Python, like many of its modern peers, is somewhat agnostic on this front.  Python enables development using all three of these paradigms and a lot of Python code is a blend of styles.

Python is also written with a focus on **readability**.  As we'll see, this is partially a technical consideration but largely a cultural one.  That difference doesn't make it any less important in defining the character of the language.

These things together make Python what it is, a language that is good for beginners but with enough extensibility that it is possible to use for a wide variety of applications.  in the real world Python is used for everything from simple web applications to intense scientific applications.

Before we get into what all of these adjectives mean, let's take a second to understand how Python came to be.

A Brief History
---------------

* **December 1989** - Guido van Rossum starts Python as a hobby project.  Loosely based on ABC, a teaching/prototyping language, but designed to also appeal to Unix/C developers.
* **January 1994 - Python 1.0**: first major public release.  Guido was working on the Computer Programming 4 Everyone (CP4E) initiative at Corporation for National Research Initiatives in Reston, VA.
* **October 2000 - Python 2.0**: switch truly community-driven and the language started down its functional path.  Many of the functional elements of Python were introduced around this time.
* **December 2008 - Python 3.0**: the first release containing major breaking changes since Python 2.0.  A major update that notably changed Python's handling of unicode.  Also fixed many inconsistencies/warts in the language.  Because of the amount of Python 2 code out there and the backwards-incompatible nature of these changes the transition to Python 3 is still in progress throughout the community.  (It started in earnest around the time of Python 3.3 in 2012.)

For our purposes we'll use Python 3.5.  Though it took longer than anyone hoped, Python 2 is on its way out and Python 2.7 will not be supported past 2020 and most major libraries now support Python 3 and many will soon require it.  It is good however to understand the distinction as you may from time to time find something that requires Python 2.


How Python Runs
---------------

As we said before- Python is an **interpreted** language, much like JavaScript or Ruby.  It is somewhat unique in that it is typically bytecode interpreted, which is why you'll see .pyc files as you work in your ``__pycache__`` directory.  This step allows code to execute somewhat faster but without the necessary compilation step present in languages like C or Go.

This means that instead of the Python interpreter interpreting your code line by line it first precompiles it to **bytecode** or a series of instructions analogous to assembler code.  This allows the code to run somewhat faster without the overhead of parsing the Python syntax at runtime.

To really understand this let's compare it to some other languages:

C
~~

Steps to run code in a traditional compiled language like C:

* hello.c is compiled, creating an intermediate architecture-specific assembler version (hello.s)
* hello.s is assembled, creating an intermediate 'object' file with the platform's object code (hello.o)
* hello.o is linked with other object files, creating the executable (hello)
* the user runs ./hello - a fully contained executable for their OS/architecture

::

    .----------------.
    |    hello.c     |
     ----------------
            V
    /===============\
    |   compiler    |
    \===============/
            V
    .----------------.
    |  hello-x86.s   |
     ----------------
            V
    /===============\
    |   assembler   |
    \===============/
            V
    .--------------------.
    |  hello-x86-osx..o  |
     -------------------- 
            V
    /===============\
    |    linker     |
    \===============/
            V
    .-------------.
    |    hello    |  (executable)
     -------------

    
JavaScript
~~~~~~~~~~

Steps to run JS code (on V8):

* JavaScript is run directly within the V8 interpreter without any intermediate step
* while the JS is running the intepreter also JIT (just-in-time) compiles portions of the code directly to architecture-specific machine code

::

    .----------------.
    |    hello.js    |
     ----------------
            V  
    /=================================\
    |   V8 interpreter & compiler     |
    \=================================/
           |             ^
           |             |
           \-------------/
              JIT compilation

CPython
~~~~~~~

::


    .----------------.
    |    hello.py    |
     ----------------
            V  
    /====================\
    | Python (compiler)  |
    \====================/
            V
    .----------------.
    |   hello.py c   |
     ----------------
            V  
    /=======================\
    | Python (interpreter)  |
    \=======================/


.. note::

    Wait, what is CPython?

    You'll sometimes see people refer to CPython.  This isn't a different language, but is likely the version of Python interpreter you're using.

    CPython refers to the implementation of Python that is written largely in C, derived from that initial version released years ago by Guido.

    The reason this distinction is made when we talk about the interpreter is do differentiate the language from different interpreters that the community has
    created.  The most popular of these is PyPy, an implementaiton of Python largely written in Python (mindblowing? yeah).  PyPy uses different techniques
    such as JIT compilation to gain faster speeds than CPython.


Typing
~~~~~~

Python is a **strongly** & **dynamically** typed language.  This varies from other languages:

If you're familiar with another interpreted language like JavaScript or Ruby those too are dynamically typed.  A language where the following code is valid is generally said to be dynamically typed::

    x = "hello"
    x = 42

Notice we do not declare a type for x, and later when we assign a number to x it doesn't break anything.  This is in contrast to statically typed languages in which types must explicitly be defined (C) or can be inferred but then not changed (Go).

As for strongly typed, this means that the language is aware of the type of a variable and does not coerce it to other things implicitly.  This can be contrasted with a language like JavaScript:

.. code-block:: javascript

    x = '3'
    x = x * 4
    // x is now 12

Languages like JavaScript are said to be weakly typed (or sometimes untyped).

While strong, weak, dynamic, and static typing all have their advantages and disadvantages, it is good to understand that in Python's case types are strong and dynamic. 

While not strictly part of the type system, Python APIs also tend to adhere to the principle of duck typing.  That is "if it walks like a duck, and it talks like a duck, then it's a duck."

What this means in practical terms is that instead of checking if an object is ``SomeType`` we often just check that it adheres to a *protocol*, that is to say- it has the methods we would expect.

For example::

    def print_sorted(thing):
        thing.sort()
        print(thing)
        
    >>> print_sorted([3,4,5,1,2])
    [1,2,3,4,5]

This function could be called with anything that has a ``.sort()`` method and is printable.

Contrast this to how this might be written in C:

.. code-block:: c

    void print_sorted(int thing[]) {
        sort_integer_array(thing)
        print_integer_array(thing)
    }

While sometimes you care about the exact type, typically you'll let these duck-typing situations iron themselves out, our first attempt at print_sorted() is going to be a lot more flexible than::

    def print_sorted(thing):
        if isinstance(thing, list):
            thing.sort()
            print(thing)
        else:
            raise ValueError('can only sort and print lists!')

Which is longer and generally not considered idiomatic Python, or to use an idiom, Pythonic.


Pythonic?
---------

Like any language, Python developers have a lot of opinions about how to write "good" Python.  Keeping these guidelines in mind will help keep your Python code **readable** and maintainable.

One thing that is fairly unique is that the opinion is a bit more consistent in Python than in some languages.

These constraints are made more necessary in part because of the flexible nature of Python and they are made possible in part due to the fact that Python is goverened by a single individual (Guido van Rossum, the Benevolent Dictator For Life or BDFL) and those he delegates to.

In writing good Python you'll hear references to PEP 8 and sometimes to the 'Zen of Python'.

`PEP 8 <https://www.python.org/dev/peps/pep-0008/>`_ started life a 'Python Enhancement Proposal'- the means by which language features are proposed and then ultimately decided upon by Guido.  It is a formal style guide for the language and should be followed more often than not.  (Though it is worth noting the first section is titled "A Foolish Consistency is the Hobgoblin of Little Minds" -- you'll find pragmatism is a virtue in the Python community.)

Highlights of PEP 8 are:

* Use 4 spaces for indentation, do not use tabs.
* Top level class and function declarations should be preceeded by two blank lines.
* All other declarations should be preceeded by one blank line.
* Operators should be separated from operands by a single space ``a + b``  not ``a+b``
* Extra whitespace within parens/braces should be minimized:
    * ``spam(ham[1], {eggs: 2})`` - GOOD
    * ``spam( ham[ 1 ], { eggs: 2 } )`` - BAD
* Naming:
    * ``ClassNamesLookLikeThis``
    * ``function_names_and_variable_names_like_this``
    * ``CONSTANTS_LIKE_THIS``
    * ``_private_members_preceeded_with_underscore``

There's plenty more, and as you become an experienced Python developer you'll see a lot of code and learn from it.  There are also popular source code linters that'll let you know if your code is foolishly consistent.  They can be helpful especially in maintaining consistency across large codebases.

The other commonly referenced meaning of "Pythonic" is less practical and more philisophical.  It is embodied in the following "poem" which you can see by typing ``import this``::

    >>> import this

    The Zen of Python, by Tim Peters

    Beautiful is better than ugly.
    Explicit is better than implicit.
    Simple is better than complex.
    Complex is better than complicated.
    Flat is better than nested.
    Sparse is better than dense.
    Readability counts.
    Special cases aren't special enough to break the rules.
    Although practicality beats purity.
    Errors should never pass silently.
    Unless explicitly silenced.
    In the face of ambiguity, refuse the temptation to guess.
    There should be one-- and preferably only one --obvious way to do it.
    Although that way may not be obvious at first unless you're Dutch.
    Now is better than never.
    Although never is often better than *right* now.
    If the implementation is hard to explain, it's a bad idea.
    If the implementation is easy to explain, it may be a good idea.
    Namespaces are one honking great idea -- let's do more of those!

Congratulations!
----------------

So we've covered what it means that Python is **interpreted** and **dynamically typed**.

We've also talked about the emphasis on **readability** and the importance Python developers place on making things "pythonic."

Now you're ready to actually start learning Python!

We'll start with :doc:`scalar-types`.
