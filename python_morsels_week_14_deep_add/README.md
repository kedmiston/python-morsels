CHALLENGE Email (7/2/2018):

Hiya!

This week I'd like you to write a function that takes a nested list of numbers and adds up all the numbers.

The phrase "nested list of numbers" might not be obvious. Here are examples of what I mean:
>>> deep_add([1, 2, 3, 4])
10
>>> deep_add([[1, 2, 3], [4, 5, 6]])
21
>>> deep_add([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])
36

So your deep_add function should accept lists of numbers, lists of lists of numbers, lists of lists of lists of
numbers, and so on. It should even work when there are multiple hierarchies involved or when there are empty lists:
>>> deep_add([1, [2, [3, 4], [], [[5, 6], [7, 8]]]])
36
>>> deep_add([[], []])
0

For the first bonus this week, I'd like you to make sure your function works with any iterable, not just lists. It
should work with tuples, sets, and even generators ✔️:
>>> deep_add([(1, 2), [3, {4, 5}]])
15

For the second bonus I'd like you to allow your function to accept an optional "start" value (which defaults to 0) ✔️:
>>> deep_add([[1, 2], [3, 4]], start=2)
12

For the third bonus, make sure your function works with anything number-like (anything you can use + on except for
iterables), not just numbers themselves ✔️:
>>> from datetime import timedelta
>>> deep_add([[timedelta(5), timedelta(10)], [timedelta(3)]], start=timedelta(0))
18

While you're at it, see if you can find a few different ways to solve this exercise. If you need a hint, read the fine
print at the end of this email.

Automated tests for this week's exercise can be found here. You'll need to write your function in a module named
deep_add.py next to the test file. To run the tests you'll run "python test_deep_add.py" and check the output for "OK".
You'll see that there are some "expected failures" (or "unexpected successes" maybe). If you'd like to do the bonus,
you'll want to comment out the noted lines of code in the tests file to test them properly.

Hint
I recommend attempting to use recursion for this one (passing the inner lists individually to the same function and
passing their inner lists to the same function until there are no more inner lists).


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SOLUTIONS email (7/4/2018):

Hiya!

If you haven't attempted to solve deep_add yet, close this email and go do that now before reading on. If you have
attempted solving deep_add, read on...

This week you needed to make a function that takes a nested list of numbers and adds up all the numbers given.

The simplest way to solve this exercise was through recursion.

Here's a solution that checks if the given value is a list and then recursively calls deep_add on each item in the list,
iterating deeper or adding up items as we go:
def deep_add(list_or_number):
    """Return sum of values in given list, iterating deeply."""
    total = 0
    if type(list_or_number) == list:
        for x in list_or_number:
            total += deep_add(x)
        return total
    else:
        return list_or_number
Here we're allowing deep_add to be used with lists or numbers and then we're calling deep_add from within itself
(calling a function inside itself is called recursion) to get sums of inner lists all the way down. Our "base case" in
this recursion is the else clause (the "list_or_number is not a list" case).

We can do something similar without recursion, but it's a little trickier:
def deep_add(lists):
    """Return sum of values in given list, iterating deeply."""
    total = 0
    lists = list(lists)
    while lists:
        item = lists.pop()
        if isinstance(item, list):
            lists.extend(item)
        else:
            total += item
    return total
