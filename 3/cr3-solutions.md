# Code Review 3
Melissa Kwan, mkwan@college.harvard.edu
Presentation link forthcoming!

1. **Problem Set 1 grades are out!** You can resubmit your code for a design / style regrade, provided your correctness score is at least as good as it was before. Even if your new submission passes more unit tests, your correctness score will remain the same.

2. **Lab attendance.** Grading for lab attendance is extremely punitive (a third of a letter grade for every two unexcused absences.)

# Design tips.

1. Avoid unnecessary match statements and catchall match cases.

2. Get rid of conditionals that result in `true` or `false`.

3. If your helper function is only used in one function, define the helper inside the other function.

# Defining your own types[^1]
## Type synonyms

A “type synonym” is a new name for an already existing type.

```ocaml
type point = float * float ;;
type matrix = float list list ;;
```

Anywhere that a `float * float` is expected, you could use `point`, and vice-versa. The two are completely interchangeable. The function `dist_from_origin` doesn’t care whether you pass it a value that is annotated as a `point` versus as a `float * float`. one vs. the other:

```ocaml
let dist_from_origin ((x, y) : point) : float =
	sqrt (x ** 2 +. y ** 2) ;;
        
let pt : point = (3., 4.) ;;
let floatpair : float*float = (3., 4.) ;;

let pt_dist  = dist_from_origin pt ;;
let float_dist = dist_from_origin floatpair ;;
```

## Variants
Here are some examples of constant variants. (Syntax note: These all
have to be capitalized, so OCaml knows you’re defining a new variant.)

```ocaml
type day = Sun | Mon | Tue | Wed
		 | Thu | Fri | Sat ;;
type bender = Air | Fire | Water | Earth ;;
type class_year = FirstYear | Sophomore | Junior | Senior | Alum ;;
```

## Records.
You can define a record out of pre-existing types. (Note that I’m using `class_year` from the previous section.)

```ocaml
type student = {name : string;
				concentration : string;
				year : class_year;} ;;
let melissa = {name = "Melissa Kwan"; year = Junior} ;;
```

When taking in a record as input, you can use these handy constructs to make your code more readable.

```ocaml
(* Bad *)
let label (s : student) : string =
	match p with
	| {concentration = c; year = y; _} ->
		c ^ " " ^ y ;;

(* Using field punning and field selection *)
let label ({concentration; year; _} : student) : string =
	concentration ^ " " ^ year ;;
```

## Algebraic Data Types
Putting all these types together, you get algebraic data types! How to define a variant type:

```ocaml
type t =
| C1 [of t1]
| ...
| Cn [of tn]
```

The square brackets above denote the the [of ti] is optional. Every constructor may individually either carry no data or carry data.  Variants also make it possible to discriminate which tag a value was constructed with, even if multiple constructors carry the same type.

```ocaml
type t = Left of int | Right of int ;;
let x = Left 1 ;;
let double_right i =
	match i with
	| Left i -> i
	| Right i -> 2*i ;;
```

Using variants, we can express a type that represents the union of several other types, but in a type-safe way. Here, for example, is a type that represents either a `string` or an `int`: Variants can provide a type-safe way of doing something that might before have seemed impossible.

```ocaml
type string_or_int =
| String of string
| Int of int
```

**Problem 3.4.1. ** Write a function `sum` that returns the sum of a list of `string_or_int` elements. It should return a value tagged with `String` if the last element of the list is a string, `Int` otherwise.

**Solution.** This problem is a spin on the typical sum problem, which we already know how to do with `fold_left`. The goal is to convert our `string_or_int list` input into a list of integers that we can sum as usual. To do this, we define an `extract_int` function to return the integer value of a `string_or_int` type. I would typically define a helper function inside the `sum` function, but since this helper function is so general (and probably useful for other functions), I’m defining it separately.

Now we have a way of extracting the integer from each list. We map this function over the list to get a list of integers, and we find integer sum using `fold_left`.

