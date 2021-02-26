# `times` strategy

These notes are informal. If you have questions, email mkwan@college.harvard.edu

## Part 1: How do we break down this problem?
Our final goal is to multiply two bignums, each with their own sign. We can split this up into parts:
- **Part 1**: Multiply two lists of integers, disregard the signs
	- **Part 1a**: Multiply one list of integers by a single digit
	- **Part 1b** Add the intermediate results together.
- **Part 2**: Add the signs in later to form the final result

To make this concrete, let's look the example 543 * 27, or [3; 4; 5] x [7; 2].

> Note that here we are reversing the coefficients from the usual `bignum` representation by putting the least significant digit first. When we reverse the list,[1; 2; 3; 4; 5] represents 5004003002001 (assuming cBASE is 1000).

- Part 1
	- Part 1a
		- [3; 4; 5] * 7 = [1; 0; 8; 3]
		- [3; 4; 5] * 2 = [6; 8; 0; 1]
	- Part 1b
		- [3; 4; 5] * [7; 2]
			- = 543 * 7 + 10 * 543 * 2
			- = [3; 4; 5] * 7 + [0] @ ([3; 4; 5] * 2)
			- = [1; 6; 6; 4; 1]
- Part 2
	- {neg=false; coeffs=[1; 6; 6; 4; 1]}

## Step 1a: Multiplying 1 integer by a bignum
1. Multiplied the least significant digit by your number: (x * y + carry)
	1. Value in your multiplication = (x * y + carry) % cBASE
		1. (7 * 4) % 10 -> 8 (this is our "digit")
	2. Carry = (x * y + carry) / cBASE
		1. (7 * 4) / 10 -> 2 (this is our "carry")

What should carry start out as? Answer: 0.

What should we do if the list we're trying to multiply is empty?
lst = [], n = 234, carry = 7
Answer: [7]

Design consideration: Whenever we have single elements in lists, we want to write as [elt], not elt :: [] 

The function we want, takes in a reversed list (such as [1; 2; 3; 4; 5]), an integer (aka a "digit"), a carry value, and outputs a final list

37 x 4 = 148

[3; 7]
4

First step: reverse to get [7; 3], starting carry value of 0

4 * 7 + 0  = 28 (digit x first digit of the list + carry from before)
28 -> split into **digit** = 28%10 = 8, **carry** =28/10 = 2
Then we make our recursive call:
8 :: [   RECURSIVE CALL   ]

**What should go in our recursive call?**
- list of integers: tail of the list
- integer: n
- carry: carry that we calculated above

And that's it!

## Add the intermediate results together into a shifted list

Example to get us started

37 x 24 = 148 + 740

[8; 4; 1] + [0] @ [4; 7]

We assume we know how to do 4 x [7; 3] and 2 x [7; 3] at this point.

Q: What is the best way to keep track something that needs to change when we make recursive calls?
A: We can use an accumulator. An accumulator is an input to a recursive function that keeps track of some value that you want to pass in between recursive calls.

In this function, we have two accumulators:

**Accumulator 1:** the zeros we need to shift the digit-by-digit results. For example, when multiplying 37 by 24, we don't need to shift 37 * 4 at all, but we need to shift 37 * 2 by one digit.
Starts at []
[0]
[0; 0]
[0; 0; 0]
Each time, we add one 0 to what it was before.

**Accumulator 2:** product_so_far
This is the accumulator that keeps track of our product so far. If we're multiplying 543 by 127, it would first store 7 * 543, then 27 * 543, and finally 127 * 543.

Q: How does our recursive call work?
- constant list: earlier coefficients (this would be equivalent to [3; 4; 5] in the above example.
- remaining list: this would be equivalent to [2; 1] on the first recursive call, then [1], then [].)
- zeros: we know that every time we make a call, the digit is getting _more_ significant, so we need to add a zero to the list. So zeros will start out as [], and then it will become [0], [0; 0], etc. on subsequent calls
- updated progress: this is our accumulator! so of course, we're going to want to use our previous accumulator value, and then we'll add the result of multiplying the constant list (aka [3; 4; 5]) by the most recent digit (which will be 7, then 2, then 1). Notice that we already have a function `plus` to add bignum values, so you can use that.
	- plus
	- product_so_far (aka our accumulator)
	- most recent digit * the constant list (use function from Part 1a), shifted over by a certain number of zeros. Remember that we're storing an accumulator of zeros for this very reason!