Here we're copying our original incoming lists to make sure the caller of our function doesn't get unhappy (if we
didn't do this our original list would be modified because Python is call-by-assignment).

Then we loop repeatedly until we've processed all the lists. Each time we loop we pop off the last item and either
extend the list with it (if the last item was a list also) or add it to the total (if it was a number).

I'm going to focus on recursive solutions because they tend to be a little easier to work with for this problem.

Before we jump into the bonus, let's take a look at one way we could slightly rewrite our recursive solution:
def deep_add(list_or_number):
    """Return sum of values in given list, iterating deeply."""
    if type(list_or_number) == list:
        return sum(deep_add(x) for x in list_or_number)
    else:
        return list_or_number
Notice here that we're using the built-in sum function. The sum function accepts an iterable. The iterable we're
passing it is a generator object that we're making with the generator expression (deep_add(x) for x in list_or_number).
Because we're summing up our numbers using the sum function, we're able to remove our "total = 0" and "return total"
and "total +=" and in fact we've removed the total variable entirely.

Note that we're able to leave off the extra pair of parenthesis that would normally be around our generator expression.
Instead of sum((deep_add(x) for x in list_or_number)) we've written just sum(deep_add(x) for x in list_or_number). When
a generator expression is the only thing inside a set of parenthesis, Python allows you to drop the extra parenthesis.

If we really wanted to, we could even use an inline if statement in this function:
def deep_add(list_or_number):
    """Return sum of values in given list, iterating deeply."""
    return (
        sum(deep_add(x) for x in list_or_number)
        if type(list_or_number) == list
        else list_or_number
    )
That's up to personal style though. I don't find this solution more readable than the one before it.

Bonus #1
Okay let's attempt the first bonus now.

The first bonus asked that we accept any iterable to our function, not just lists.

Our current code won't work for this case because we're doing an isinstance check to see if the given argument is a
list. In Python we often practice the principle "it's easier to ask forgiveness than permission" (EAFP). We could
embrace this paradigm and attempt to loop over the given argument and then handle an exception if the argument isn't
an iterable:
def deep_add(iterable_or_number):
    """Return sum of values in given iterable, iterating deeply."""
    try:
        return sum(deep_add(x) for x in iterable_or_number)
    except TypeError:
        return iterable_or_number
This passes our first bonus! But it fails one of our other tests. When we pass invalid values like None, a TypeError
should be raised. It isn't right now.

We're not seeing a TypeError raised for invalid values because we're currently suppressing type errors everywhere in
our loop, including when we use the += operator on our items!

We could go back to using an "if" statement but flip our logic around, checking for numbers instead of lists:
def deep_add(iterable_or_number):
    """Return sum of values in given iterable, iterating deeply."""
    if isinstance(iterable_or_number, (int, float, complex)):
        return iterable_or_number
    else:
        return sum(deep_add(x) for x in iterable_or_number)
This also passes our bonus, but it fails one of our tests also. This solution fails for numeric types outside of the
ones we specified, like decimal.Decimal. We could try to keep track of all possible numeric types and put them in that
tuple in our isinstance check, but we can't possibly get all of them (especially since it's possible to invent your
own).

We can fix this problem by relying on the Number abstract base class in the numbers module:
from numbers import Number

def deep_add(iterable_or_number):
    """Return sum of values in given iterable, iterating deeply."""
    if isinstance(iterable_or_number, Number):
        return iterable_or_number
    else:
        return sum(deep_add(x) for x in iterable_or_number)
It's considered a best practice for all numeric types to register themselves with this abstract base class, so our code
should work for numeric types this way.

Bonus #2
Let's tackle the second bonus now.

For the second bonus, we were supposed to accept an optional "start" argument that would change the start value of our
sum (which should default to 0).
from numbers import Number

def deep_add(iterable_or_number, start=0):
    """Return sum of values in given iterable, iterating deeply."""
    if isinstance(iterable_or_number, Number):
        return iterable_or_number
    else:
        total = start
        for x in iterable_or_number:
            total += deep_add(x)
        return total
Here we're doing the same thing we were doing before, but we've added start argument that defaults to 0 and we're using
that to initialize our "total" variable.

Notice that I've moved back to the "for" loop += approach instead of using a generator expression with the sum function
here. We could move back to using the sum function if we pass in the start to our sum call:
from numbers import Number

def deep_add(iterable_or_number, start=0):
    """Return sum of values in given iterable, iterating deeply."""
    if isinstance(iterable_or_number, Number):
        return iterable_or_number
    else:
        return sum((deep_add(x) for x in iterable_or_number), start)
