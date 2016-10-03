Scalar Types
============

Python's types are similar to what you'd find in other dynamic languages.  This section will move pretty quickly, just showing off the major types and an example or two of their usage.  It might be worth looking over Python's `built-in types documentation <https://docs.python.org/3/library/stdtypes.html>`_.

We'll start with the **scalar** types.  A scalar is a type that can have a single value such as 5, 3.14, or 'Bob'.

The commonly used scalar types in Python are:

**int**
    Any integer.
**float**
    Floating point number (64 bit precision)
**complex**
    Numbers with an optional imaginary component.
**bool**
    True, False
**str**
    A sequence of characters (can contain unicode characters).
**bytes**
    A sequence of unsigned 8-bit entities, used for manipulating binary data.
**NoneType** (``None``)
    Python's null or nil equivalent, every instance of ``None`` is of ``NoneType``.

Numeric Types
-------------

Like most languages, Python has a separate type for integers and floating point numbers.

Unlike most languages Python's int type is unbounded.  There is no such thing as integer overflow in Python.

Python will do implicit conversions from integer to float when necessary.  This makes it possible to add ``3 + 2.5`` without worrying about the fact that they are different types.  One perhaps unexpected time this will happen is when dividing::

      >>> 10 / 4
      2.5 
      >>> 10 / 2
      5.0
          
You'll notice that even when the decimal part is 0, Python will give you a float.

Something that may seem unusual is the ``complex`` type.  Unless you're doing mathematical operations you probably won't see them but they're included for completeness.  They take the form::

      1+2j        # 1 is the real component and 2 is the imaginary component.  
      
They can be used like any other variable::

      >>> a = 1+2j
      >>> b = a * 2
      >>> b
      (2+4j)

Truthiness
----------

Unlike the limitless field of numbers.  A ``bool`` can only take on two values: ``True`` or ``False``.

Like many languages, Python has rules for coercion of other values to ``True`` and ``False`` allowing you to use other types where a boolean might be expected (such as an if statement).  You can also call ``bool(var)`` to cast ``var`` to ``True`` or ``False`` explicitly.

The rules for truthiness are as follows (some of these types we haven't seen yet)::

      type        | false value     
      ------------|-----------
      int         | 0         
      float       | 0         
      complex     | 0+0j     
      str         | ""        
      list        | []        
      dict        | {} 
      tuple       | ()
      set         | set()
      NoneType    | None

Strings
-------

Strings in Python are much like strings in other languages with first-class support for them.  They are sequences of characters, in Python 3's case they are full Unicode.

::

     a = 'hello world'
     b = '☃'          # snowman
     c = 'घङचछज'      # sanskrit

If you're coming from a language like C or Go, Python has no distinct ``char`` or ``rune`` type.

Strings can be concatenated with the ``+`` operator::

    >>> 'hello' + ' ' + 'world'
    'hello world'

It is also possible to use ``*`` to repeat a string::

    >>> 'hey ' * 3
    'hey hey hey '

Another common operation is **string formatting**, the putting other data inside of a string.  We do this using the ``.format`` method on the string instance.

The way this works is you insert placeholders like ``{}`` into the string, and then ``format()`` will fill them with its parameters.

Examples::

    >>> 'hello {}'.format('world')
    hello world
    >>> '{} {}'.format('hello', 'world')
    hello world
    >>> '{1} {0}'.format('world', 'hello')
    hello world
    >>> '{first} {second}'.format(first='hello', second='world')

As you can see, it is possible to either leave the placeholder blank, in which case it will use its position within the string to determine the order, or you can explicitly order them or name them.  We'll see how this works when we touch on function arguments.

`More on format <https://docs.python.org/3.4/library/string.html#formatspec>`_ and `other string methods <https://docs.python.org/3/library/stdtypes.html#string-methods>`_.

Bytes
-----

Prior to Python 3, Python strings were only allowed to have characters with values betweeen 0-255 (extended ASCII).  This made dealing with binary data as a string easy, but dealing with unicode required constant conversion between these strings and (the now removed) ``unicode`` type.

In order to make dealing with Unicode easier while still making it possible to deal with binary data, Python 3 introduced the ``bytes`` type and essentially replaced ``unicode`` with ``str`` as defined above.  ``bytes`` have many of the same features as ``str`` but without several that don't make sense such as those related to capitalization.

For now we'll leave it at that, but know that ``bytes`` is there for you when you need to process binary data.


Operators
---------

Types aren't much good if we can't do anything to them.  So now's probably a good time to cover some of the basic operators.

Commonly used operators::

    =    Assignment

    Arithmetic Operators
    --------------------
    +    Addition
    -    Subtraction
    *    Multiplication
    /    Division
    //   Floor Division
    %    Modulo
    **   Power
    

    Comparison Operators
    --------------------
    ==   Equal To
    >    Greater Than
    >=   Greater Than or Equal To
    <    Less Than
    <=   Less Than or Equal To
    !=   Not Equal
    
    
    Boolean Operators
    -----------------
    and
    or   
    not

Most of these are probably pretty familiar to you, but let's look at a few more examples:


Notice that we have a division and floor division operator.  Many languages treat ``/`` as floor division (Python did between ints until Python 3)

    >>> 3 / 2
    1.5
    >>> 3 // 2
    1
    >>> 3.0 // 2.0
    1.0


Another interesting/unexpected feature is that comparisons in Python are chainable:

    >>> 1 < 2 < 3
    True
    >>> 5 >= 4 == 4
    True

This lets you set up inequalities without the use of ``and`` in many cases::

    temp = 98.6

    # how you'd typically do a check
    if temp > 97.5 and temp < 99.5:
        print('pretty reasonable temperature')

    # or, perhaps easier to read as a range
    if 97.5 < temp < 99.5:
        print('pretty reasonable temperature')


And let's take a look at Python's implicit conversion to bools::

    >>> 3 and True
    True

OK, let's unpack that.  The first statement ``3 and True`` evaluated to ``True``, which is probably what you expected.  ``3`` is non-zero, so as per the table above it'll be considered truthy.

So let's try::

    >>> True and 3
    3

So why when we reverse the order does it return ``3``?

Python returns the final value evaluated that determines if the statement is True or not.  In an ``and`` that'll be the last value if they're all true.

Because Python 'short-circuits', meaning that it stops looking at values once it determines the entire expression will be false, Python will return the false value if one is false::

    >>> 0 and 'the truth'
    0

The ``or`` operator obeys the same rules, but short-circuiting rules means it will return the first true value it sees::

    >>> '' or 7
    7
    >>> 'first' or 'second'
    'first'

In practicality this doesn't often matter too much as ``and`` and ``or`` are typically used in conditionals ``if`` and ``while``, but it can be useful to know.

Moving On
---------

In the next section we'll look at the remaining basic types, representing sequences and mappings of other types.

Proceed to :doc:`more-types`.
