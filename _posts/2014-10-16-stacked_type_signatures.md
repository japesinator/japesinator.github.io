---
layout: post
title: Stacked Type Signatures
tags: practices
---

Stacked Type Signatures
=======================

In general, I greatly prefer programming in functional, strongly-typed languages.  I'm currently working on [a project](https://github.com/Lopi/HackMan) in [elm](http://elm-lang.org/), [a project](https://github.com/japesinator/tarts) in [idris](http://www.idris-lang.org/) and a few miscellaneous odds and ends in Haskell.  In all of these languages, types are relatively important, and especially so in the style of programming I tend to prefer.

In said functional, strongly-typed languages, type signatures are either best practices or required for functions.  The typical format of a type signature is something along the lines of:

{% highlight haskell %}

map :: (a -> b) -> [a] -> [b]

{% endhighlight %}

Which is read "`map` is a function from a function from something of type `a` to something of type `b` to a function from a list of type `a` to a list of type `b`", or, to put it less completely unitelligibly, `map` takes a function and a list of things that function can be applied to and returns a list of things of the same type as that function returns.  However, lately I've switched from the style above in writing my own type signatures to something more like:

{% highlight haskell %}

map :: (a -> b) ->
       [a] ->
       [b]

{% endhighlight %}

At first this seems totally unnecessary and waste-of-space-y, but I actually have grown to greatly prefer it over the standard style.  While for `map` it doesn't make a ton of difference, in more complex type signatures, I've found it makes both reading and documenting code much more intuitive, at least for me.  Consider the function `zipWith3`:

{% highlight haskell %}

zipWith3 :: (a -> b -> c -> d) -> [a] -> [b] -> [c] -> [d]

{% endhighlight %}

Rewritten in stacked style:

{% highlight haskell %}

zipWith3 :: (a -> b -> c -> d) ->
            [a] ->
            [b] ->
            [c] ->
            [d]

{% endhighlight %}

Immediately, it seems a bit clearer, but now consider the well-commented version.

{% highlight haskell %}

zipWith3 :: (a -> b -> c -> d) -> -- The function to zip with
            [a] ->                -- The first list to zip together
            [b] ->                -- The second list to zip together
            [c] ->                -- The third list to zip together
            [d]                   -- The list of results when the function is
                                  --   applied element-by-element to the given
                                  --   lists.

{% endhighlight %}

Obviously, that's an unnecessary amount of information, but it gives a far clearer picture of what the function really does.  Note that the description of each argument is able to be immediately next to the kind itself, which is nice for casual reading of the code, and the difference between the `->`'s in the function used to zip with and the `->`'s in the actual type signature.  In fact, the type signature serves as pretty much all the documentation the function needs.  Of course, the `|||` documentation style also works well, but between commenting next to a stacked type signature and commenting above/below a function, I personally find the former much more legible.
