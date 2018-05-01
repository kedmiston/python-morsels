CHALLENGE Email (4/30/2018 @ 7:00 AM):

Hi friend! 😄

This week I want you to make a function that takes a sequence (like a list, string, or tuple) and a number n and
returns the last n elements from the given sequence, as a list.

For example:
>>> tail([1, 2, 3, 4, 5], 3)
[3, 4, 5]
>>> tail('hello', 2)
['l', 'o']
>>> tail('hello', 0)
[]
As a bonus, make your function return an empty list for negative values of n: ✔️
>>> tail('hello', -2)
[]
As a second bonus, make sure your function works with any iterable, not just sequences. ✔️ For example:
>>> squares = (n**2 for n in range(10))
>>> tail(squares, 3)
[49, 64, 81]
See if you can make your function relatively memory efficient (if you're looping over a very long iterable, don't store
the entire thing in memory).

Automated tests for this week's exercise can be found here (
http://t.dripemail2.com/c/eyJhY2NvdW50X2lkIjoiMzk0NTg2MCIsImRlbGl2ZXJ5X2lkIjoiMjU2OTUxNDgwMCIsInVybCI6Imh0dHBzOi8vZ2lzdC5naXRodWIuY29tL3RyZXlodW5uZXIvNDYyYjg3MTA0YTBmYmQ5OWQ5NjFhYjUwZmExZGEzN2I_X19zPXA1MjJnaGt5dW5wdzZrb3ZxanFzIn0
). You'll need to write your function in a module named tail.py next to the test file. To run the tests you'll run
"python test_tail.py" and check the output for "OK". You'll see that there are some "expected failures" (or "unexpected
successes" maybe). If you'd like to do the bonus, you'll want to comment out the noted lines of code in the tests file
to test them properly.


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SOLUTIONS email (5/2/2018 @ 7:00 AM):

