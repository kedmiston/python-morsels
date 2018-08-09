CHALLENGE Email (5/28/2018):

Hiya!

See if you can figure out at least one of the bonuses for this week's exercise if you have time.

I want you to write a function that accepts an iterable and returns a new iterable with all items from the original
iterable except for duplicates.

Here's an example:
>>> uniques_only([1, 2, 2, 1, 1, 3, 2, 1])
[1, 2, 3]
>>> nums = [1, -3, 2, 3, -1]
>>> squares = (n**2 for n in nums)
>>> uniques_only(squares)
[1, 9, 4]

As a bonus, return an iterator (for example a generator) from your uniques_only function instead of a list. This should
allow your uniques_only function to accept infinitely long iterables (or other lazy iterables). ✔️

Here's another bonus to do after you've made your uniques_only function return a lazy iterable: allow your uniques_only
function to work with unhashable objects. ✔️

For example when two lists with equal items are provided, they should be seen as duplicates:
>>> list(uniques_only([['a', 'b'], ['a', 'c'], ['a', 'b']]))
[['a', 'b'], ['a', 'c']]
For an extra bonus, make sure your function works efficiently with hashable items while still accepting unhashable
items. ✔️ This one is a little harder to test. There's an automated test (included below) that attempts to performance
test your function.

Automated tests for this week's exercise can be found here. You'll need to write your function in a module named
uniques.py next to the test file. To run the tests you'll run "python test_uniques.py" and check the output for "OK".
You'll see that there are some "expected failures" (or "unexpected successes" maybe). If you'd like to do the bonus,
you'll want to comment out the noted lines of code in the tests file to test them properly.


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SOLUTIONS email (5/30/2018):

Hiya! 😄

If you haven't attempted to solve uniques_only yet, close this email and go do that now before reading on. If you have
attempted solving uniques_only, read on...

When you read that you're only supposed to return unique values, you might have thought to convert the incoming items
to a set and then back to a list, to keep only the unique ones around:
def uniques_only(sequence):
    """Return sequence in the same order but with duplicates removed."""
    return list(set(sequence))
But this doesn't work consistently because sets are unordered. If we pass in [1, 1, 3, 3, 2], we might get out [1, 2, 3]
which isn't in the right order!

So we definitely can't rely just on sets.

You might have instead thought to use indexes to check items that came before the current item while looping:
def uniques_only(sequence):
    """Return sequence in the same order but with duplicates removed."""
    items = []
    for i, item in enumerate(sequence):
        if item not in sequence[:i]:
            items.append(item)
    return items
Here we're slicing the sequence that comes into our function to get all items before our current one (the i-th one).
This allows us to check whether an item appeared before our current item.

This will only work for sequences though.

To get our function to work with all types of iterables, we'll need to keep track of all the items we've seen so far.
We're actually already doing that with our items list. So we could simply check whether each new item is already
contained in our list:
def uniques_only(iterable):
    """Return iterable in the same order but with duplicates removed."""
    items = []
    for item in iterable:
        if item not in items:
            items.append(item)
    return items
This works but it's going to be very slow for large lists of items because checking for containment (X not in Y) in a
list requires looping through the whole list.

We can make this faster by using a set:
def uniques_only(iterable):
    """Return iterable in the same order but with duplicates removed."""
    seen = set()
    items = []
    for item in iterable:
        if item not in seen:
            items.append(item)
            seen.add(item)
    return items
Sets rely on hashes for lookups so containment checks won't slow down as our hash grows in size. Notice we're building
up both a list and a set but we're checking only the set for containment and returning only the list. We still need to
make a list in addition to our set because sets are unordered by nature and we want the order of first appearance of
each item to be maintained.

If you're using Python 3.6+, you could also use dict.fromkeys to create a dictionary (which has unique keys that
maintain insertion ordered as of Python 3.6) and then grab just the keys from the dictionary:
def uniques_only(iterable):
    """Return iterable in the same order but with duplicates removed."""
    return dict.fromkeys(iterable).keys()
This is a somewhat clever and hacky solution.  It's short, but it's not very clear.  It also only works on Python 3.6+.
I prefer the set solution from a readability standpoint, but the dict.fromkeys solution is likely faster.

Bonus #1
Let's try tackling the first bonus.
def uniques_only(iterable):
    seen = set()
    for item in iterable:
        if item not in seen:
            yield item
            seen.add(item)
We're using a generator function here.  Generator functions are unlike regular functions.  They return a generator
object which will return items every time a yield statement is hit in our generator function.

We can't turn this generator function into a generator expression because we need to add items to our set as we iterate,
which isn't possible with a generator expression.

Bonus #2
Let's talk about the second bonus now.

If we want our function to work with non-hashable types, we'll need to use something besides a set or a dictionary to
store seen values.  We could use a list, like we did before:
def uniques_only(iterable):
    seen = []
    for item in iterable:
        if item not in seen:
            yield item
            seen.append(item)
This works, but it will be slower.  Answering the question "item not in seen" when using a list requires iterating all
the way through the list looking for a match.  A set can answer the same question without iterating at all.

Bonus #3
If we want to tackle the third bonus problem and make our code continue to work efficiently for hashable types, we
could check each item to see if it's hashable:
from collections.abc import Hashable

def uniques_only(iterable):
    seen_hashable = set()
    seen_unhashable = []
    for item in iterable:
        if isinstance(item, Hashable):
            if item not in seen_hashable:
                yield item
                seen_hashable.add(item)
        else:
            if item not in seen_unhashable:
                yield item
                seen_unhashable.append(item)
The collections.abc.Hashable type uses duck typing when doing an isinstance check, so asking isinstance(item, Hashable)
will always give us the correct answer for each object in Python.

Another way we could check for hashability is to simply try to add each item to the set and if it fails, add it to the
list instead:
def uniques_only(iterable):
    seen_hashable = set()
    seen_unhashable = []
    for item in iterable:
        try:
            if item not in seen_hashable:
                yield item
                seen_hashable.add(item)
        except TypeError:
            if item not in seen_unhashable:
                yield item
                seen_unhashable.append(item)
This is practicing "it's easier to practice forgiveness than permission", which is a common programming practice in
the Python world.  The exception handling involved does make the non-hashable case a bit slower though, so that if
statement might be better overall depending on what we're optimizing for.

I hope a couple of this week's solutions made you think about this problem differently!
