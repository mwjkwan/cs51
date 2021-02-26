# Less conceptual

1. Check sign: if the signs don't match, then the one with the negative sign will always be smaller. (b1.neg <> b2.neg)
	1. let b1 = {neg = true, coeffs = [7; 3; 5; 9; 4; 6]}
	2. let b2 = {neg = false, coeffs = [1; 3; 5]}
	3. -> `less b1 b2` is true (note that it's just the same as b1.neg)
		1. if b1.neg = true then true else false -> write as b1.neg
2. Length of the list. (using List.length)
	1. List.length b1.coeffs > List.length b2.coeffs: false if the signs are positive true if signs are negative.
		1. Output what b1.neg is
	2. List.length b1.coeffs < List.length b2.coeffs: true if signs are positive, false if signs are negative
		1. Output not b1.neg
3. Next step: compare individual digits — go element by element <- HELPER FUNCTION!
	1. Compare first two elements of b1 and b2.
		1. If h1 < h2 then not b1.neg
		2. If h1 > h2 then b1.neg
		3. If h1 = h2, then we call recursively on the tails

- b1.neg tells you whether they're both positive or both negative


cBASE = 10
	1. let b1 = {neg = true, coeffs = [7; 3; 5; 9; 4; 6]} -> -735946
	2. let b2 = {neg = true, coeffs = [1; 3; 5]} -> -135
	`less b1 b2` -> List.length b1 > List.length b2 and they're both negative, so that's true.
	
1. let b1 = {neg = false, coeffs = [7; 3; 5; 9; 4; 6]} 735946
2. let b2 = {neg = false, coeffs = [1; 3; 5; 4; 4; 4]}  135444
	`less b1 b2` -> List.length b1 > List.length b2 and they're both negative, so that's true.


Structure of a bignum
- neg: boolean
- coeffs: int list

cBASE = 10
let b1 = {neg = true, coeffs = [1; 3; 5]}
-135

cBASE = 10
let b1 = {neg = true, coeffs = [2; 3; 5]}
-235

cBASE = 1000
let b1 = {neg = true, coeffs = [1; 3; 5]}
1003005

cBASE = 57
let b1 = {neg = true, coeffs = [1; 3; 5]}
5 * 57^0 + 3 * 57^1 + 1 * 57^2


First indicator: the sign.
- If one of them is negative, and the other is positive, then the negative one is smaller
	- if b1 is negative and b2 is positive, then `less b1 b2` is true
	- if they have the same sign, the sign doesn't give enough information to tell you which one is bigger, so you need to use other information

Second indicator: Length
At this point, if you even care about the length, then you know that b1 and b2 have the same sign.
- If b1 is negative and b1 is "longer" than b2, then `less b1 b2` is true

Third indicator: the numbers themselves
At this, 
- Check the first integer - if they're not equal, then 


let b1 = {neg = true, coeffs = [5; 1; 7; 8; 9]}
let b2 = {neg = true, coeffs = [5; 1; 2; 4; 7]}

- Based on the first digit, then we check the second (recursive call to the tails of both lists)
	- RECURSIVE CALL ON [1; 7; 8; 9], [1; 2; 4; 7] : Based on the first digits (1), we also can't tell, so we make another recursive call
		- RECURSIVE CALL ON [7; 8; 9], [2; 4; 7] : Based on the first digits, the left one is bigger in *magnitude*, so we would give the answer false, but since the sign is negative, we flip our answer to true.


What would you get in the empty list case? ([], [])
- Return false