---
layout: post
title: Let's Write a Monad in Idris!
tags: Idris, Haskell
---

Let's Write a Monad in Idris!
=============================

To preface this, I'm not writing another monad tutorial.  There's a
(not-too-far-from-the-truth) stereotype that every Haskell programmer writes
their own terrible monad tutorial that I do not intend to perpetuate, and if
you really want to learn monads, I would recommend first reading the excellent
[You Could Have Invented
Monads](http://blog.sigfpe.com/2006/08/you-could-have-invented-monads-and.html)
and then writing a bunch of monadic code (Note: the second part of this is much
more important than the first.)  This instead assumes you have a basic idea of
what a monad is, and want to get started using them in Idris.

Whenever I personally am getting started with a new topic in Idris (or most
languages, really) I start by playing around with it in the REPL. `:t Monad` is
the expression to get the type of `Monad`, and this returns
`Prelude.Monad.Monad : (Type -> Type) -> Type`.  `:doc Monad` is slightly more
helpful, returning

{% highlight Idris %}

Type class Monad

Parameters:
    m

Methods:
    (>>=) : Monad m => m a -> (a -> m b) -> m b

        infixl 5

Instances:
    Monad id
    Monad PrimIO
    Monad IO
    Monad Maybe
    Monad (Either e)
    Monad List
    Monad (Vect n)
    Monad Stream

{% endhighlight %}

The most interesting thing that I personally found here is that Idris only
requires one operator (`>>=`) to declare an instance of `Monad`.  This is a far
cry from the Haskell I learned about monads from, which requires `>>=`, `>>`,
`return`, and `fail` to be defined on anything you want to make a monad. 
Curious, I went to the
[source
code](https://github.com/Idris-lang/Idris-dev/blob/master/libs/prelude/Prelude/Monad.idr)
to learn a bit more about how exactly one can do the work of four functions
with only one.  The actual definition of `Monad` there is quite interesting:

{% highlight Idris %}

class Applicative m => Monad (m : Type -> Type) where
(>>=) : m a -> (a -> m b) -> m b

{% endhighlight %}

In Idris, monads are a kind of applicatives!  This actually makes a lot of
sense from a category theory perspective, as monads are actually applicative,
but is still slightly surprising.  Since we're already looking around in Idris'
source code, we may as well look at [the source for
`Applicative`](https://github.com/Idris-lang/Idris-dev/blob/master/libs/prelude/Prelude/Applicative.idr)
which defines `Applicative` as:

{% highlight Idris %}

class Functor f => Applicative (f : Type -> Type) where
pure : a -> f a
(<$>) : f (a -> b) -> f a -> f b

{% endhighlight %}

Notably, `Applicative` itself is a subclass of `Functor`, which also makes
sense (an applicative is literally just a functor with application), and
looking at one more [bit of the
source](https://github.com/Idris-lang/Idris-dev/blob/master/libs/prelude/Prelude/Functor.idr)
we can finally see all the stuff that goes into being a monad in Idris:

{% highlight Idris linenos %}

class Functor (f : Type -> Type) where
map : (m : a -> b) -> f a -> f b

{% endhighlight %}

A few things immediately become a bit clearer now that we can see how to
declare a function from a type to a type a monad from start to finish.  First,
there actually are four operations that any monad has to have defined (`map`
from `Functor`, `pure` and `<$>` from `Applicative`, and `>>=` from `Monad`).
While those aren't the same four as Haskell, they're roughly
equivalent.`return` is just `pure`, `>>` is quite easy to rewrite if you should
want it (it's a specific case of `$>`, which is defined in
`Prelude.Applicative`)(and not necessary for being a monad, technically
speaking), and `fail` is actually not even part of the definition of a monad,
and is just useful for graceful failure in pattern-matchy `do` expressions.

Just to ensure that I truly understood Monads in Idris, I decided to rewrite
`Maybe`.  In Haskell, for comparison:

{% highlight Haskell linenos %}

data Maybe a = Nothing
             | Just a

instance Monad Maybe where
    (Just x) >>= k      = k x
    Nothing  >>= _      = Nothing

    (Just _) >>  k      = k
    Nothing  >>  _      = Nothing

    return              = Just
    fail _              = Nothing

{% endhighlight %}

First, `Maybe` is defined and then an instance of `Monad` is written for it.
In `Idris`, we start the same way, by defining the type (or function to a type,
really):

{% highlight idris linenos %}

data Maybe a = Nothing
             | Just a

{% endhighlight %}

So far, this is exactly the same.  However, in Idris, before we can declare our
instance of `Monad` we need an instance of `Applicative`, for which we need an
instance of `Functor`.

{% highlight idris linenos %}

instance Functor Maybe where
    map f (Just x) = Just (f x)
    map f Nothing = Nothing

{% endhighlight %}

Then we can define `Applicative`:

{% highlight idris linenos %}

instance Applicative Maybe where
    pure = Just

    (Just f) <$> (Just a) = Just (f a)
    _ <$> _ = Nothing

{% endhighlight %}

And finally `Monad`:

{% highlight idris linenos %}

instance Monad Maybe where
    Nothing >>= k = Nothing
    (Just x) >>= k = k x

{% endhighlight %}
