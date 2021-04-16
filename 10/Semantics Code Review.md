# Environment Semantics
**April 16, 2021**
Notes adapted from Ahan Malhotra

## The Environment Model
So far we've been using the _substitution model_ to evaluate programs. It's a great mental model for evaluation, and it's commonly used in programming language theory.

But when it comes to implementation, the substitution model is not the best choice. It's too eager: it substitutes for every occurrence of a variable, even if that occurrence will never be needed.
- For example, let `x = 42` in e will require crawling over all of `e`, which might be a very large expression, even if `x` never occurs in `e`, or even if `x` occurs only inside a branch of an if expression that never ends up being evaluated.

For sake of efficiency, it would be better to substitute **lazily**: only when the value of a variable is needed should the interpreter have to do the substitution. That's the key idea behind the **environment model**. In this model, there is a data structure called the **dynamic environment**, or just "environment" for short, that is a dictionary mapping variable names to values. Whenever the value of a variable is needed, it's looked up in that dictionary.

### Notation
-  `{x1 -> v1}` represents an environment that binds `x1` to `v1`
-   `E(x)` represents the binding of `x` in the environment.
-    `E{x -> v}` represents the environment `E` with the variable `x` additionally bound to the value `v`.

## The Naive Solution
In this section, we will show why simply looking up a variable in one dictionary is incorrect and use that to motivate the need for scope.

Remember that the rule for variables just says to lookup the variable name in the environment.

```ocaml
E ⊢ x ⇓ E(x)
```

This rule for functions says that an anonymous function evaluates just to itself. After all, functions are values:

```ocaml
E ⊢ fun x -> P ⇓ fun x -> P
```

Finally, this rule for application says to evaluate the left-hand side `P` to a function `fun x -> B`, the right-hand side to a value `vQ`, then to evaluate the body `B` of the function in an extended environment that maps the function's argument `x` to `vQ`:

```ocaml
E ⊢ P Q ⇓  
        | E ⊢ P ⇓ fun x -> B
		| E ⊢ Q ⇓ vQ  
        | E{x -> vQ} ⊢ B ⇓ vB
		⇓ vB
```

Seems reasonable, right? The problem is, it's **wrong**. At least, it's wrong if you want evaluation to behave the same as OCaml. Or, to be honest, nearly any other modern language.

Let's look at example:

Suppose we have two more language features:

**Integer constants** always evaluate to themselves:
```ocaml
E ⊢ n ⇓ n
```

**Let expressions** have the following semantics:
```ocaml
E ⊢ let x = D in B ⇓
				   | E ⊢ D ⇓ vD  
				   | E{x -> vD} ⊢ B ⇓ vB
				   ⇓ vB
```

Note: `let` expressions are actually just syntactic sugar, because `let x = D in B` can be rewritten as `(fun x -> B) D`.

We can use this idea to evaluate the following expression:
```ocaml
let x = 1 in  
let f = fun y -> x in
let x = 2 in  
f0
```

If we used the semantic rules we just defined, we would get:
1.  `let x = 1` would produce the environment `{x -> 1}`
2.   `let f = fun y -> x` would produce the environment `{x -> 1; f -> (fun y -> x)}`
3.   `let x = 2` would produce the environment `{x -> 2; f -> (fun y -> x)}`. Note how the binding of `x` to `1` us shadowed by the new binding.
4.   Now we would evaluate `{x -> 2,f -> (fun y -> x)} ⊢ f 0`:

```ocaml
{x -> 2; f -> (fun y -> x)} ⊢ f 0
	⇓
	| {x -> 2; f -> (fun y -> x)} ⊢ f ⇓ fun y -> x
	| {x -> 2; f -> (fun y -> x)} ⊢ 0 ⇓ 0  
	| {x -> 2; f -> (fun y -> x)}{y -> 0} ⊢ x
		⇓  
		| {x -> 2; f -> (fun y -> x); y -> 0} ⊢ x ⇓ 2
		⇓ 2
	⇓ 2
```

