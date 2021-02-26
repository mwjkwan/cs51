# to_int

cBASE = 1000
[1; 2; 3] = 1 * 1000^2 + 2 * 1000^1 + 3 * 1000^0

cBASE = 1000
[1; 2; 3]

STEP 1: 0
STEP 2: 1000 * 0 + 1
STEP 3: 1000 * (1000 * 0 + 1) + 2
STEP 4: 1000 * (1000 * (1000 * 0 + 1) + 2) + 3
1002003

let to_int_helper (coeffs: int list) (total: int) =
	match coeffs with
	| [] -> 0
	| hd :: tl -> let new_total = cBASE * total + hd in to_int_helper tl new_total