All that remains is to figure out whether the `string_or_int` output should be packaged as a `String` or an `Int` in the final result, which requires knowing whether the last element is a `String` or an `Int`. We can identify the last element of the list by using `List.hd (List.rev lst)`. Just one problem: `List.hd` throws an error when called on an empty list. After checking for the empty list case, then we’re all set.

```ocaml
let extract_int (x : string_or_int) : int =
    match x with
    | String s -> int_of_string s
    | Int i -> i ;;

let rec sum (lst : string_or_int list): string_or_int =
    if lst = [] then Int 0
    else
        let int_lst = List.map extract_int lst in
        let int_sum = List.fold_left (+) 0 int_lst  in
        match List.hd (List.rev lst) with
        | String _ -> String (string_of_int int_sum)
        | Int _ -> Int int_sum ;;
```

## Parametrized + Recursive variants.

**Recursive variants.** Variant types may mention their own name inside
their own body.

**Parametrized variants.** In the following example, `bintree` is a “type constructor”: there is no concrete way to write a value of type `bintree`. But we can write value of type `int bintree` and `string bintree`. Think of a type constructor as being like a function, but one that maps types to types, rather than values to values. Binary trees are both parametrized and recursive.

```ocaml
type 'a bintree =
| Leaf
| Node of 'a * 'a bintree * 'a bintree ;;

let rec leaf_count (tree : 'a bintree) : int =
match tree with
| Leaf -> 1
| Node (_, left, right) -> leaf_count left + leaf_count right ;;
```

**Problem 3.5.1. ** An updated `find` from lab. Write a function called
`find_level_opt` that searches for an element within a tree and returns
an `int option` of the “level” of the tree at which it found it. If it
does not find the element, it outputs `None.`

**Solution.** Options are super annoying, so we’re going to try at all
costs try to find ways to minimize our match statements with `Some` and
`None`. To do so, we’re defining an internal helper function called
`find_level_int` that simply returns -1 if the value is not found. We’ll
deal with the options later.

How do we write this function? Our base case is if we hit a `Leaf`, at
which point we know we haven’t found the value and return -1. If we
match with a node, we check whether the node has the desired value. If
it does, we return the current level. If it doesn’t, then we search the
left and the right children with `(level + 1)`, which signals that we’re
moving down a level.

If at least one of the left or the right tree did not find the value
(which is highly likely), we set the level to be the maximum of the two
found levels. That way, if one of the left or right trees found the
value, we’ll note the true level and not -1. If we reach the else
statement, we know that both trees found the value, so we return the
minimum to show which side technically found it “first.”

Finally, we call our helper function. If it returns $-1$, we know that
it wasn’t found, so we return `None`. Otherwise, we know it was found,
so we return `Some depth`.

```ocaml
let find_level_opt  (tree : 'a bintree) (value : 'a) : int option =
    let rec find_level_int tree level =
        match tree with
        | Leaf -> -1
        | Node (stored, left, right) ->
            if stored = value then level
            else
                let l = find_level_int left (level + 1) in
                let r = find_level_int right (level + 1) in
                if l == -1 || r == -1 then max l r
                else min l r
    in
    let depth = find_level_int tree 0 in
    if depth = -1 then None
    else Some depth ;;
```

**Problem 3.5.2.** A min-heap is a data structure whose “children” are
always greater than its “root.” Likewise, a tree is not a min-heap if
there are leaves somewhere in the tree that are smaller than the values
stored in the nodes. Write a function `is_heap` that returns `true` if
the given `int bintree` is a min-heap and `false` otherwise.

**Solution.** To check whether the tree is a min-heap, we’ll need to
check every node to see whether its value is less than its childrens’.
However, in order to extract the value of the `left` and `right`
subtree, it looks like we’ll need to do more annoying match statements
to handle the case where `left` is a `Node` and `right` is a `Leaf`,
etc. etc. God forbid.

