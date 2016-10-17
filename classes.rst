Classes
=======

The tools we've seen in the last two chapters are geared towards **procedural** and **functional** programming.

As we mentioned up front, Python is a hybrid language that doesn't force a particular paradigm on you, but it is, at its core an object based language.

Unlike other object oriented languages like Java, this fact can remain mostly in the background until it is needed.  Now however, we're going to peel back the facade and look at Python's object system.  Let's start by looking at some things we've already seen.

Basic Types
-----------

In a few examples we've used the builtin ``type`` function.  Let's look at how it works::

    >>> type(4)
    int
    >>> type(lambda: 5)
    function
    >>> type(None)
    NoneType
    >>> type('hello')
    str
    >>> type([1,2,3])
    list

What are these values coming back from the ``type`` function though?

::

    >>> type(type(4))
    type
    >>> type(type(type(4)))
    type

So all of our values we've seen so far have a ``type``.

We can also use the builtin ``isinstance`` to check if a value if of a certain type::

    >>> isinstance(4, int)
    True
    >>> isinstance(4.0, int)
    False

This can be useful when that whole **duck-typing** thing that we learned about in the introduction doesn't meet our needs.  Sometimes you want to be sure you're dealing with a specific type.

``object``
''''''''''

Within Python all values are objects.  Like in traditional object-oriented languages this means they can have attributes and methods. 

We've seen a few already, here are some examples::

    >>> c = 1+2j
    >>> c.imag
    2.0
    >>> s = 'this is a string'
    >>> s.title()
    This Is A String
    >>> x = 2**8
    >>> x.bit_length()
    9

In fact we can even verify this fact using ``isinstance``::

    >>> isinstance(4, object)
    True
    >>> isinstance('string', object)
    True
    >>> isinstance(lambda: 4, object)
    True

But weren't all of these things other types?

In fact that's because ``isinstance`` takes the concept of subclasses into account.  Something is an instance of a class if it is an instance of that class or a *child* of that class.  Let's use ``issubclass`` to check this out::

    >>> issubclass(int, object)
    True
    >>> issubclass(str, object)
    True
    >>> issubclass(list, object)
    True

**Every type is a subclass of** ``object``.

Defining Classes
----------------

Of course, the next step is to define our own types.  We do this by using the ``class`` keyword.

Let's start by looking at an example::

    class Item:
        def __init__(self, name, number=0):
            self.name = name
            self.number = 0

        def restock(self, number):
            self.number += number

        def sell_item(self):
            if self.number > 0:
                self.number -= 1
            else:
                raise ValueError("sold out!")

Which we'd then use like::

    >>> a = Item('corn', 10)
    >>> a.sell_item()
    >>> a.number
    9
    >>> a.restock(50)
    >>> a.number
    59

So let's break this down:

First we see ``class Item``, this defines the name of our new type.  You may also see people write ``class Item(object):``, this was encouraged in Python 2 and works fine in Python 3, but is not required.

We define a few functions inside the class body.  These will be **methods** on the class by default.

The first method we define is ``__init__``, this is the **constructor** and is the method that will be called when you instantiate the class via ``Item('corn')``.  Note that we can use our normal function syntax here, including default and variadic arguments.

But what's this first parameter? ``self``  This is how Python knows which instance of the class the method is being called upon.  Unlike other languages there's no special ``this``, instead ``self`` is passed implicitly.  You *could* name this variable anything you wish, but 99% of the time the right thing to call it is *self*.

We can then assign to and access members on ``self``, there's no need to declare them but typically you'll initialize them in the ``__init__`` function so you can use them in other functions without checking if they've been defined first.

From there the remaining functions work the same way, just notice that they each take ``self`` as their first argument.

Warning: forgetting ``self``
''''''''''''''''''''''''''''

Let's define our class again but forget ``self`` on a method::

    class BrokenItem:
        def __init__(self, name, number=0):
            self.name = name
            self.number = 0

        def restock(number):            # Notice the error here.
            self.number += number

::

    >>> bi = BrokenItem('egg')
    >>> bi.restock(40)
    TypeError: sell() takes 1 positional argument but 2 were given

This is a particularly unfriendly error, what is happening here?

