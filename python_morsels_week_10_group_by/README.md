CHALLENGE Email (6/4/2018):

Hi again!

This week I want you to make a function that takes an iterable and a key function and returns a dictionary of items
grouped by the values returned by the given key function.

For example, let's say we have a list of numbers, and a function that checks their remainder when we divide them by
three.
>>> numbers = [1, 4, 5, 6, 8, 19, 34, 55]
>>> def mod3(n): return n % 3
...

If we call our function with this list and key function, we should get the numbers back in a dictionary of numbers tied
to their remainders (0, 1, 2).
>>> group_by(numbers, key_func=mod3)
{0: [6], 1: [1, 4, 19, 34, 55], 2: [5, 8]}

As a bonus, create a default behavior of returning the value back in the case that no key function is given. ✔️
>>> group_by([1, 2, 1, 3, 2, 1])
{1: [1, 1, 1], 2: [2, 2], 3: [3]}

If you finish the exercise and the bonus, also consider whether there's something that could be helpful for solving
this in Python's standard library. You don't necessarily need to return a dictionary for this one, but you do need to
return a dictionary-like object for the tests to pass.

Automated tests for this week's exercise can be found here. You'll need to write your function in a module named
group_by.py next to the test file. To run the tests you'll run "python test_group_by.py" and check the output for "OK".
You'll see that there are some "expected failures" (or "unexpected successes" maybe). If you'd like to do the bonus,
you'll want to comment out the noted lines of code in the tests file to test them properly.


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SOLUTIONS email (6/6/2018):

If you haven't attempted to solve group_by yet, close this email and go do that now before reading on. If you have
attempted solving group_by, read on...

This week you needed to make a function that takes an iterable and a key function and returns a dictionary of items
grouped by the values returned by the given key function.

The simplest way to write the group_by function is to use a dictionary and an if statement.  As we loop over the items
in our iterable, we can check whether each item has a key in our dictionary or not.  If the key in not yet in our
dictionary, we'll add it with an empty list as the value.  We'll always append to the list as we loop.
def group_by(iterable, key_func):
    groups = {}
    for item in iterable:
        key = key_func(item)
        if key not in groups:
            groups[key] = []
        groups[key].append(item)
    return groups
This if statement could be refactored a number of ways.  An else block could be used along with the if block.  Or a
try-except block could be used.

We could also use the get method to both get the list or default the list to an empty list and then assign back to
the same key:
def group_by(iterable, key_func):
    groups = {}
    for item in iterable:
        key = key_func(item)
        groups[key] = groups.get(key, [])
        groups[key].append(item)
    return groups
That's a bit messy though.  There's not really an elegant way to refactor that to one line while still using the get
method unless we stopped appending and started making a new list every time, which wouldn't be very efficient.

The setdefault method was invented for dictionaries to solve just this situation:
def group_by(iterable, key_func):
    groups = {}
    for item in iterable:
        key = key_func(item)
        groups.setdefault(key, []).append(item)
    return groups
The setdefault method does two things: it returns the value for the given key from the dictionary and sets the value to
the given key (an empty list) if the key isn't set yet.  So the append is appending to either the new empty list or the
existing value at that key.

I rarely use the setdefault method because I often find that it does a lot all at once and it doesn't make things any
more clear than using an if statement.

When I find myself using the dictionary setdefault method I instead sometimes reach for a completely different
dictionary-like object: defaultdict from the collections module.
from collections import defaultdict

def group_by(iterable, key_func):
    groups = defaultdict(list)
    for item in iterable:
        groups[key_func(item)].append(item)
    return groups
This defaultdict object is kind of cool.  Whenever a missing key is accessed, defaultdict will call the callable that
was given to it (list in this case) and use the return value as the new value for that key.

Notice that with both the setdefault method and with defaultdict we only need to reference the key once.  With
defaultdict, our code line is so short it makes sense to put the key_func(item) access all on one line.

So groups[key_func(item)] defaults to setting the given key to an empty list and whatever list we get back we can then
append to.

While searching for ways of grouping things in Python, you might have discovered the groupby utility in the itertools
module.  This works for grouping things, but it only works when our items-to-be-grouped are all consecutive.  We could
try to still use it if we sort our items by their keys first:
from itertools import groupby

def group_by(iterable, key_func):
    groups = {}
    iterable = sorted(iterable, key=key_func)
    for key, items in groupby(iterable, key=key_func):
        groups[key] = list(items)
    return groups

The groupby utility gives us back our key and a lazy iterator object of the items for each key.  We're using list(items)
to convert that iterator object to a list because that's what we want for the values in our dictionary.

You might notice that we could use a dictionary comprehension here:
from itertools import groupby

def group_by(iterable, key_func):
    iterable = sorted(iterable, key=key_func)
    return {
        key: list(items)
        for key, items in groupby(iterable, key=key_func)
    }
While these groupby solutions work, they're a little silly because they require that you get a new sorted version of
our given iterable first.  That will take up both more memory (temporarily at least) and more time.  I prefer the
defaultdict(list) approach the most.

Bonus
Okay let's talk about the bonus now. We were supposed to make the key function argument optional.

We could do that by defaulting key_func to None and then re-assigning it if it is None:
from collections import defaultdict

def group_by(iterable, key_func=None):
    groups = defaultdict(list)
    if key_func is None:
        key_func = lambda x: x
    for item in iterable:
        groups[key_func(item)].append(item)
    return groups
That lambda thing makes an anonymous function.  An anonymous function is a function that doesn't have a name.  You can
create anonymous functions and then immediately pass them around without assigning a variable name to them.

PEP8, the Python style guide, says that we should never write "key_func = lamdba x: x".  Specifically it says that if
we're using lambda to make a function and then assign it to a variable, we should use a def statement to make a function
instead:
from collections import defaultdict

def group_by(iterable, key_func=None):
    groups = defaultdict(list)
    if key_func is None:
        def key_func(x): x
    for item in iterable:
        groups[key_func(item)].append(item)
    return groups
The reason for this is that Python programmers tend to know what def means but lambda is more mysterious.  If you're
giving an anonymous function a name, just use def and make the function not anonymous anymore.

We can actually take this a little further though.  Instead of defaulting key_func to None we could default it to a
function.  We'll use a lambda expression since that allows us to do this on one line:
from collections import defaultdict

def group_by(iterable, key_func=lambda x: x):
    groups = defaultdict(list)
    for item in iterable:
        groups[key_func(item)].append(item)
    return groups
Note that this doesn't define a new function every time we call group_by, but only defines the function once and uses
that function reference as the default whenever a key_func isn't specified while calling group_by.

Also note that this lambda expression looks to be assigned to a variable (as PEP8 recommends against) but we're using
it as the default value for an argument in our function and there's no way to use def to define a function in this case
without making a second variable to then reference as the default.

I don't use lambda expressions often, but this is one of those cases where I might use one in my own code.

I hope you learned something about dictionaries and the collections module!