5. Therefore, the result is 2.

But if we were to plug this into `utop` or use the substitution model, this would evaluate as follows:

```ocaml
#   let x = 1 in
    let f = fun y -> x in
    let x = 2 in
    f 0 ;;

- : int = 1
```

And the result is therefore 1. Obviously, 1 and 2 are different answers!

What went wrong? The problem has to do with **scope**.

## Dynamic vs. Lexical Scope
There are two different ways to understand the scope of a variable: variables can be dynamically scoped or lexically scoped.

It all comes down to the environment that is used when a function body is being evaluated:
- **Dynamic scope**: the body of a function is evaluated in the *current dynamic environment* at the time the function is applied, not the old dynamic environment that existed at the time the function was defined.
- **Lexical scope**: the body of a function is evaluated in the *old dynamic environment* that existed at the time the function was defined, not the current environment when the function is applied.

The rule of dynamic scope is what our incorrect semantics, above, implemented. Let's look back at the semantics of function application:

```ocaml
E ⊢ P Q ⇓  
    	| E ⊢ P ⇓ fun x -> B
		| E ⊢ Q ⇓ fun x -> vQ
		| E{x -> vQ} ⊢ B ⇓ vB
		⇓ vB
```

Note how the body `B` is being evaluated in the same environment `E` as when the function is applied. In the example program:

```ocaml
let x = 1 in
let f = fun y -> x in
let x = 2 in
f 0 ;;
```

that means that `f` is evaluated in an environment in which `x` is bound to 2, because that's the most recent binding of `x`.

OCaml implements lexical scope, which means that `x` is bound to 1 in the body of `f` where defined, and the later binding of `x` to 2 doesn't change that fact. Note that this coincides with the substitution model's result.

**Discussion question**: What are the pros and cons of lexical vs. dynamic scoping? Which do you think is more popular in the real world?

## Implementing Lexical Scope
The question then becomes, how do we implement lexical scope? It seems to require time travel, because function bodies need to be evaluated in old dynamic environment that have long since disappeared.

The language implementation must arrange to keep old environments around. And that is indeed what OCaml and other languages must do. They use a data structure called a **closure** for this purpose.

A **closure** consists of:
-  a code part, which contains a function `fun x -> B`, and
- an environment part, which contains the environment `E` at the time that function was defined.

You can think of a closure as being like a pair, except that there's no way to directly write a closure in OCaml source code, and there's no way to destruct the pair into its components in OCaml source code. The pair is entirely hidden from you by the language implementation.

**Closure notation:** `[E ⊢ P]`, where `P` is the function and `E` is the environment. Think about the brackets as "packaging" the environment and the function together.

We must update our function rule, since we need to capture the environment with it. Instead of simply returning a function, the function rule will now return a closure!

```ocaml
E ⊢ fun x -> P ⇓ [E ⊢ fun x -> P]
```

Now, when we evaluate a function, we now need to use that closure. We’ll update our application rule to do just that.

```ocaml
Ed ⊢ P Q ⇓  
		 | Ed ⊢ P ⇓ [El ⊢ fun x -> B]
		 | Ed ⊢ Q ⇓ vQ  
		 | El{x -> vQ} ⊢ B ⇓ vB
		 ⇓ vB
```

That rule uses the closure's environment `El` to evaluate the function body `B`. The derived rule for let expressions remains unchanged:

```ocaml
E ⊢ let x = D in B
	⇓
	| E ⊢ D ⇓ vD  
	| E{x -> vD} ⊢ B ⇓ vB ⇓ vB
```

That's because the defining environment for the body `e2` is the same as the current environment `env` when the let expression is being evaluated.

**Problem 1:** Evaluate the following expression using environment semantic rules. Assume an initially empty environment.

```ocaml
let x = 2 in
let f = fun y -> x + y in
let x = 8 in
f x
```