When we call ``bi.restock(40)`` Python sees it as ``BrokenItem.restock(bi, 40)`` - passing the current instance of ``BrokenItem`` as the first parameter.

Usually this would be assigned to ``self``, but in this case it gets assigned to ``number``, and then the ``40`` is the mystery second positional argument.

It is good to understand this error as it is easy to forget to include ``self`` in method definitions, especially if you are switching between languages.

Properties
----------

If you're coming from other languages you may be used to writing accessor functions like **getName** and **setCount**.  In Python we typically don't write these, allowing people to write directly to members.  In the above example though maybe we want to protect the number so that people have to restock & sell through the methods we provide.  For this case we'd use the following pattern::

    class Item:
        def __init__(self, name, number=0):
            self.name = name
            self._number = 0            # we modify the name of the instance variable

        def restock(self, number):
            self._number += number

        def sell_item(self):
            if self._number > 0:
                self._number -= 1
            else:
                raise ValueError("sold out!")

        @property
        def number(self):
            return self._number

What we've done here is prefix the 'private' variable with an underscore, this doesn't actually prevent anyone from accessing it, but is a signal to other developers that it isn't intended to be accessed directly.  All of our internal uses have been updated to use ``self._number``.

We still want people to be able to check the number, so we have added a property.  Let's look at what happens when we try to access ``.number``::

    >>> i = Item('honey', 10)
    >>> i.number
    10
    >>> i.number = 9
    AttributeError: can't set attribute

So we've effectively made this attribute read-only.

It is also possible to make read-write properties::

    class Item:
        def __init__(self, name, number=0):
            self.name = name
            self._number = 0            # we modify the name of the instance variable

        def restock(self, number):
            self._number += number

        def sell_item(self):
            if self._number > 0:
                self._number -= 1
            else:
                raise ValueError("sold out!")

        @property
        def number(self):
            return self._number

        @number.setter
        def number(self, value):
            if value >= 0:
                self._number = value
            else:
                raise ValueError('must be non-negative')


Which would use the new function during assignment, guarding against negative values being assigned.

Full `property docs <https://docs.python.org/3/library/functions.html#property>`_.

The actual implementation of property is beyond the scope of what we'll cover here, they are implementing the **descriptor** protocol.  (Read `more on descriptors <https://docs.python.org/3/howto/descriptor.html>`_).

Operators
---------

``__init__`` isn't the only special method you can define on a class.  These are often called *magic*, *double underscore*, or *dunder* methods.  These methods are typically called by invoking an operator like ``+`` or ``<=``.

Let's look at an example::

    class Animal:
        def __init__(self, name):
            self.name = name

        def __add__(self, other):
            return Animal(self.name + '-' + other.name)

::
    >>> cow = Animal('cow')
    >>> pig = Animal('pig')
    >>> child = cow + pig
    >>> child.name
    cow-pig

    >>> child
    <__main__.Animal at 0x1075071d0>

So ``__add__`` is called when we use the ``+`` operator on an ``Animal``.

But what is the deal with that ugly representation of our class?

Let's add another magic method::

    class Animal:
        def __init__(self, name):
            self.name = name

        def __add__(self, other):
            return Animal(self.name + '-' + other.name)

        def __repr__(self):
            return 'Animal: ' + self.name

::

    >>> Animal('cow') + Animal('pig')
    Animal: cow-pig

``__repr__`` governs what happens when the builtin ``repr`` method is called.  The method is supposed to return a machine-oriented representation of the class.  Often you'll see these include class names and memory addresses if that's all that Python knows to show.  The interactive Python interpreter will display the ``repr`` of each statement as we go, which is why overriding it here gives us better output.

You'll often also want to override ``__str__`` which controls how the instance is converted to a string, for example in a ``print`` statement.

Essentially every operator and many of the builtin methods can be overriden by using appropriate methods on the class.

Here are a few we've already seen:

``__len__``
    Overrides the response of the instance to ``len()``::

        class Snake:
            def __init__(self, eyes, length):
                self.eyes = eyes
                self.length = length
            def __len__(self):
                return self.length

::

        >>> s = Snake(2, 10)
        >>> len(s)
        10

``__next__``
    Can be used to convert a class into a generator::

        class Infinite:
            def __init__(self):
                self.n = 0
            def __next__(self):
                self.n += 1
                return self.n