Notice that we have to add back the parenthesis around our generator expression because it's not the only thing in the
sum function call. That's unfortunate, but I still find this more readable than our loop approach.

Cool! That bonus was probably a bit simpler than the first one.

Bonus #3
Let's take a look at the third bonus.

For this bonus we were supposed to make sure our deep_add works even for pseudo-numeric types like datetime.timedelta.
It can sometimes be handy to add up types that aren't strictly numbers but do operate like numbers with the + operator.

We've already determined that we can't do an isinstance check to see if the argument given to us is a list and we also
can't do an isinstance check to see whether it's a number. We also know that we can't try to loop over it and handle
TypeErrors because that also caught TypeErrors from addition.

We could try to check if the given argument is an iterable though. If you've read my article on how for loops work in
Python, you know that you can get an iterator from any iterable by using the built-in iter function. So if we call
iter() on the argument given to us and we don't get a TypeError, we know that we're working with an iterable. Let's
make a function that uses that fact:
def deep_add(iterable_or_number, start=0):
    """Return sum of values in given iterable, iterating deeply."""
    try:
        iter(iterable_or_number)
    except TypeError:
        return iterable_or_number
    else:
        return sum((deep_add(x) for x in iterable_or_number), start)

Here we're attempting to get an iterator from the argument given to us. If a TypeError occurs while we're doing that,
we'll treat the argument as a number. If no exceptions occurred (that's what try-else means), then we'll loop over the
argument and recursively add up its items.

You might instead decide to get clever and not actually call iter() but instead check to see whether our iterable has
a __iter__:
def deep_add(iterable_or_number, start=0):
    """Return sum of values in given iterable, iterating deeply."""
    if hasattr(iterable_or_number, '__iter__'):
        return sum((deep_add(x) for x in iterable_or_number), start)
    else:
        return iterable_or_number
This works for most iterables, but it doesn't work for some built-ins in Python 2 and it doesn't necessarily work for
all iterables. It's possible to make a sequence without specifying a __iter__. Don't rely on dunder method checking
unless you really understand how the high-level operations (like the iter function) interact with and rely on the
various dunder methods (like __iter__).

There is a better way to test for iterability without directly calling the iter function though.

Python has an abstract base class called Iterable in the collections.abc module:
from collections.abc import Iterable

def deep_add(iterable_or_number, start=0):
    """Return sum of values in given iterable, iterating deeply."""
    if isinstance(iterable_or_number, Iterable):
        return sum((deep_add(x) for x in iterable_or_number), start)
    else:
        return iterable_or_number
Now you might think "wait a minute... if things can be number-like but not inherit from numbers.Number, won't we have
the same problem with collections.abc.Iterable and custom iterables?" And this is a valid point!

It's a valid point, but that's actually not how collections.abc.Iterable works. If we make an Iterable that doesn't
inherit from collections.abc.Iterable, it will still answer True to the question isinstance(custom_iterable, Iterable):
>>> class I:
...     def __iter__(self):
...         return iter([])
...
>>> x = I()
>>> from collections.abc import Iterable
>>> isinstance(x, Iterable)
True
This is a little odd. Normally isinstance only answers True if objects actually inherit from the given type. The
Iterable class uses a metaclass abc.ABCMeta, which defines a __subclasscheck__ that allows Iterable to customize the
behavior of isinstance.

So the behavior of the isinstance function in Python is actually customizable for different types, the same way len()
and iter() are.

So we're practicing "duck typing" (checking whether our iterable is actually an iterable) and not strict type checking
here.

If this section on isinstance, Iterable, duck typing, and meta classes didn't make sense, don't worry much. This is
pretty advanced stuff that is interesting to know but definitely not essential to understand deeply. Feel free to email
me questions on this.

I hope you learned something from these solutions!