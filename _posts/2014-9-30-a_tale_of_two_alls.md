---
layout: post
title: A Tale of Two Alls
tags: haskell, idris
---

One of my favorite parts of programming in strongly typed languages is the
 ability to use the type signature of a function to figure out what it does.
 For instance, `:t map` returns either `map :: (a -> b) -> [a] -> [b]` in
 Haskell, or `Prelude.Functor.map : Functor f => (a -> b) -> f a -> f b` in
 idris, both of which make it relatively clear how map works (provided, in the
 case of idris, you're at least moderately familiar with what a functor is.)
 Both [haskellmode-vim](http://projects.haskell.org/haskellmode-vim/) and
 [idris-vim](https://github.com/idris-hackers/idris-vim) make it very easy to
 check types from inside the editor.

Recently when I was working on a [project of
mine](http://writes.co.de/2014/09/29/proving_time_constancy_of_equality/) I
 came across two different `all`s in idris.  One is quite intuitive and has
 type of `Prelude.Foldable.all : Foldable t => (a -> Bool) -> t a -> Bool`,
 which is quite similar to Haskell's all, which has a type of `all :: (a ->
 Bool) -> [a] -> Bool`, but another one has a type of
 `Data.Vect.Quantifiers.all : ((x : a) -> Dec (P x)) -> (xs : Vect n a) -> Dec
 (All P xs)`.  Yet more confusing is the fact that in idris, `:t All` (with a
 capital 'A') returns `Data.Vect.Quantifiers.All : (a -> Type) -> Vect n a ->
 Type`.

After finding this, I was fairly confused, as my intuitive understanding of
 `all` was that it should take a list-like group of items and return a Boolean
 representing whether they all have some certain property.  This absolutely
 makes sense in with the `all` defined in Prelude.Foldable and Haskell's all,
 but the difference between `Type`, `Dec (All P xs)`, and a `Bool` seemed hard
 to reconcile to me.

Doing some more analysis, I decided to start by picking apart `Dec (All P xs)`.
  `:t Dec` shows `Prelude.Basics.Dec : Type -> Type` shows that apparently
 `Dec` is a function from a Type to a Type, which isn't necessarily helpful by
 itself, but `:doc Dec` actually is, returning:

{% highlight lidr linenos %}

Data type
> Dec : Type -> Type
    Decidability. A decidable property either holds or is a
    contradiction.

Constructors:
> Yes : (prf : A) -> Dec A
        The case where the property holds
        Arguments:
> prf : A  -- the proof

> No : (contra : A -> _|_) -> Dec A
        The case where the property holding would be a contradiction
        Arguments:
> contra : A -> _|_  -- a demonstration that A would be a contradiction

{% endhighlight %}

Now we're getting somewhere.  `Dec` takes as a parameter a type, but since this
 is idris, types and propositions are the same thing, and `Dec` just states
 that either the proposition holds or doesn't.  Since the proposition is either
 true (it can be shown to hold in this case) or false (it implies a
 contradiction if true), `Dec` will return either `Yes` or `No`.  This is quite
 similar to a standard Bool, but arguably more useful in that it can speak
 about general cases and also state both whether the given statement is true or
 false, but also in some sense why.  For instance idris defines a class `DecEq`
 as:

{% highlight idris linenos %}

class DecEq t where
  total decEq : (x1 : t) -> (x2 : t) -> Dec (x1 = x2)

{% endhighlight %}

In some sense this serves the same purpose as `(==)` (which has a type of
 `Prelude.Classes.(==) : Eq a => a -> a -> Bool` in idris), but because `(==)`
 returns a `Bool`, it can only tell us whether two things are equal, whereas
 `DecEq` can tell us *why* two things are equal.

In fact, `DecEq` is actually defined on Bools in Decidable.Equality as:

{% highlight idris linenos %}

total trueNotFalse : True = False -> _|_
trueNotFalse Refl impossible

instance DecEq Bool where
  decEq True True = Yes Refl
  decEq False False = Yes Refl
  decEq True False = No trueNotFalse
  decEq False True = No (negEqSym trueNotFalse)

{% endhighlight %}

Note the difference between this and the definition of `Eq` for Bools, which is
 simply:

{% highlight idris linenos %}

instance Eq Bool where
  True == True = True
  True == False = False
  False == True = False
  False == False = True

{% endhighlight %}

The crucial difference between the two is that while `True == False = False`
 says that True and False *are not* equal, `decEq True False = No trueNotFalse`
 says True and False *cannot* be equal.  Back to the case of lists, if we look
 at the definitions of `All` and `all`, we find:

{% highlight idris linenos %}


data All : (P : a -> Type) -> Vect n a -> Type where
  Nil : {P : a -> Type} -> All P Nil
  (::) : {P : a -> Type} -> {xs : Vect n a} -> P x -> All P xs -> All P (x :: xs)

notAllHere : {P : a -> Type} -> {xs : Vect n a} -> Not (P x) -> All P (x :: xs) -> _|_
notAllHere _ Nil impossible
notAllHere np (p :: _) = np p

notAllThere : {P : a -> Type} -> {xs : Vect n a} -> Not (All P xs) -> All P (x :: xs) -> _|_
notAllThere _ Nil impossible
notAllThere np (_ :: ps) = np ps

all : {P : a -> Type} -> ((x : a) -> Dec (P x)) -> (xs : Vect n a) -> Dec (All P xs)
all _ Nil = Yes Nil
all d (x::xs) with (d x)
  | No prf = No (notAllHere prf)
  | Yes prf =
  case all d xs of
    Yes prf' => Yes (prf :: prf')
    No prf' => No (notAllThere prf')

{% endhighlight %}

This is now much clearer (at least to me.) `all` takes a proposition, a way to
 verify that proposition, and a list of things, and returns why the proposition
 does or doesn't hold for everything in the list.  It's the difference between
 a multiple-choice and short-answer question, in that while they can state the
 same thing, the answer to one is much more helpful in reasoning about future
 questions than the other.  While a Bool is fine for simple questions, there's
 a lot to be gained from using `Dec` when it's practical.

One of the more interesting applications for this is [this quasigroup
 completion solver](https://github.com/Ralith/quasigroup-completion), later
 adapted to a benchmark of idris itself.  A quasigroup is just a set for and an
 operation such that each element of the set appears exactly once in each row
 and column of the [Cayley Table](https://en.wikipedia.org/wiki/Cayley_table)
 generated from the elements in the set and the operation.  This property also
 means that the quasigroup completion problem is often referred to as the Latin
 Square problem, since said Cayley table is also a [Latin
 Square](https://en.wikipedia.org/wiki/Latin_square) by definition.  It is a
 common combinatorial problem to take a given partially filled Latin Square and
 complete it in a way consistent with the definition.  This process is actually
 quite similar to sudoku, where a grid of spaces that are either empty or
 containing a value are given, and the goal is to fill in the empty squares so
 that the same number does not appear twice in any row or column.  Of course,
 quasigroup completion is a more general and abstract problem, but the
 fundamental process is the same.

The solver shown uses the Data.Vect.Quantifier version of `all` to reason about
 things like whether given rows and columns are "safe" to place an element in,
 perhaps most notably in the definition of legalVal, which is used to evaluate
 whether a given cell assignment is "legal" with regard to the definition of
 quasigroup.

{% highlight idris linenos %}

LegalVal : Board n -> (Fin n, Fin n) -> Fin n -> Type
LegalVal b (x, y) val = (Empty (getCell b (x, y)), All (LegalNeighbors (Just val)) (getCol x b), All (LegalNeighbors (Just val)) (getRow y b))

legalVal : (b : Board n) -> (coord : (Fin n, Fin n)) -> (val : Fin n) -> Dec (LegalVal b coord val)
legalVal b (x, y) v =
case rowSafe b y v of
  No prf => No (\(_, _, rf) => prf rf)
  Yes prf =>
  case colSafe b x v of
    No prf' => No (\(_, cf, _) => prf' cf)
    Yes prf' =>
    case empty (getCell b (x, y)) of
      No prf'' => No (\(ef, _, _) => prf'' ef)
      Yes prf'' => Yes (prf'', prf', prf)

{% endhighlight %}

In the definition of `legalVal`, `prf`, `prf'`, and `prf''` all refer to proofs
 of the "safety" of the given value at the given coordinates with respect to
 its row, column, and occupancy status.  This allows `LegalVal` to be a type
 that only represents legal values, and thus guarantee that functions using
 that type can only ever give legal values, because otherwise the program would
 type check.  Not that this is totally impossible in a language like Haskell
 with no dependant types, as you cannot have a function like `LegalVal` to a
 type, and hence the type cannot *depend* on parameters that aren't other
 types, and hence can't carry guarantees about them like in a language like
 idris.

This makes our initial question of the behavior of the two `all`s much simpler.
  One returns either True or False, and the other returns either `Yes` or `No`,
 but the second provides a reason for the returned value that can be used to
 reason about the behavior of the program in general and guarantee certain
 behaviors.
