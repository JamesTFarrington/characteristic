.. _why:

Why not…
========


…tuples?
--------


Readability
^^^^^^^^^^^

What makes more sense while debugging::

   <Point(x=1, y=2)>

or::

   (1, 2)

?

Let's add even more ambiguity::

   <Customer(id=42, reseller=23, first_name="Jane", last_name="John")>

or::

   (42, 23, "Jane", "John")

?

Why would you want to write ``customer[2]`` instead of ``customer.first_name``?

Don't get me started when you add nesting.
If you've never ran into mysterious tuples you had no idea what the hell they meant while debugging, you're much smarter then I am.

Using proper classes with names and types makes program code much more readable and comprehensible_.
Especially when trying to grok a new piece of software or returning to old code after several months.

.. _comprehensible: http://arxiv.org/pdf/1304.5257.pdf


Extendability
^^^^^^^^^^^^^

Imagine you have a function that takes or returns a tuple.
Especially if you use tuple unpacking (eg. ``x, y = get_point()``), adding additional data means that you have to change the invocation of that function *everywhere*.

Adding an attribute to a class concerns only those who actually care about that attribute.


…namedtuples?
-------------

The difference between namedtuple_\ s and classes decorated by ``characteristic`` is that the latter are type-sensitive and less typing aside regular classes:


.. doctest::

   >>> from characteristic import Attribute, attributes
   >>> @attributes([Attribute("a", instance_of=int)])
   ... class C1(object):
   ...     def __init__(self):
   ...         if self.a >= 5:
   ...             raise ValueError("'a' must be smaller than 5!")
   ...     def print_a(self):
   ...         print self.a
   >>> @attributes([Attribute("a", instance_of=int)])
   ... class C2(object):
   ...     pass
   >>> c1 = C1(a=1)
   >>> c2 = C2(a=1)
   >>> c1.a == c2.a
   True
   >>> c1 == c2
   False
   >>> c1.print_a()
   1
   >>> C1(a=5)
   Traceback (most recent call last):
      ...
   ValueError: 'a' must be smaller than 5!


…while namedtuple’s purpose is *explicitly* to behave like tuples:


.. doctest::

   >>> from collections import namedtuple
   >>> NT1 = namedtuple("NT1", "a")
   >>> NT2 = namedtuple("NT2", "b")
   >>> t1 = NT1._make([1,])
   >>> t2 = NT2._make([1,])
   >>> t1 == t2 == (1,)
   True


This can easily lead to surprising and unintended behaviors.

Other than that, ``characteristic`` also adds nifty features like type checks or default values.

.. _namedtuple: https://docs.python.org/2/library/collections.html#collections.namedtuple
.. _tuple: https://docs.python.org/2/tutorial/datastructures.html#tuples-and-sequences


…hand-written classes?
----------------------

While I'm a fan of all things artisanal, writing the same nine methods all over again doesn't qualify for me.
I usually manage to get some typos inside and there's simply more code that can break and thus has to be tested.

To bring it into perspective, the equivalent of

.. doctest::

   >>> @attributes(["a", "b"])
   ... class SmartClass(object):
   ...     pass
   >>> SmartClass(a=1, b=2)
   <SmartClass(a=1, b=2)>

is

.. doctest::

   >>> class ArtisanalClass(object):
   ...     def __init__(self, a, b):
   ...         self.a = a
   ...         self.b = b
   ...
   ...     def __repr__(self):
   ...         return "<ArtisanalClass(a={}, b={})>".format(self.a, self.b)
   ...
   ...     def __eq__(self, other):
   ...         if other.__class__ is self.__class__:
   ...             return (self.a, self.b) == (other.a, other.b)
   ...         else:
   ...             return NotImplemented
   ...
   ...     def __ne__(self, other):
   ...         result = self.__eq__(other)
   ...         if result is NotImplemented:
   ...             return NotImplemented
   ...         else:
   ...             return not result
   ...
   ...     def __lt__(self, other):
   ...         if other.__class__ is self.__class__:
   ...             return (self.a, self.b) < (other.a, other.b)
   ...         else:
   ...             return NotImplemented
   ...
   ...     def __le__(self, other):
   ...         if other.__class__ is self.__class__:
   ...             return (self.a, self.b) <= (other.a, other.b)
   ...         else:
   ...             return NotImplemented
   ...
   ...     def __gt__(self, other):
   ...         if other.__class__ is self.__class__:
   ...             return (self.a, self.b) > (other.a, other.b)
   ...         else:
   ...             return NotImplemented
   ...
   ...     def __ge__(self, other):
   ...         if other.__class__ is self.__class__:
   ...             return (self.a, self.b) >= (other.a, other.b)
   ...         else:
   ...             return NotImplemented
   ...
   ...     def __hash__(self):
   ...         return hash((self.a, self.b))
   >>> ArtisanalClass(a=1, b=2)
   <ArtisanalClass(a=1, b=2)>

which is quite a mouthful and it doesn't even use any of ``characteristic``'s more advanced features like type checks or default values
Also: no tests whatsoever.
And who will guarantee you, that you don't accidentally flip the ``<`` in your tenth implementation of ``__gt__``?

If you don't care and like typing, I'm not gonna stop you.
But if you ever get sick of the repetitiveness, ``characteristic`` will be waiting for you. :)
