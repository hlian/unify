## OK so

This is a simple type unifier. Maybe it's best to show you.

Let's unify these five types:

* `int -> int -> 'a` and `'a -> int -> int` to get `int -> int -> int`.
* `'a -> int -> 'b` and `int -> 'c` to get the bottom type (failure).
* `a -> b -> c` and `b -> c -> a` to get `c -> c -> c`.
* `a -> a` and `(b -> b) -> (b -> b)` to get `(b -> b) -> (b -> b)`.
* `a -> [b]` and `b -> [a]` to get a cycle error.

That looks like this:

```haskell
    spew :: Type -> Type -> IO ()
    spew left right =
      case unify [(left, right)] empty of
       Left e ->
         print e
       Right links ->
         print (generalize links left)

    main :: IO ()
    main = do
      spew ex1l ex1r
      spew ex2l ex2r
      spew ex3l ex3r
      spew ex4l ex4r
      spew ex5l ex5r
      where
        ex1l = TFn [TPrim PrimInt, TPrim PrimInt, TVar A]
        ex1r = TFn [TVar A, TPrim PrimInt, TPrim PrimInt]
        ex2l = TFn [TVar A, TPrim PrimInt, TVar B]
        ex2r = TFn [TPrim PrimInt, TVar C]
        ex3l = TFn [TVar A, TVar B, TVar C]
        ex3r = TFn [TVar B, TVar C, TVar C]
        ex4l = TFn [TVar A, TVar A]
        ex4r = TFn [TFn [TVar B, TVar B], TFn [TVar B, TVar B]]
        ex5l = TFn [TVar A, TList (TVar B)]
        ex5r = TFn [TVar B, TList (TVar A)]

    λ> main
    TFn [TPrim PrimInt,TPrim PrimInt,TPrim PrimInt]
    LinksErrorBottom (TFn [TVar A,TPrim PrimInt,TVar B]) (TFn [TPrim PrimInt,TVar C])
    TFn [TVar C,TVar C,TVar C]
    TFn [TFn [TVar B,TVar B],TFn [TVar B,TVar B]]
    LinksErrorCycle B (TVar A)
```
