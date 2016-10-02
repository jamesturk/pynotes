Functions
=========

Functions in Python are a lot like functions in most languages, there are a few things to be aware of:

* all functions return a value, if no value is specified there's an implicit ``return None``
* functions can take any number of arguments, including named and unnamed variadic arguments
* arguments can have default values

Defining Functions
------------------

A function in Python takes the form::

    def func(a, b=2):
        return a + b

As noted above, an argument without a ``return`` statement has an implicit ``return None``::

    >>> def func():
    ...     print('hello')
    >>> x = func()
    hello
    >>> print(x)
    None

Default arguments can be overriden or ommitted::

    >>> def func(a, b=2):
    ...    return a + b
    >>> func(3)
    5
    >>> func(3, 0)
    3
    >>> func()
    TypeError: func() missing 1 required positional argument: 'a'

Functions are first-class objects, this means they can be assigned to variables::

    >>> def func(a, b=2):
    ...    return a + b
    >>> add2 = func
    >>> add2(2)
    4

Can be passed to functions::

    >>> def call_func_with_0(f):
    ...     print('going to call function', f)
    ...     return f(0)
    >>> call_func_with_0(add2)
    going to call function <function func at 0x105e4d400>
    2

And they have attributes even::

    >>> def call_func_with_0(f):
    ...     print('going to call function', f.__name__)
    ...     return f(0)
    >>> call_func_with_0(add2)
    going to call function func
    2

Argument Lists
--------------

Arguments can be passed **positional** or **by keyword**.

    >>> def lessthan(a, b):
    ...     return a < b
    >>> lessthan(1, 3)
    True
    >>> lessthan(b=1, a=3)
    False

Note that all positional arguments must preceed keyword arguments.

You can also let a function take an arbitrary list of positional arguments::

    >>> def add(*nums):
    ...     total = 0
    ...     for num in nums:
    ...         total += num
    ...     return total
    >>> add()
    0
    >>> add(1)
    1
    >>> add(1,2,3,4,5)
    15

``*nums`` packs all positional arguments into a tuple.

You can also let a function take an arbitary number of keyword arguments::

    >>> def mash(**monsters):
    ...     for k, v in monsters.items():
    ...         for i in range(v):
    ...             print(k)
    >>> mash(wolfman=3, mummy=2)
    mummy
    mummy
    wolfman
    wolfman
    wolfman

``**kwargs`` packs the keyword arguments into a dictionary.

It is possible to combine these.  You'll often see syntax like::

    def complex_function(arg1, arg2='default', *args, **kwargs):
        ...

There's nothing special about the names ``args`` and ``kwargs``, but they are often used in functions like this.

It can also be nice to have arguments that can only be passed by keyword, this can reduce errors in functions that take lots of arguments where you don't want people to rely on the order of the arguments::

    def setup_server(hostname,
                     port=80,
                     tls_port=443,
                     basic_auth_username=None,
                     basic_auth_password=None,
                     directory=None,
                     debug_mode=False)
    # it'd be easy to confuse the order of the 7 arguments

    def setup_server(hostname,
                     *,
                     port=80,
                     tls_port=443,
                     basic_auth_username=None,
                     basic_auth_password=None,
                     directory=None,
                     debug_mode=False)
    # now all arguments except the required hostname must be passed by keyword

.. note::

    A common gotcha with default arguments is assigning a mutable type to them::

        >>> def build_list(*items, start_list=[]):
        ...     for item in items:
        ...         start_list.append(item)
        ...     return start_list
        >>> build_list(1,2,3)
        [1,2,3]
        >>> build_list(1,2,3)
        [1,2,3,1,2,3]

    Wait... what?

    It turns out that declaration of ``start_list`` isn't called once per function, it is called in the declaration.  Most likely this isn't what you meant to do.

    If you want to use a mutable like a list or dict as a default argument usually you'll benefit from a pattern like::

        >>> def build_list(*items, start_list=None):
        ...     if start_list is None:
        ...         start_list = []
        ...     for item in items:
        ...         start_list.append(item)
        ...     return start_list
        >>> build_list(1,2,3)
        [1,2,3]
        >>> build_list(1,2,3)
        [1,2,3]

Lambdas
-------

Sometimes you want to pass a function as an argument but don't want to declare an entire function before doing so.  An example might be to the built-in ``sorted`` function that sorts an iterable and takes an optional ``key`` parameter which expects a function returning the value to sort by::

    >>> animals = ['cat', 'Dog', 'Fish']
    >>> sorted(animals)
    ['Dog', 'Fish', 'cat']

    >>> def lower(s):
    ...     return s.lower()
    >>> sorted(animals, key=lower)
    ['cat', 'Dog', 'Fish']

But we'd like to not create the extra function ``lower`` just for this purpose.

``lambda`` allows us to create a single expression function where that expression is the return value.  We'd rewrite our above example like so::

    >>> sorted(animals, key=lambda s: s.lower())
    ['cat', 'Dog', 'Fish']

Lambda can take any number of arguments (variable names before the ``:``) but can only have a single statement.  This statement is what is returned.

This means lambdas are inherently quite limited, this is intentional.  Long & complex lambdas would make code less readable, and are almost always better written as a named function for clarity.

Decorators
----------

Imagine a situation where you want to wrap a bunch of function calls in similar behavior, perhaps you're checking a credential before accessing sensitive information.

You might have a bunch of code like::

    def check_account_balance(username, password):
        user = login(username, password)
        if user:
            ...
        else:
            raise InvalidUserException()

    def set_account_balance(username, password, value):
        user = login(username, password)
        if user:
            ...
        else:
            raise InvalidUserException()

In many cases it is desirable to simplify the handling of the redundant code.  A decorator allows you to specify a function that 'wraps' another function, performing consistent behavior before and after.  We might rewrite the above like::

    def login_decorator(f):
        def new_function(username, password, *args, **kwargs):
            user = login(username, password)
            if user:
                f(user, *args, **kwargs)
            else:
                raise InvalidUserException()
        return new_function

    @login_decorator
    def check_account_balance(user):
        ...

    @login_decorator
    def set_account_balance(user, val):
        ...

This may seem complex but it builds on everything we've seen here:

    * ``login_decorator`` is a normal python function, it just happens to take a single arugment that is another function ``f`` and returns a new function that it defines on the fly that does the user logic and then calls the wrapped function ``f``.
    * We're capturing the extra arguments in ``*args`` and ``**kwargs`` and passing them along if present.
    * The only new piece is ``@login_decorator``, this is the call to the decorator.  It is equivalent to ``set_account_balance = login_decorator(set_account_balance)``, effectively replacing our defined function with whatever the return value of the decorator function is.
