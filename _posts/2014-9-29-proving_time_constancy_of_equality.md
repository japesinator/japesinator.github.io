---
layout: post
title: Proving Time-Constancy of Equality
tags: crypto, idris
---

Proving Time-Constancy of Equality
==================================

Lately I've been working with [idris](http://www.idris-lang.org/) a bunch. Idris is a purely functional, dependently-typed, programming language, which, to quote one of my friends, makes it "Haskell for Haskell people." There are a lot of things that it does well, but one of the most interesting ones is that dependent types and the Curry-Howard isomorphism allow the writing of propositions as types and the proofs thereof to be written as the values for the variables of those types.  For example:

{% highlight idris linenos %}

total eqSucc : (left : Nat) -> (right : Nat) -> (p : left = right) ->
S left = S right
eqSucc left _ refl = refl

{% endhighlight %}

The above is a function taken from idris' [prelude](https://github.com/idris-lang/Idris-dev/tree/master/libs/prelude/Prelude), which is the standard library that contains most of the basic functions and proofs.  It states that if you have two natural numbers and a proof that they are equal, the successor to the first is equal to the successor to the second, and that the proof is true from the reflexive property.

This is also notable in that the same module both defines natural numbers and proves many of their properties, erasing the line between proof and program. Similarly, the module that defines the vector type also contains proofs about vectors, the module that defines lists contains proofs about lists, and so on. This is quite useful in that when you import these modules, you not only get the data types and the basic functions on them, but also the tools for reasoning about their generalized behavior.

In my humble opinion, some of the most interesting proofs in computer science are those about algorithm runtime. The big O runtime analyses in The Art of Computer Programming some of the most elegant and beautiful proofs I have ever seen, and as soon as I saw idris' proof system, I was immediately curious about how to implement these.  This also has several practical applications, as there have [been](https://www.schneier.com/paper-side-channel.html) [some](http://dl.acm.org/citation.cfm?id=706156) [problems](http://cr.yp.to/papers.html#cachetiming) with people not verifying that certain algorithms in cryptography are time-constant.

Perhaps the clearest example of how this might be a problem is equality.  Equality between two bitstrings is often implemented by comparing bits of the same index one by one until either an inequality is found (and then returning False) or there aren't any bits left (and then returning True).  This is usually pretty good, but it's notable that comparing `00000001` and `00000000` takes a long time, since it doesn't halt until all eight bits have been compared, but when `00000001` and `10000000` are compared, the comparison algorithm returns false as soon as the first bit is compared (assuming the algorithm scans from left to right.) In other words, the time the algorithm takes is a function of the "close-ness" of the two things being compared.

This is usually totally fine, but occasionally crypto algorithms need to compare things to some really sensitive data.  To use a contrived example, imagine a system that has you enter a password, compares it to a known correct password, and returns whether the two are equal.  Ideally, you would have to simply try passwords until you found one that's right, which if our password is only eight bits, takes about 89 guesses to have better-than-even odds of guessing the password.  However, if the program uses the equality algorithm described above, we can just try two bitstrings, one of which starts with 0 and one of which starts with 1, and whichever one takes longer to evaluate has the correct first bit.  We can then repeat this process with the correct first bit and bitstrings with 0 for the second bit and 1 for the second bit. By repeating this eight times, we can always get the key in with a maximum of 16 guesses, two for each bit.  Also note that as the size of our key increases, the number of guesses to get it by chance increases exponentially, but the number of guesses to get it by timing attack only increases linearly.

Slightly more elaborate versions of this attack have been used to successfully exploit AES, SSL, RSA, and most other common crytosystems on at least some level.  This is probably a bad thing.  I thus decided to write a program as proof-of-concept that compares two bytes but also proves that that comparison will always take the same time for any two pairs of bytes.  This algorithm can be found [on my github](https://github.com/japesinator/tarts).

The equality comparison itself is quite simple.  First, bits are defined as either One or Zero, and bytes are defined as a vector of eight bits.  Then, to compare the equality of two bytes, we zip them together with not xor and then fold across the resulting list with And.  Not xor returns one only if the two bits it operates on are the same, so zipping two lists with it creates a new list with one in the indices where the two lists have matching values and zero where the two lists have differing values.  And can only return one if both of its input values are one, so folding it across a list will return one only if all values in the list are one, and zero otherwise.  Thus, zipping with not xor and folding with and will return one if and only if the two lists are the same, which is the definition of equality.  At some point it's likely worth proving this formally, but I have yet to do that.

To prove time-constancy, I decided to prove that the number of primitive operations performed on bits is the same no matter what the bits are, and thus the time will be constant.  This isn't perfect due to the pattern-matching implementation of boolean logic I provided, but it's close enough for proof of concept.  To do this I wrote what's essentially a monad (I have yet to make it an instance of the monad typeclass, but that's on my todo list) that tracks the number of operations performed to yield a given bit.  The core code for this is below

{%highlight idris linenos %}

addCount : (a -> a -> a) -> (a, Nat) -> (a, Nat) -> (a, Nat)
addCount f (a, n) (b, m) = (f a b, n + m + 1)

{& endhighlight %}

This essentially turns a regular function from two elements of the same type to another and turns it into a function that does the same thing but counts operations. The natural number that is the second element of each tuple represents the number of operations performed to yield that value.  It takes a function and two tuples of a value and an operation count and returns a tuple of the function applied to the first elements of each tuple and one plus the counts added together, the one coming from the operation that just took place.

To write a version of equality that also provides an operation count, we can write the same equality function as before, but using (addCount bibNXOR) instead of bitNXOR and (addcount bitAnd) instead of bitAnd and (initializeCount a) and (initializeCount b) for the two bytes being compared, where initializeCount just maps `(\x => (x,0))` on a vector, saying that (relative to our function) it took no operations to produce the initial values, as they were just given to us.

This function will return a tuple of a bit representing whether the two bytes are equal and a natural number representing the number of operations that took place to compare them.  It's arguable that sing a single natural number is an oversimplification, as bitNXOR and bitAnd could conceivably take different times, and it should instead return a bit and a table of functions used and the number of times they were used, and I may change the behavior of addCount at some future point to provide this functionality, but at the moment it doesn't seem necessary to me.

Once we have this function, we can write a number of lemmas to eventually give us a proof with type of `(a,b,c,d:Byte) -> snd (countingByteEq a b) = snd (countingByteEq c d)`, or for any two pairs of bytes, the time to compare them is equal.  I'm not going to walk through the proof of that in this blog post, as at the moment it's hideous, but the code does include a working, total proof of that proposition.

Clearly, there's still work to be done even on the definition of equality, but the fundamental principles are sound, and hopefully similar tactics can be used on a larger scale to prove time-constancy for things like RSA and AES where that's actually really important.