::

        >>> inf = Infinite()
        >>> next(inf)
        1
        >>> next(inf)
        2
        >>> next(inf)
        3

``__call__``
    Can be used to make a class instance callable as a function::

        class AddN:
            def __init__(self, n):
                self.n = n
            def __call__(self, val):
                return self.n + val

::

        >>> add_five = AddN(5)
        >>> add_five(3)         # invokes __call__
        8

`Python's datamodel reference <https://docs.python.org/3/reference/datamodel.html>`_ covers the entire list.


Inheritance
-----------

Like many object oriented languages Python allows classes to inherit from one another. 

Inheritance means that a child class will have the same attributes and methods as the parent class, and can add (or override) these methods.

Let's look at an example::

    class Square:
        def __init__(self, w, h):
            self.w = w
            self.h = h

        def whoami(self):
            print('square')

        def area(self):
            return self.w * self.h


    class Cube(Square):
        def __init__(self, w, h, d):
            super().__init__(w, h)
            self.d = d

        def whoami(self):
            print('cube')

        def volume(self):
            return self.w * self.h * self.d

::

    >>> s = Square(3, 4)
    >>> c = Cube(2, 3, 4)
    >>> s.whoami()
    square
    >>> c.whoami()
    cube
    >>> s.area()
    12
    >>> c.volume()
    24
    >>> isinstance(s, Square)
    True
    >>> isinstance(c, Square)
    True
    >>> isinstance(s, Cube)
    False
    >>> isinstance(c, Cube)
    True
    >>> issubclass(Cube, Square)
    True

A few things to note:

    * We use ``super()`` to access the parent's methods.  There's a lot more that ``super()`` can do, but in Python 3 this is the most common & easiest usage.
    * ``whoami`` is overridden, if you're coming from a language like C++ you may notice that this is done without any special syntax (``virtual`` or its ilk)
    * ``volume`` is an entirely new method, and only available on the subclass.
    * ``isinstance`` will tell us that a ``Cube`` instance is a ``Square`` but not vice versa.
    * ``issubclass`` operates on types, and helps us verify that ``Cube`` is indeed a subclass of ``Square``.

Note that any methods can be overridden, including special ``__dunder__`` methods.  In fact, when we define any class we're subclassing ``object`` and overriding
the defaults that it provides.

Just to hammer home this point let's subclass ``int`` and ``list``::

    >>> class badint(int):
    ...    def __add__(self, other):
    ...        return self - other

    >>> a = badint(4)
    >>> a + 5
    -1

    >>> class badlist(list):
    ...    def __getitem__(self, index):
    ...        return ':P'

    >>> b = badlist([1,2,3,4])
    >>> b[0]
    ':P'
    >>> b[500]
    ':P'

Multiple Inheritance
''''''''''''''''''''

It is also possible to have a class subclass multiple other classes.  This is known as multiple inheritance and opens up an order of magnitude more confusing situations.

The most important detail is that method calls (and calls to ``super()``) will walk through the "MRO" or method resolution order.  This is generally defined by the order you specify your subclasses.

This means if two subclasses specify a method or attribute with the same name, the one specified first will win.  For example::

    class Color:
        def __init__(self, name, hex):
            self.name = name
            self.hex = hex

        def identify(self):
            return 'Color {} ({})'.format(self.name, self.hex)

    class Fruit:
        def __init__(self, name, juicy):
            self.name = name
            self.juicy = juicy

        def identify(self):
            return 'Fruit {} is {} juicy'.format(self.name, self.juicy)

    class Orange(Color, Fruit):
        def __init__(self):
            super().__init__('orange', '#ffa500')
            Fruit.__init__(self, 'orange', 2)


We can see that by default ``super()`` called the first listed parent::

    >>> o = Orange()
    >>> o.identify()
    'Color orange (#ffa500)'

We can explicitly check this via the ``__mro__`` (method resolution order)::

    >>> Orange.__mro__
    (__main__.Orange, __main__.Color, __main__.Fruit, object)

Explicit calls to other conflicting properties are possible, but generally avoided::

    >>> Fruit.identify(o)
    'Fruit orange is 2 juicy'
