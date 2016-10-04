Functions
=========

Functions in Python are a lot like functions in other languages with a few major things to note:

* Functions are first-class objects, this means that they can be passed as arguments and treated like any other type.
* All functions return a value, if no ``return`` statement is specified then ``None`` will be returned.
* It is common to use tuples to in effect return multiple values.
* Functions can take **positional** and **keyword** arguments.
* It is possible to define functions that take variable numbers of arguments, and define defaults for arguments.

Defining Functions
------------------

Let's take a look at a very simple Python function::

    >>> def func(a, b=2):
    ...     return a + b

This function returns the sum of two arguments.  The second argument defaults to ``2``.  Let's call it a few ways::

    >>> func(1, 1)
    2
    >>> func(5)
    7
    >>> func()
    TypeError: func() missing 1 required positional argument: 'a'

As you can see, ``b`` is optional, but ``a`` is not.

We've been passing these arguments by position, the first argument is ``a`` and the second (if supplied) is ``b``.  We can also pass these arguments by keyword::

    >>> func(a=3, b=4)
    7
    >>> func(b=4, a=3)
    7

As you can see, the order doesn't matter when we pass arguments by keyword.  It is common to pass arguments by keyword when there are many arguments.  This makes code both easier to read and less error-prone.

If a function doesn't return a value explicitly, it will return ``None``::

    >>> def say_hello():
    ...     print('hello')
    >>> x = say_hello()
    hello
    >>> print(x)
    None

    >>> def pos_or_neg(num):
    ...     if num > 0:
    ...         return 'pos'
    ...     elif num < 0:
    ...         return 'neg'
    >>> print(pos_or_neg(0))
    None

Warning: Mutable Default Arguments
''''''''''''''''''''''''''''''''''

Let's say you have a function that has a default argument that would be a mutable object such as a ``dict`` or ``list``::

    >>> def make_list(item, times, initial=[]):
    ...     new_list = initial
    ...     for x in range(times):
    ...         new_list.append(item)
    ...     return new_list
    >>> make_list(0, 10)
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    >>> make_list(2, 4)
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 2, 2, 2, 2]

That's probably not what we wanted to happen.

At issue here is that the declaration of the function happens once, whereas the
body happens every time it is called.  This means ``initial`` is set to an empty list once, but persists between calls.

What you'd want to do in this case would be something like::

    >>> def make_list(item, times, initial=None):
    ...     if initial is None:
    ...         initial = []        # this will be a fresh list each call
    ...     new_list = initial
    ...     for x in range(times):
    ...         new_list.append(item)
    ...     return new_list
    >>> make_list(0, 10)
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    >>> make_list(2, 4)
    [2, 2, 2, 2]

Argument Lists
--------------

We've already seen how to define normal arguments and arguments with optional default values.  Let's look at how we can define functions that take multiple **positional** arguments::

    >>> def add(*numbers):
    ...     total = 0
    ...     for num in numbers:
    ...         total += num
    ...     return total

``*numbers`` will take any number of positional args and pack them into a single tuple.  Let's try calling our function::

    >>> add(1,2)
    3
    >>> add(1,2,3)
    6
    >>> add(5)
    5
    >>> add()
    0

We can also define functions that take multiple **keyword** values::

    >>> def mash(**monsters):
    ...     for name, num in monsters.items():
    ...         for x in range(num):
    ...             print(name)

``**monsters`` will pack all keyword arguments into a dictionary.  For example::

    >>> mash(werewolf=4, mummy=3)
    mummy
    mummy
    mummy
    werewolf
    werewolf
    werewolf
    werewolf

(Note that because this is a dictionary, order isn't preserved.)


It is possible to combine all the types of arguments we've seen into a single function, but positional arguments must always come before keyword arguments.
It's common to see functions like::

    def configure_server(hostname, port=80, **kwargs):
        ...

Or even::

    def send_message(message, *args, **kwargs):
        ...

There's nothing special about the names ``args`` and ``kwargs`` but they're often used in this context.


Passing Functions
-----------------

As mentioned before, functions are first-class objects.  They can be passed around just like any other type in Python.

For example::

    >>> def add(a, b):
    ...     return a + b

    >>> def mult(a, b):
    ...     return a * b

    >>> def call_func(f):
    ...     print('about to call function', f, 'with 3 and 4')
    ...     return f(3, 4)

    >>> call_func(add)
    about to call function <function add at 0x10edc9ea0> with 3 and 4'
    7
    >>> call_func(add)
    about to call function <function mult at 0x10ed27a60> with 3 and 4'
    12


Lambdas
-------

Sometimes it is nice to be able to declare a function inline, typically as you pass it to another function.

A common example of this is the builtin ``sorted`` function that takes an optional ``key`` argument to determine the value to sort on.

Let's look at a use case::

    >>> animals = ['cat', 'Dog', 'Fish']
    >>> sorted(animals)
    ['Dog', 'Fish', 'cat']

    >>> def lowercase(s):
    ...     return s.lower()

    >>> sorted(animals, key=lowercase)
    ['cat', 'Dog', 'Fish']

But it'd be nice not to have to define this throwaway ``lowercase`` function, so we'll use a lambda::

    >>> sorted(animals, key=lambda s: s.lowercase())
    ['cat', 'Dog', 'Fish']

Lambdas in Python are intentionally constrained.  They can take any number of arguments, but the body (after the ``:``) must consist of a single expression which will be considered the return value.

If you find yourself wanting a longer ``lambda`` it is generally a better idea to just define a full function.  This leads to clearer and more maintainable code.

Decorators
----------

Sometimes you'll find yourself with functions resembling::

    def check_account_balance(username, password):
        success = login(username, password)
        if success:
            ...
        else:
            raise InvalidUserError()

    def set_account_balance(username, password, val):
        user = login(username, password)
        if success:
            ...
        else:
            raise InvalidUserError()

It'd be nice not to have that same boiler plate in each function.

This is where **decorators** come in handy.  Decorators allow you to wrap a function call with another, for example::

    def login_decorator(oldfunc):
        def newfunc(username, password, *args, **kwargs):
            success = login(username, password)
            if success:
                return oldfunc(username, password, *args, **kwargs)
            else:
                raise InvalidUserError()
        return newfunc

    @login_decorator
    def check_account_balance(username, password):
        ...

    @login_decorator
    def set_account_balance(username, password, val):
        ...

This looks a bit scary, but let's break it down:

    * ``login_decorator`` is a function that takes a single argument ``oldfunc``, which is the function it is decorating.
    * Within this function we're defining another function ``newfunc`` that does our boilerplate and might call ``oldfunc`` if appropriate.
    * We then use ``@login_decorator`` which is the decorator syntax.  It is equivalent to ``set_account_balance = login_decorator(set_account_balance)``, essentially calling the function and replacing the old function with it.

Decorators are a powerful concept, when we look at Django we'll see many uses for them.

For now we'll move on to :doc:`generators-comprehensions`.
