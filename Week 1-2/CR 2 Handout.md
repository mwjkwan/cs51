# [Code Review 2](https://docs.google.com/presentation/d/1aOlll20latEGnH5XK7dy_M9IeQARdhGJH3_iy2zdXXg/edit?usp=sharing)
Melissa Kwan, mkwan@college.harvard.edu

## Polymorphic types
Write functions / expressions that will evaluate to the given type.

```ocaml
('a * 'b) list -> ('a * 'b option) list
```

```ocaml
('a -> 'b) -> ('c -> 'a) -> 'c -> 'b
```

## Currying

**Currying**

`curry` is the function that takes in an uncurried function and returns a curried function.

`uncurried` is the function that takes in an input of type  `` (`a * `b) `` and returns an output of type `` `c ``.

`(curry uncurried)` is the function that takes in an input of the form `` `a -> `b `` and returns an output of type `` `c ``.

**Uncurrying**

`uncurry` is the function that takes in an curried function and returns an uncurried function.

`curried` is the function that takes in an input of the form `` `a -> `b `` and returns an output `` `c ``.

`(uncurry curried)` is the function that takes in an input of type `` (`a * `b) `` and returns an output of type `` `c ``.

```ocaml
(* Curry: first pass *)
let curry : ('a * 'b -> 'c) -> 'a -> 'b -> 'c =
	fun uncurried ->
		fun x ->
			fun y -> uncurried (x, y) ;;
    
(* Uncurry: first pass *)
let uncurry : ('a -> 'b -> 'c) -> 'a * 'b -> 'c  =
	fun curried ->
		fun (x, y) -> curried x y ;;

(* Curry: Final implementation *)
let curry uncurried x y = uncurried (x, y) ;;
    
(* Uncurry: Final implementation *)
let uncurry curried (x, y) = curried x y ;;
```

## Map, Filter, Fold, and Other List Module Functions

```ocaml
(* Map: Use when you want to apply a function or transformation to
every element of a list *)
let rec map (f : 'a -> 'b) (lst : 'a list) : 'b list =
	match lst with
	| [] -> []
	| hd :: tl -> (f hd) :: (map f tl) ;;

(* Filter: When you want to limit a list to only include
specific elements. *)
let rec filter f lst =
	match lst with
	| [] -> []
	| hd :: tl -> if f hd
                  then hd :: filter f tl
                  else filter f tl

(* Fold_right: Iterate through a list to get a single value. *)
let rec fold_right (f : 'a -> 'b -> 'b) (lst : 'a list) (acc : 'b) : 'b =
	match lst with
	| [] -> acc
    | hd :: tl -> f hd (fold_right f tl acc) ;;
  
(* Fold_left: Iterate through a list to get a single value. *)
let rec fold_left (f : 'b -> 'a -> 'b) (acc : 'b) (lst : 'a list) : 'b =
    match lst with
    | [] -> acc
    | hd :: tl -> fold_left f (f acc hd) tl ;;
```

## The Relative Power of Higher-Order functions

1. Can you implement `filter` using `map`? What about vice versa?
2.  Implement `square_sum`, which squares all the elements in the list
    and returns the sum.
3.  Implement `map` using `fold`.
4.  Implement `partition` using `fold`.


## Polymorphic types in higher-order functions

Identify the type of these functions.

```ocaml
let mystery_1 f = List.fold_left f 0 ;;
```
    
```ocaml
let mystery_2 f acc = List.fold_left (f "cs51") acc ;;
```

## Options and Error Conditions

1.  What’s the type of the following function?

```ocaml
let calc_option f x y =
	match x, y with
	|None, None -> None
	|None, _ -> y 
	|_, None -> x 
	|_, _ -> f x y ;;
```

2.  Implement the function `all_some : ’a option list -> bool`, which
    returns `true` if all elements in a list of options are of type
    `Some x` and `false` otherwise.

3.  Implement the function
    `assoc_opt : ’a -> (’a * ’b) list -> ’b option`, which returns the
    value associated with key `a` in the list of pairs `l`. That is,
    `assoc_opt a [ ...; (a,b); ...] = b` if `(a,b)` is the leftmost
    binding of `a` in list `l`. Returns `None` if there is no value
    associated with a in the list `l`.


