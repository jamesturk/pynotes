Generators and Comprehensions
=============================

Now that we've written functions we're going to look at another type of iterable.  Sometimes we want to create an iterable that doesn't require us storing all of the data in memory.  An example would be when we're trying to iterate over a large file or an iterable that is potentially infinite.

So far we'd iterated over sequences and dictionaries.  Creating a ``list`` with every line in a 1TB file wouldn't be desirable or necessary if we were processing it one line at a time.

In fact we've already seen this, let's take another look at ``range``.


Revisiting Iterables
--------------------

If you recall, we can use ``range`` to iterate over arbitrary sequences::

    >>> for x in range(5):
    ...     print(x)
    0
    1
    2
    3
    4

But what is ``range``?  It looks like a function, maybe it returns a list?

::

    >>> r = range(5)
    >>> print(r)
    range(0, 5)
    >>> type(r)
    range

What?

It turns out ``range`` is a **generator**.  A generator is a function that can return multiple values.  Let's look at how to write one.

Generator Functions
-------------------

In a typical function, the first ``return`` statement we reach terminates the function::

    >>> def abc():
    ...     return 1
    ...     return 2
    ...     return 3
    >>> abc()
    1
    >>> abc()
    1

A **generator function** uses the ``yield`` statement instead::

    >>> def abc():
    ...     yield 1
    ...     yield 2
    ...     yield 3
    >>> abc()
    <generator object abc at 0x10edb6fc0>
    >>> abc()
    <generator object abc at 0x10edb7110>


Each time the **generator function** is called it returns a **generator**.

You'll notice each of these generators is at a different memory address, each one is maintaining its own state.

Generators are iterable, just like collections and ``range``.

So we can iterate over the result of a generator function like this::

    >>> for x in abc():
    ...     print(x)
    1
    2
    3

We can also store a generator and pass it around.  Let's try passing one to the ``list`` function::

    >>> gen = abc()
    >>> list(gen)
    [1, 2, 3]

That's a handy way to convert a generator to a sequence type.

What happens if we try to iterate over a generator twice?

::

    >>> gen = abc()
    >>> list(gen)
    [1, 2, 3]
    >>> for x in gen:
    ...     print(x)

    # nothing prints

Our generator in this case is **exhausted**, it won't yield any more values.

It is possible to make infinite generators::

    def forever(n):
        while True:
            yield n

You'd need to make sure you used ``break`` or raised an exception while iterating over the result of ``forever(5)`` or you'd hang the program.

There's also a way to just get the next value from the generator, the builtin function ``next``::

    >>> gen = abc()
    >>> gen2 = forever(2)
    >>> for x in gen:
    ...     print(x, next(gen2))
    1 2
    2 2
    3 2

If you try to call ``next`` on an exhausted generator, it'll raise ``StopIteration``::

    >>> gen = abc()
    >>> next(gen)
    1
    >>> next(gen)
    2
    >>> next(gen)
    3
    >>> next(gen)
    StopIteration


Generators are extremely powerful, the `Python docs for generators <https://docs.python.org/3/tutorial/classes.html#generators>`_ explain in more detail.


Comprehensions
--------------

We don't need to define a function to create a generator, we can also use a **generator expression**.

A generator expression is a statement in the format:

(*expr* ``for`` *var* ``in`` *iterable*)

This looks kind of like an inside-out for loop.  Let's look at an example:

    >>> gen = (n*2 for n in [1,2,3])
    >>> for x in gen:
    ...     print(x)
    2
    4
    6

Keep in mind here that [1,2,3] could be anything iterable, including another generator.

We could use this to get the sum of all squares under 100::

    >>> sum(i*i for i in range(100))
    328350

It is also possible to add a conditional:

(*expr* ``for`` *var* ``in`` *iterable* ``if`` *truth-expr*)

Let's get the squares of odd numbers::

    >>> gen_exp = ((i, i**2) for i in range(10) if i % 2)
    >>> for n, n2 in gen_exp:
    ...     print(n, 'squared is', n2)
    1 squared is 1
    3 squared is 9
    5 squared is 25
    7 squared is 49
    9 squared is 81

Notice the addition of the trailing ``if`` statement, as well as the fact we're generating a tuple here.

(Also recall that we'd probably do this by passing the step argument to range in real code, which would avoid the evaluation step here.)

map, filter, reduce
-------------------

You have have seen functions with these names in other contexts.  These are available in Python but often not used since generator expressions provide the same behavior.

Let's look at some examples::

    >>> for i in map(lambda x: x**2, range(5)):
    ...     print(i)
    0
    1
    4
    9
    16

The first argument of map is a function (often a ``lambda``) and the second is an iterable.  The result is an iterable where each value is processed through the function.

This is equivalent to the simple generator expression::

    >>> for i in (x**2 for x in range(5)):
    ...     print(i)
    0
    1
    4
    9
    16

``filter`` allows for removing values from the iterable based on a test.  For example::

    >>> for i in filter(lambda x: x % 2, range(10)):
    ...     print(i)
    1
    3
    5
    7
    9

This too is equivalent to a generator expression::

    >>> for i in (x for x in range(10) if x % 2):
    ...     print(i)
    1
    3
    5
    7
    9

The above generator expression ``((i, i**2) for i in range(10) if i % 2)`` can be recreated with map & filter::

    >>> gen_exp = map(lambda i: (i, i**2), filter(lambda i: i % 2, range(10)))
    >>> for n, n2 in gen_exp:
    ...     print(n, 'squared is', n2)
    1 squared is 1
    3 squared is 9
    5 squared is 25
    7 squared is 49
    9 squared is 81

But you'll probably agree that the generator expression looks nicer.


The other function often mentioned with ``map`` and ``filter`` is ``reduce``.

``reduce`` is a bit different, it takes a function that takes two items and combines them to one, it is called continuously on the iterable's items until a single value is produced.  A simple example would be a sum function::

    >>> from functools import reduce
    >>> reduce(lambda a, b: a+b, [1,2,3,4])
    10

We'll talk more about ``import`` and appropriate uses of ``reduce`` later.


Comprehensions
---------------

Earlier we saw an example of using a generator to construct a list.  This is a useful thing to be able to do, and there's a more direct way to get this functionality without making a generator as an intermediary.

Similar to the generator expression, we can use a **list comprehension**.  The only difference is that these list comprehensions are surrounded by ``[]`` instead of parentheses.

Let's look at some list comprehensions::

    >>> [n for n in range(10)]
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    >>> [n*2 for n in range(10) if n*2 < 10]
    [0, 2, 4, 6, 8]

In addition to list comprehensions, there are also **set comprehensions** and **dict comprehensions**.  These look similar, but just like ``set`` and ``dict`` differ by whether or not there are (key, value) pairs or not.

Some set comprehensions::

    >>> {n*2 for n in range(10) if n*2 < 10}
    {0, 2, 4, 6, 8}
    >>> {n for n in [0, 0, 1, 1, 0]}
    {0, 1}

And dict comprehensions::

    >>> {i: i**2 for i in range(10) if i % 2}
    {1: 1, 3: 9, 5: 25, 7: 49, 9: 81}
    >>> {word: len(word) for word in ['dog', 'frog', 'python']}
    {'dog': 3, 'frog': 4, 'python': 6}


Should you want a tuple comprehension you can wrap a generator expression in ``tuple()`` or ``frozenset()``.

Coming from languages without comprehensions these can look quite foreign, but embracing them can lead to shorter but also more efficient and readable code.  Some books treat comprehensions as an "advanced" concept but hopefully these examples convince you they can be just as clear as a simple for loop.

Take a breath, then let's dive into :doc:`classes`.
