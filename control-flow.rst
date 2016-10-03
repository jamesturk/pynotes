Control Flow
============

In this section we'll be covering Python's various control flow mechanisms.

This includes:
    * ``if``
    * ``for``
    * the concept of **iterables**
    * ``while``, ``continue``, ``break``
    * exception handling


``if`` Statement
----------------

Python's ``if`` statement is relatively straightforward.  A simple example::

    if x < 0:
        print('negative')
    elif x == 0:
        print('zero')
    else:
        print('positive')

``for`` Statement
-----------------

Python's ``for`` statement is also quite simple.  It takes the form:

``for`` *variable* ``in`` *iterable*:
    *loop body*

Unlike many other languages there aren't other forms, that's it.

As you may expect, *variable* is the value that changes on each iteration of the loop.

So what is *iterable*?

iterables
---------

The simplest iterable is any of the types we covered in the last section: ``list``, ``tuple``, ``dict``, ``set``, ``frozenset``.

Let's look at some examples.  Lists and tuples are straightforward, you'll iterate over them one element at a time::

    >>> for x in [1, 2, 3]:
    ...     print(x)
    1
    2
    3

    >>> for x in ('a', 'b', 'c'):
    ...     print(x)
    a
    b
    c

What about unordered collections like ``set``?

::

    >>> for x in {1, 2, 3}
    ...     print(x)
    2
    3
    1

If you try this locally you might get a different order.  Since ``set`` isn't ordered there's no guarantee on order.


range and enumerate
'''''''''''''''''''

Sometimes it is useful to iterate over a range of numbers the way you would in other programming languages.  Fortunately there's a builtin ``range()`` function for just this purpose::

    >>> for x in range(3):
    ...     print(x)
    0
    1
    2

The ``range`` operator takes up to three arguments.  ``start``, ``stop``, ``step``.

You may recall that this is the way that **slices** worked as well.

A few examples::

    >>> for x in range(2, 5):
    ...     print(x)
    2
    3
    4

    >>> for x in range(0, 50, 9):
    ...     print(x)
    0
    9
    18
    27
    36
    45

Sometimes it is also useful to know which index in a list you're at while you iterate.  You may think to use ``range`` this way::

    >>> my_list = ['first', 'second', 'third']
    >>> for x in range(len(my_list)):
    ...     print(x, my_list[x])
    0 first
    1 second
    2 third

But Python provides a cleaner way using ``enumerate``::

    >>> my_list = ['first', 'second', 'third']
    >>> for i, x in enumerate(my_list):
    ...     print(i, x)
    0 first
    1 second
    2 third

You'll notice that it looks like we're specifying two variables.  In reality ``enumerate`` returns a ``tuple``, and we're using **tuple unpacking** to automatically turn that tuple into two variables.


``dict`` Iteration
''''''''''''''''''

Similarly, ``dict`` is unordered.  Iterating over a ``dict`` returns the dictionary's keys::

    >>> for x in {'dog': 'woof', 'cat': 'meow', 'duck': 'quack'}:
    ...     print(x)
    dog
    cat
    duck

It's not uncommon to want to iterate over more than just the keys.  ``dict`` provides three methods to be explicit in what you prefer to iterate over::

    >>> d = {'dog': 'woof', 'cat': 'meow', 'duck': 'quack'} 
    >>> for x in d.keys():
    ...     print(x)
    duck
    dog
    cat

    >>> for x in d.values():
    ...     print(x)
    meow
    quack
    woof

    >>> for x in d.items():
    ...     print(x)
    ('cat', 'meow')
    ('duck', 'quack')
    ('dog', 'woof')

Since ``items`` returns tuples we can use **tuple unpacking** again::

    >>> for k, v in d.items():
    ...     print(k, v)
    cat meow
    duck quack
    dog woof

We'll cover more iterables later when we get to **generators**, but for now let's move on to look at other control flow elements.


``while`` Statement
--------------------

The ``while`` statement is also pretty much what you'd expect if you've used one in another language::

    while condition:
        do_something()

``break`` and ``continue``
--------------------------

Within both ``for`` and ``while`` loops you can use ``break`` and ``continue`` statements.

Example::

    >>> for x in range(10):
    ...     if x == 3:
    ...         continue
    ...     print(x)
    ...     if x == 5:
    ...         break
    0
    1
    2
    4
    5

Exception Handling
------------------

There's one other form of flow control that is common within Python, exception handling.

You may be familiar with the concept from languages like C++, Java, or JavaScript.

One thing that differs compared to many other languages is that in Python exceptions are relatively lightweight.  This means they aren't only meant to be used in the most extreme circumstances, instead it is not uncommon to use them as a type of control flow.

First let's look at how Exception handling works:

try & except
''''''''''''

Let's try to access an element that doesn't exist within a list::

    >>> my_list = [1, 2, 3]
    >>> try:
    ...     my_list[99]
    ... except Exception as e:
    ...     print(e)
    list index out of range

the ``except`` clause takes the form:

``except`` *ExceptionTypes* ``[as *variable*]``

Where *ExceptionTypes* is one or more ``Exception`` names and the optional
``as variable`` portion allows storing the exception for further processing or display.

You can also have multiple ``except`` clauses that handle different types differently::

    >>> my_list = [1, 2, 3]
    >>> try:
    ...     my_list[99]
    ... except IndexError:
    ...     print('index error')
    ... except ValueError:
    ...     print('value error')
    index error

Common Exceptions
'''''''''''''''''

``BaseException``
    The base exception, catching this will catch **all** exceptions.
``Exception``
    The lowest-level non-system exiting exception.  Typically this is the lowest level exception you'd want to catch.
``AttributeError``
    Raised when attempting to access an attribute of an object that doesn't exist.  (e.g. ``x.this_probably_isnt_a_function()``)
``ImportError``
    Raised when something cannot be ``import``ed.
``IndexError``
    Raised when a sequence index is out of range.
``KeyError``
    Raised when trying to access a dictionary key that does not exist.
``StopIteration``
    Raised when an iterable is exhausted.
``TypeError``
    Raised when an operation is invalid for a specific type.
``ValueError``
    Raised when a function receives a value of an appropriate type but inappropriate value.
``ZeroDivisionError``
    Raised when attempting to divide by zero.

Check out the `full list of built-in exceptions <https://docs.python.org/3/library/exceptions.html>`_.

You'll also define your own Exceptions once we get to **classes**.

else & finally
''''''''''''''

It is also possible to have two more clauses in a try-except block.

``else`` is called if no exception was caught.  ``finally`` is caused no matter what, and is useful for cleaning up resources.  Here's an example demonstrating::

    try:
        x / y
    except ZeroDivisionError:
        print('y was zero')
    else:
        print('y was not zero')
    finally:
        print('always prints')


For more on Python exceptions, see `the error handling section of the tutorial <https://docs.python.org/3/tutorial/errors.html>`_ or the `full exception list <https://docs.python.org/3/library/exceptions.html>`_

Now that we know basic control flow constructs we'll look at how Python handles :doc:`functions`.
