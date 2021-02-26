```ocaml

DESIRED RESULT
['z', 'z', 'y', 'x'] -> [(2, 'z'), (1, 'y'), (1, 'x')]

to_run_length ['z', 'z', 'y', 'x']
			= to_run_length ['z', 'y', 'x'] = [(2, 'z'), (1, 'y'), (1, 'x')] 	 
			except with a 2 instead of a 1
			
[]
				



If it's empty, then return empty list []
Otherwise, split this into hd :: tl

NEXT STEP: Figure out how to translate to_run_length tl into to_run_length lst, where lst = hd :: tl

hd = 'r'
tl = []
to_run_length tl = []
to_run_length lst = [(1, 'r')]

1. Insight 1: If to_run_length tl is the empty list, then our answer is just [(1, hd)]



hd = 'z'
tl = ['z', 'y', 'x']
to_run_length tl = [(1, 'z'), (1, 'y'), (1, 'x')]
to_run_length lst [(2, 'z'), (1, 'y'), (1, 'x')]


2. Insight 2: If the first character of to_run_length tl is the same as the head, then we should just bump that number up by 1 (instead of adding a new tuple to the front.)

[(1, 'z'), (1, 'y'), (1, 'x')]
Isolate the first character (which should be 'z')
Check whether 'z' is the same as head (which it is)
Now, return [(1+1, 'z'), (1, 'y'), (1, 'x')]



hd = 'a'
tl = ['z', 'y', 'a']
to_run_length tl = [(1, 'z'), (1, 'y'), (1, 'a')]
to_run_length lst = [(1, 'a'), (1, 'z'), (1, 'y'), (1, 'a')]


3. Insight 3: If the first character of to_run_length tl is DIFFERENT from the head, then we should add a tuple (1, hd) to the front.

[(1, 'z'), (1, 'y'), (1, 'x')]
Isolate the first character (which should be 'z')
Check whether 'z' is the same as head (which it isn't)
Now, return [(1, 'a'), (1, 'z'), (1, 'y'), (1, 'x')]

Approach:
Match your list to head and tail

RETURN TYPE
(int * char) list



```


```ocaml

GOAL:
[(1, 'a'); (2, 'b'); (1, 'z')] -> ['a'; 'b'; 'b'; 'z']
TYPE: (int * char) list -> char list

RECURSION CONCRETELY:
Suppose we have the answer to from_run_length [(2, 'b'); (1, 'z')]
Answer to smaller problem: ['b'; 'b'; 'z']
How do you get from_run_length on [(1, 'a'); (2, 'b'); (1, 'z')]?
What we want: ['a'; 'b'; 'b'; 'z'] = 'a' :: ['b'; 'b'; 'z']
= 'a' :: from_run_length [(2, 'b'); (1, 'z')]

CASE 2
[(3, 'a'); (1, 'z')] -> ['a'; 'a'; 'a'; 'z']
Suppose we have the answer to from_run_length [(2, 'a'); (1, 'z')]
from_run_length [(2, 'a'); (1, 'z')] = ['a'; 'a'; 'z']
How do we express from_run_legnth [(3, 'a'); (1, 'z')] in terms of from_run_length [(2, 'a'); (1, 'z')]?
a :: from_run_length [(2, 'a'); (1, 'z')]

CASE 3 (Easier way of looking at CASE 1)
[(0, 'a'); (3, 'z')] -> ['z'; 'z'; 'z']
from_run_length [(3, 'z')] = ['z'; 'z'; 'z']

Express from_run_length [(0, 'a'); (3, 'z')] in terms of from_run_length [(3, 'z')]


from_run_length [(3, 'a'); (1, 'z')]
= 'a' :: from_run_length [(2, 'a'); (1, 'z')]
		 = 'a' :: from_run_length [(1, 'a'); (1, 'z')]
		 		  = 'a' :: from_run_length [(0, 'a'); (1, 'z')]
				  		   = from_run_length [(1, 'z')]
						   	 =  'z' :: [(0, 'z')]
							 		   = []




```