**Solution 1:**
(Indentation updated so the rules fit on one line.)
```ocaml
{} ⊢ let x = 2 in let f = fun y -> x + y in let x = 8 in f x
⇓
| {} ⊢ 2 ⇓ 2 														(R_int)
| {x -> 2} ⊢ let f = fun y -> x + y in let x = 8 in f x
|	⇓
|	| {x -> 2} ⊢ fun y -> x + y ⇓ [{x -> 2} ⊢ fun y -> x + y]		(R_fun)
|	| {x -> 2, f -> [{x -> 2} ⊢ fun y -> x + y]} ⊢ let x = 8 in f x
|	|	⇓
|	|	| {x -> 2, f -> [{x -> 2} ⊢ fun y -> x + y]} ⊢ 8 ⇓ 8 		(R_int)
|	|	| {x -> 8, f -> [{x -> 2} ⊢ fun y -> x + y]} ⊢ f x
|	|	|	⇓
|	|	|	| {x -> 8, f -> [{x -> 2} ⊢ fun y -> x + y]}
|	|	|	| ⊢ f ⇓ [{x -> 2} ⊢ fun y -> x + y]						(R_var)
|	|	|	| {x -> 8, f -> [{x -> 2} ⊢ fun y -> x + y]} ⊢ x ⇓ 8	(R_var)
|	|	|	| {x -> 2, y -> 8} ⊢ x + y ⇓ 8
|	|	|	|	⇓ 
|	|	|	|	| {x -> 2, y -> 8} ⊢ x ⇓ 2							(R_var)
|	|	|	|	| {x -> 2, y -> 8} ⊢ y ⇓ 8							(R_var)
|	|	|	|	⇓ 10												(R_+)
|	|	|	⇓ 10													(R_app)
|	|	⇓ 10														(R_let)
|	⇓ 10															(R_let)
⇓ 10																(R_let)
```

## Implementing references: Evaluation with Stores
To implement references, we need a model of memory. Unlike the environment, this abstract model of memory, called a store can be modified without changing the mapping of variables to values. In evaluating references, we add to the environment a mapping from a variable to a location, and we add to the store a mapping from that location to a value. Expressions are evaluated in the context of an environment and a store. The semantics specify when the environment and store are modified. As evaluations can essentially always potentially modify the store, each step in an evaluation returns a (potentially) modified store. This new store is used in the succeeding step.

We define a full set of rules for lexical environment semantics with stores in Figure 19.4 in the textbook. Stores could also be implemented with dynamic environment semantics, but we do not provide these semantics.

**Problem 2:** Evaluate the following expression using the lexical environment semantic rules for imperative programming in Figure 19.4. Assume an initially empty environment.

```ocaml
let x = ref 42 in
(x := !x - 21; !x) + !x ;;
```

**Solution 2:**
```ocaml
{}, {} ⊢ let x = ref 42 in (x := !x - 21; !x) + !x
	⇓
	| {}, {} ⊢ ref 42
	|	⇓
	|	| {}, {} ⊢ 42 ⇓ 42, {}										(R_int)
	|	⇓ l1, {l1 -> 42}											(R_ref)
	| {x -> l1}, {l1 -> 42} ⊢ (x := !x - 21; !x) + !x
	| 	⇓
	|	| {x -> l1}, {l1 -> 42} ⊢ x := !x - 21; !x
	|	| 	⇓
	|	|	| {x -> l1}, {l1 -> 42} ⊢ x := !x - 21;
	|	|	| 	⇓
	|	|	| 	| {x -> l1}, {l1 -> 42} ⊢ x := !x - 21
	|	|	| 	|	⇓
	|	|	| 	|	{x -> l1}, {l1 -> 42} ⊢ !x
	|	|	| 	|	|	⇓
	|	|	| 	|	|	| {x -> l1}, {l1 -> 42} x ⇓ l1, {l1 -> 42} 	(R_var)
	|	|	| 	|	|	⇓ 42, {l1 -> 42}							(R_deref)
	|	|	| 	| {x -> l1}, {l1 -> 42} ⊢ 21 ⇓ 21, {l1 -> 42}		(R_int)
	|	|	| 	| 	⇓ 21, {l1 -> 42}								(R_-)
	|	|	| 	⇓ (), {l1 -> 21}								   (R_assign)
	|	|	| {x -> l1}, {l1 -> 21} ⊢ !x
	|	|	|	⇓
	|	|	| 	| {x -> l1}, {l1 -> 21} ⊢ x ⇓ l1, {l1 -> 21}		(R_var)
	|	|	| 	⇓ 21, {l1 -> 21}									(R_deref)
	|	|	⇓ 21, {l1 -> 21}										(R_seq)
	|	|	{x -> l1}, {l1 -> 21} ⊢ !x
	|	|	|	⇓
	|	|	| 	| {x -> l1}, {l1 -> 21} ⊢ x ⇓ l1, {l1 -> 21}		(R_var)
	|	|	⇓ 21, {l1 -> 21}										(R_deref)
	|	⇓ 42, {l1 -> 21}											(R_+)
	⇓ 42, {l1 -> 21}												(R_let)
```

