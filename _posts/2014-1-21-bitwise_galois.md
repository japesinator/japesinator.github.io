---
layout: post
title: Bitwise Galois Arithmetic
tags: crypto, python, math
---

Bitwise Galois Arithmetic
=========================

Recently, I began working on implementing AES in python.  My code is largely based off of [SlowAES](https://code.google.com/p/slowaes/), but using much more in-depth comments, more descriptive variable names, and snake case.  I understood most of the code, but there was a function for performing "Galois Multiplication" for column mixing that I was unable to deduce the function of entirely.  It is reproduced below, sans comments.


{% highlight python linenos %}
def galois_multiplication(a, b):
  p = 0
  for i in range(8):
    if b & 1:
      p ^= a
    hi_bit_set = a & 0x80
    a <<= 1
    a &= 0xFF
    if hi_bit_set:
      a ^= 0x1b
    b >>= 1
  return p
{% endhighlight %}


A quick wikipedia search for "Galois Multiplication" returned a [page](https://en.wikipedia.org/wiki/Finite_field_arithmetic) that seemed to be about the same thing, but not in any way I could relate back to the actual code.  There was no simple English version I could find, so I looked into the [link](http://www.samiam.org/galois.html) at the bottom of the page, which seemed to be about the same thing I was looking for.

Here I found some C code that the code I was looking at was obviously based off of, and a relatively simple explanation of what it did.  Based on that and using my rudimentary knowledge of bitwise operators in python, I was able to line-by-line annotate the code with the purpose of each line.

To understand line 4, "if b & 1", we must first understand three things:

1:  When given a number as a conditional, python evaluates true if that number does not equal 0 and false if it does.

2:  & represents bitwise and in python

3:  In python, binary numbers are treated like integers, so 0b1 == 0b01 == 0b001, and therefore n & 0b1 == n & 0b001.

Hence, the if statement first essentially performs a bitwise and on the binary representation of b and a binary number of equal length consisting of zeroes in all but the least significant place and a one in that place and then checks if the result is not equal to zero.  All this does is executes the code inside the 'if' if the least significant bit of b is one.  Simplifying further, we can see that the conditional just checks if b is odd, albeit in a faster and less easily comprehensible way.

The line inside the if statement is much easier to understand.  In python ^= is similar to += in that a ^= b is the same as a = a ^ b.  ^ is the bitwise xor function, so all that this line does is xors p with a.

The next line, "hi_bit_set = a & 0x80" is again, an application of pythons bitwise and to perform an operation quickly, but in a way that is hard to deduce from the code.  Converting 0x80 to decimal and then to binary we find that 0x80 == 128 == 0b10000000.  Bitwise and of any number with zero is zero, so our output will return 0b10000000 if a has a 1 in the 128's place and 0 otherwise.  This is important at line 11, which checks whether this bit was set earlier.

Now that we have checked whether a's highest bit was set, we can modify it with a <<=1, which simply shifts each bit to the right and adds a zero to the end, or in decimal, multiplies a by 2.

Line 8 took me a while to understand the purpose of. 0xFF == 0b11111111, and if n is any bit, n & 1 == 1, so at first is looked to me like this line did not even modify a.  However, this is the same trick as used in line 4.  Applying a bitwise and to a number and a string of ones n bits long truncates that number to its least significant n bits.  Line 8 therefore makes sure that a is only 8 bits long after a left bitshift that could make it longer than that.

In the next line, we use the high-bit-set test we devised before to see if a was >= 128 prior to our bitshift and truncation.  If it was, we xor a with 0x1b. 0x1b == 27 == 0b00011011, which has no easily apparent special properties, and only makes sense in the context of the actual math the function is based upon.

After the second conditional has been evaluated, we shift b to the right one bit, essentially dropping the least significant bit and then dividing it by two.  We repeat this 8 times, and we have our function.

Based on my analysis of the function, I was able to rewrite it and comment it to what I found a much easier to understand, if still somewhat cryptic functions, seen below.

{% highlight python linenos %}
def galois_multiplication(first_number, second_number):
  product = 0
  for i in range(8):
  # Repeat the indented block below eight times
    if (second_number % 2) != 0:
    # If b is odd,
      product = product ^ f
      # xor the product with first_number
    if first_number >= 128:
      high_bit_set = T
      # Remember if a was greater than or equal to 128 here
    first_number = 2 * f
    # double the first number
    if first_number >= 256:
    # This conditional essentially sets the first number equal to itself modulo 256
      first_number = first_number - 256
    if high-bit-set == True:
    # If the first number was >= 128, xor it with 27
      first_number = first_number ^ 0b00011011
    # The three lines below are the same as b >>=1
    if second_number % 2 == 1:
      second_number = second_number - 1
    second_number = second_number / 2
  return product
{% endhighlight %}


While this is certainly easier to understand from an arithmetic point of view, the overall purpose of the code is still a mystery.  To understand it, we must first understand why it is used.

Galois arithmetic is a family of functions analogous to standard arithmetic that only takes and returns numbers from a finite field of values.  AES uses Galois arithmetic heavily for reasons that are too complex to explain well in a post of this scale, but which can be poorly summarized by pointing out that they can be counted on to only return certain numbers and aren't too easy to reverse.

We might expect that since Galois arithmetic is analogous to our own arithmetic, Galois multiplication is simply repeated Galois addition.  However, since Galois addition of two numbers is actually equivalent to simply bitwise xoring them, and (a ^ b) ^ b == a, Galois-adding two numbers together any number of times can only result in two values.  [This website](http://www.pclviewer.com/rs2/galois.html), offers a fairly good explanation of exactly what Galois field arithmetic is and how to do it, and it provides that multiplication is the "Antilog of the modulus 256 sum of their logs."  This, while it certainly does not make our function clear, offers at least a better explanation than we have seen so far.

Looking at our definition, we already know how to perform modulo 256 addition, but logs and antilogs are both more complex in a Galois field than they are in basic arithmetic.  In standard arithmetic, if log base a of b equals c, c a to the c power equals b.  In other words, logs are the inverse of exponentiation.  If we try to directly translate this to a Galois field however, we encounter a problem in that exponentiation is repeated multiplication, and multiplication is the function we're trying to create in the first place.  Looking for less circular definitions, we find on the aforementioned Wikipedia page, that multiplication in a finite field is just multiplication modulo the polynomial that defines the field.  This makes much more sense if we are able to look at numbers written in a base n system as polynomials.  For instance, if I write the number 1365, I have written the equivalent of the polynomial 1x^3 + 3x^2 + 6x^2 + 5x^0 where x is the base of the number system I am writing in.

In AES's finite field, numbers are expressed as binary, so that base is two.  The irreducible polynomial that defines the field is actually 283, or 0b100011011.  Notably, 0b100011011 % 256 (or 0b10000000) = 0b11011 (or 27) which at least partially explains where the mysterious constant in line 10 comes from.  Thus, when we want to multiply in a Galois field, we can simply multiply our two numbers like usual and then take the result modulo 283.  This certainly makes it clearer exactly what we are doing, and even allows us to rewrite our code as:

{% highlight python linenos %}
def galois_multiplication(a, b):
  return ((a * b) % 283)
{% endhighlight %}


This is definitely an improvement from our initial algorithm in terms of legibility, but it is still hard to understand how they do the same thing.  Notably, in our original algorithm, p is only modified by xoring it with a.  Since in a Galois field, multiplication is equivalent to xor, we can begin to see how multiplication is still repeated addition in a Galois field, albeit slightly more complex.  We can also notice how the for loop runs eight times, once for each bit in the numbers.  Knowing this, we can look at the code as a set of simpler instructions:

1:  If b is odd, Galois-add a to the product

2:  Multiply a by two.

3:  If a is more than 256, Galois-add 283

4:  Divide b by two, discarding the remainder

5:  Repeat until b is 0

This is simpler, but still far removed from our normal concept of multiplication until we look at [Peasant Multiplication](https://en.wikipedia.org/wiki/Multiplication_algorithm#Peasant_or_binary_multiplication).  Expressed as a series of steps, peasant multiplication is:

1:  If b is odd, add a to the product

2:  Multiply a by two

3:  Divide b by two, discarding the remainder

4:  Do this until b is 0.

Using a basic knowledge of bitwise arithmetic, it is clear to see that these two algorithms are the same except for step 3 of the first one.  This is added because in Galois arithmetic, the only numbers "Allowed," are those in a certain group, and step three is just the procedure for taking a number outside the group and converting it into one inside the group.

It's interesting that a modern, CPU-optimized routine from an encryption algorithm is the same thing Egyptian peasants used millennia ago to calculate payment for sheep, but good math is the same everywhere.

NOTES:

*  In the actual AES algorithm, there are a few extra steps in the Galois addition function.  These serve only to prevent [Timing Attacks](https://en.wikipedia.org/wiki/Timing_attack) and are irrelevant to the math discussed here.
*  The reason 283 is the Galois group's defining constant is actually fairly complicated, but it stems from the fact that it is irreducible, which is similar to primality in standard arithmetic.
*  Group theory is hard.
