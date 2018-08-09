CHALLENGE Email (6/11/2018):

Hey! 👋

I want you to write a function that accepts an iterable and an object and returns a new iterable with all items from
the original iterable except any item at the beginning of the iterable which is equal to the object should be skipped.

Here's an example:
>>> lstrip([0, 0, 1, 0, 2, 3], 0)
[1, 0, 2, 3]
>>> lstrip('  hello ', ' ')
['h', 'e', 'l', 'l', 'o', ' ']

As a bonus, return an iterator (for example a generator) from your lstrip function instead of a list. ✔️
>>> x = lstrip([0, 1, 2, 3], 0)
>>> x
<generator object <genexpr> at 0x7f0b18b0fbf8>
>>> list(x)
[1, 2, 3]
>>> list(x)
[]

Here's another bonus to do after you've made your lstrip function return a lazy iterable: allow your lstrip function to
accept a function as its second argument which will determine whether the item should be stripped. ✔️

The function will be executed with each item individually and as long as the function returns True the item should be
removed from the beginning of the iterable.

For example:
>>> def is_falsey(value): return not bool(value)
>>> list(lstrip(['', 0, 1, 0, 2, 'h', ''], is_falsey))
[1, 0, 2, 'h', '']
>>> list(lstrip([-4, -2, 2, 4, -6], lambda n: n < 0))
[2, 4, -6]

Automated tests for this week's exercise can be found here. You'll need to write your function in a module named
lstrip.py next to the test file. To run the tests you'll run "python test_lstrip.py" and check the output for "OK".
You'll see that there are some "expected failures" (or "unexpected successes" maybe). If you'd like to do the bonus,
you'll want to comment out the noted lines of code in the tests file to test them properly.


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SOLUTIONS email (6/13/2018):

Hi! 😄

If you haven't attempted to solve lstrip yet, close this email and go do that now before reading on. If you have
attempted solving lstrip, read on...

To remove the strip value from the beginning of our new iterable, we could use an is_beginning variable to determine
whether we've seen any values that don't match our strip value yet.  As long as we've only seen our strip value, we're
still at the "beginning" of our iterable and should continue skipped values.
def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    stripped = []
    is_beginning = True
    for item in iterable:
        if is_beginning:
            if item != strip_value:
                is_beginning = False
            else:
                continue
        stripped.append(item)
    return stripped
We could also rearrange this slightly to always set is_beginning.  This doesn't make our code much cleaner though:
def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    stripped = []
    is_beginning = True
    for item in iterable:
        if is_beginning and item == strip_value:
            continue
        is_beginning = False
        stripped.append(item)
    return stripped
If we're concerned looping over long iterables that may not have many items to strip from the front, we could try
embracing the iterator protocol and use a while loop to manually remove values until we find a value we don't need to
strip and then we could continue looping with a for loop.
def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    stripped = []
    iterator = iter(iterable)
    try:
        item = next(iterator)
        while item == strip_value:
            item = next(iterator)
        stripped.append(item)
    except StopIteration:
        pass
    else:
        for item in iterator:
            stripped.append(item)
    return stripped
That code is a little bit silly though because we've really just re-implemented a for loop fully. If you don't
understand why that is, check out my talk on looping. What we really want is a for loop before a for loop.
def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    stripped = []
    iterator = iter(iterable)
    for item in iterator:
        if not item == strip_value:
            stripped.append(item)
            break
    for item in iterator:
        stripped.append(item)
    return stripped
That first for loop keeps looping until we find a value that shouldn't be stripped and then we append the value and
break. The Python standard library has a helper function for doing what we're hoping to do.  The dropwhile function in
the itertools module will give us an iterable that has our original iterable's values except it drops any at the
beginning that pass a certain test.
from itertools import dropwhile

def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    stripped = []
    def is_strip_value(value): return value == strip_value
    for item in dropwhile(is_strip_value, iterable):
        stripped.append(item)
    return stripped
The is_strip_value function is our test for dropwhile.  As long as that function returns True, values are dropped from
the beginning of our new iterable.  Once it's False it stops dropping values.

Note that Python allows us to put functions inside of functions. We can use a lambda statement to make an anonymous
function if we wanted to, but I prefer to name my functions and using def to define functions often makes your code
easier to understand for newer Python programmers.

Because dropwhile returns an iterable itself, we could just return it from our function:
from itertools import dropwhile

def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    def is_strip_value(value): return value == strip_value
    return dropwhile(is_strip_value, iterable)

Bonus #1
Okay let's talk tackle the first bonus. We're supposed to make lstrip return an iterator.

It turns out that dropwhile returns an iterator, so we've already done the first bonus.

Let's look at another bonus solution.  If we start with our two for loops solution, we could pass the bonus by making
our function into a generator function:
def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    iterator = iter(iterable)
    for item in iterator:
        if item != strip_value:
            yield item
            break
    for item in iterator:
        yield item
We're yielding values instead of appending them to a list which is how we make a generator function.

We can actually improve this slightly by using Python 3's "yield from":
def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    iterator = iter(iterable)
    for item in iterator:
        if item != strip_value:
            yield item
            break
    yield from iterator
Whenever you see "for item in iterable: yield item" you can instead write "yield from item".

Bonus #2
Let's try the second bonus.  For this one we're supposed to optionally accept a function as our strip value and call
that function to determine whether values should be removed.

We could do this with our generator function by using the built-in callable function to check whether our strip value
is a callable:
def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    iterator = iter(iterable)
    for item in iterator:
        if (callable(strip_value) and not strip_value(item)
                or not callable(strip_value) and item != strip_value):
            yield item
            break
    yield from iterator
That "if" statement really complex. Here's a simpler way to write this:

def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    iterator = iter(iterable)
    if callable(strip_value):
        predicate = strip_value
    else:
        def predicate(value): return value == strip_value
    for item in iterator:
        if not predicate(item):
            yield item
            break
    yield from iterator

Here we're either using the predicate function that was given to us (strip_value) if it is a function or we're creating
our own. Then we're looping over the items until we find a match and then yielding and breaking once we do. After that
we keep looping as usual, yielding each value along the way.

We are kind of re-implementing dropwhile here though. We could take the function we end up taking/creating and pass it
straight to dropwhile and it'll do the rest of the work for us.
from itertools import dropwhile

def lstrip(iterable, strip_value):
    """Return iterable with strip_value items removed from beginning."""
    if callable(strip_value):
        predicate = strip_value
    else:
        def predicate(value): return value == strip_value
    return dropwhile(predicate, iterable)
If you didn't know about dropwhile before this email, I'd highly recommend skimming the itertools documentation and
take a look at the many functions for working with iterables in a lazy and efficient fashion.

I hope you learned something new this week!