## Recap
**TL;DR**
- **Substitution**: Substitute the free-variables in a given expression for values to which they are bound, continuing until all bindings are complete.
- **Environment**: Do not substitute values immediately after variable definitions.
	- **Dynamic**: Uses an environment structure to track the value of each variable, with variables adopting the values stored in the environment at runtime.
		- Allows future definitions to inadvertently change earlier definitions.
	- **Lexical**: Instead of evaluating variables with their values at runtime, we retain their values from the environment in which they were originally defined.
		- To handle this, we define a closure: a structure that stores the value of the expression and a snapshot of the current environment.

Why did we take the leap from substitution to environment semantics?
-   **Side effects:** Environment semantics allow us to reason about computation with side effects, which we cannot do with substitution semantics (why?).
	-   As a result, they can handle imperative constructs.
-   **Efficiency:** In practice, most programming languages implement environment semantics because its evaluation is more efficient.**
	- Instead of substituting greedily, we evaluate expressions in an environment, which is a mapping of variables to values.

**Problem 3:** Imagine that someone gives you an expression with an unbound variable. When will you first detect this error in the substitution case? What about the environment case?

**Solution 3**
- In substitution evaluation, you’re greedily substituting every variable into the expression going downwards. So if you complete the recursion and find yourself with a “plain variable” base case, you know there’s a problem.
- In environment evaluation, you’ll know there’s an issue if you try to look up a variable and find that there’s no mapping for it in the environment.

## Final project
1. Convert abstract syntax to strings, for testing and output purposes
2. Identify free variables
3. Implement substitution semantics
4. `eval_s`: evaluate with substitution semantics
5. Write functionality for environment module
6. `eval_d`: evaluate with dynamically scoped environment semantics

**Extension 1:** `eval_l`: evaluate with lexically scoped environment semantics

**Other possible extensions:** Different types, references (detailed in textbook, analogous to the store we’re covering in this code review).

**The Writeup:** Talk about your extensions: what you found and learned!
Here is where you’ll get exposed if you only did the lexical extension.

### Things to watch out for
- How do you define semantics for `let rec`?
	- You’ll be storing “unevaluated” and “evaluated” forms of functions to make sure that you can call a function before it’s formally defined.

### Misc. advice
- **Logistics:** Start early and put a good amount of effort into the writeup.
	- **Important**: Before starting the final project, you must install the opam package `menhir`, which is a parser generator used to build a MiniML REPL. Run the following command to install `menhir`: `opam install -y menhir`.
	- Download the latest version of the textbook
	- Expect around 2 problem sets worth of work
	- No late days
- **Extensions:** You’ll do well grade-wise if you implement `eval_l` and a couple of other atomic types, but I’d encourage you to do more if you’re interested!
- **Code quality:** You’re going to be writing `eval_s`, `eval_d`, and (most likely) `eval_l`, and they’re going to have a lot in common. Can you abstract functionality from the three of them to avoid repeating code?
	- Many of the tricks from PSET 4 will also apply to the final project. Try to make your implementation as clean as possible!