We can make this slightly better by defining a function called
`min_preserved` to check whether a given value is less than the value of
a given child. If the child is a leaf, it automatically returns true.

Now that we have the helper function, we can cut down on our match
statements. At this point, we just need to recur through each node of
the tree, returning true only if each node and its subnodes pass the
check.

```ocaml
let rec is_heap (tree : int bintree) : bool =
    let min_preserved value child =
        match child with
        | Leaf -> true
        | Node (stored, _, _) -> value < stored
    in
    match tree with
    | Leaf -> true
    | Node (stored, left, right) ->
        min_preserved stored left
        && min_preserved stored right
        && is_heap left
        && is_heap right ;;
```

**Problem 3.5.3 (Challenge).** Write a function `fold_tree` that takes in a function `f` of type `’a -> ’b -> ’a`, a variable `acc` of type `’a`, and a binary tree of type `’b bintree`. The function `fold_tree` applies `f` to `acc` and an element in the binary tree, sets `acc` to the output, then repeats until all elements in the binary tree have been iterated through; at the very end, `fold_tree` returns the final value of `acc`. The order of elements that `fold_tree` iterates through is the Depth-First-Search order.

**Solution.** As always, the base case for `fold` should give you back `acc`. The tricky part is knowing how to incorporate the left and the right. Because depth first search will go entirely down one path before incorporating its result into the top, we can define `leftFoldResult` as the recursive call on `fold_tree f acc leftTree`. We set that to be the new value of the accumulator, and we call `fold_tree` once more on the right sub-branch. Finally, we have the value of the accumulator that reflects both the left and the right subtrees. All we have the do is incorporate the folding function on the top element, leading to the result `f rightFoldResult el`.

```ocaml
let rec fold_tree (f: 'a -> 'b -> 'a) (acc: 'a) (root: 'b bintree) : 'a =
    match root with
    | Leaf -> acc
    | Node (el, leftTree, rightTree) ->
        let leftFoldResult = fold_tree f acc leftTree in
        let rightFoldResult = fold_tree f leftFoldResult rightTree in
        f rightFoldResult el ;;
```

# Advice on Bignums
## Exercise-specific tips

1.  `less / greater / equal`: Think about the role of both negatives and
    the lengths of the respective bignums. You shouldn’t need to define
    a condition for each one.

2.  `int_to_bignum, bignum_to_int`: Write out integers in terms of
    powers of `cBASE`. How can you use `mod` and `/` to break up the
    bignum?

3.  `add`: Think about carrying. How does that factor into `cBASE`?

4.  `multiply`: Again, think about powers of `cBASE`. It’s helpful to
    look at an example of the grade-school algorithm for multiplication.
    The only thing that changed is that `cBASE` is no longer $10$; it’s
    $1000$. How can you use your intuition from grade school to solve
    this problem in a similar way?

5.  **Design considerations.**

    -   Make sure you’re not spelling out repetitive operations within
        functions; define those as internal helper functions.

    -   If your conditional depends on multiple things

## Testing: Utop Alternatives
1.  `#use`. Change directories (`cd`) into your lab / problem set
    folder. Then, from `utop`, you can use `#use mapfold.ml` to directly
    import all your functions.

2.  `.makefile`. Create a file called `.makefile` in your problem set
    directory with commands adjusted based on the template below. Then,
    you can run `make all` on your command line to build the files, and
    `./mapfold_tests.byte` to run the tests. Before you submit to
    Gradescrope, make sure to run `make clean` in your directory to
    remove all the `byte` files that resulted from the build.

```
all: ps2 ps2_tests

ps2: mapfold.ml
	ocamlbuild -use-ocamlfind mapfold.byte

ps2_tests: mapfold_tests.ml
	ocamlbuild -use-ocamlfind mapfold_tests.byte

clean:
	rm -rf _build *.byte

```

[^1]: Adapted from Cornell’s CS 3110.
