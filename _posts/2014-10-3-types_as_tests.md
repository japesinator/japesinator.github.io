---
layout: post
title: Types as Tests
tags: idris
---

Recently, a friend of mine and I were going over the ages old tests vs. types
debate.  Specifically, if a programmer wants to have some degree of certainty
that their program will function as intended, does it work better to use unit
testing or static typing?  However, the more I looked at the argument we were
having, the less it made sense to me as a strict dichotomy.  Obviously, best
practices for ensuring Python code works as intended are different from
ensuring Haskell code works as intended, and you'll catch some bugs with Python
best practices that you won't catch with Haskell best practices and vice versa,
but in a sufficiently advanced language, assertions about the code are the same
as function types and vice versa.

For instance, consider the (rather trivial) example of a function that reverses
lists.  In python:

{% highlight python linenos %}

def reverse(list):
  return list[::-1]

{% endhighlight %}

In Idris

{% highlight idris linenos %}

reverse : List a -> List a
reverse = reverse' []
  where
    reverse' : List a -> List a -> List a
    reverse' acc [] = acc
    reverse' acc (x::xs) = reverse' (x::acc) xs

{% endhighlight %}

Suppose we want to show that an example list reversed twice is itself.  In
Python, we might try an assertion.

{% highlight python linenos %}

import unittest

testVar = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

assert testVar == reverse(reverse(testVar))

{% endhighlight %}

If we were doing something more complex, we could use
[unittest](https://docs.python.org/2/library/unittest.html) or
[nose](https://nose.readthedocs.org/en/latest/) or any number of other options,
but for the sake of simplicity, we'll stick with with `assert` for now.

Now we wish to make the same assertion about our idris function.

{% highlight idris linenos %}

testvar : List Int
testVar = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

reverseTwice : testvar = reverse $ reverse testvar
reverseTwice = refl

{% endhighlight %}

Note that the assertion happens in the *type* of `reverseTwice`, not the
definition like in the Python example, but the two serve the exact same
purpose.  This is because an assertion is a proposition about a program, and a
[proposition is a
type](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence), so
any assertion can be written as the type of a variable, given sufficient tools
to do so.  The definition of the variable with that type then, is the proof of
the proposition the assertion makes in the first place.  Notably, this allows
us to use relatively advanced tactics to make much more powerful assertions
than in languages without this functionality.  For instance, [Jesse Hallett's
category theory
proofs](http://sitr.us/2014/05/05/category-theory-proofs-in-idris.html).

One concrete example would be [this rewrite of
reverse](https://github.com/defanor/idris-stuff/blob/master/Vect.idr) by
defanor, that generalizes our earlier assertion to not just `testVar`, but any
Vect.  Obviously, this is more work than a simple assert statement, but an
assert statement is also more work than not testing at all, and for code that
*really needs to work*, a proof that it always will work is often desirable.

Obviously, there are types of testing that can't be written as a type.  It's
unlikely that [selenium](http://www.seleniumhq.org/) can ever have a type-level
equivalent, and something like
[quickcheck](https://github.com/nick8325/quickcheck) that uses stochastic
methods to give a fuzzier guarantee is a fundamentally different tactic, but
the vast majority of unit testing is equivalent to simple types.  Not only
that, but dependent types and totality checking allow us to check every
possible value for a parameter against its expected output all at once using
mathematical reasoning.  And while there is certainly a place for assertions,
dependent types make them seem as primitive as the goto when it comes to
ensuring program correctness.
