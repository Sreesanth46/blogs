---
title: Some fancy Bitwise Tricks
date: 2024-08-11 01:37:26 +0530
---
Bitwise operations are often underappreciated, lurking in the shadows of more straightforward arithmetic and logical operations. However, these operations are powerful tools that, when used correctly, can lead to elegant and efficient solutions to common programming problems. In this blog post, we'll explore two fascinating applications of bitwise operations: using XOR to swap variables without a temporary variable and using AND to check if a number is even or odd.

### Swapping Variables Using XOR

Swapping two variables is a fundamental task in programming. Traditionally, this is done using a temporary variable:
```js
// Traditional swap 
let a = 10;
let b = 20;
const temp = a; 
a = b; 
b = temp;

// Now a = 20 and b = 10
```

However, with the XOR bitwise operator, you can swap two variables without needing a temporary variable. Here's how it works:

```js
let a = 10;
let b = 20;
a ^= b // XOR a = a ^ b
b ^= a
a ^= b

// Now a = 20 and b = 10, without the temp!!
  /**
   * Let's see how this works.
   * for simplicity can take a = 1 and b = 2
   * Binary representation -> a = 0001 and b = 0010
   * a = a XOR b -> 0001 XOR 0010 = 0011
   * b = b XOR a -> 0010 XOR 0011 = 0001
   * a = a XOR b -> 0011 XOR 0001 = 0010
   */
```

After these three operations, `a` and `b` have swapped their values without using any extra memory. This method is not only space-efficient but also showcases the versatility of the XOR operation.

### Checking If a Number Is Even or Odd Using AND

Determining if a number is even or odd is another common task. While most programmers use the modulus operator for this:

```js
// Traditional
const oddOrEven = (n) => {
	if(n % 2 === 0) return "Even"
	
	return "Odd"
} 
```

There’s a faster, bitwise way to achieve the same result using the AND operator:
```js
const oddOrEven = (n) => {
	/**
	* n & 1 returns either 0 or 1
	* Since 0 or 1 is taken as false or true,
	* skipping the condition check
	*/
	if(n & 1) return "Odd"
	
	return "Even"
} 

/**
* Lets see how this work
* 
* Say you've a odd number - 21
* Binary representation of 21: 0010 0001
* Binary representation of 1 : 0000 0001
* 0010 0000 & 0000 0001      = 0000 0001 => 1
* if result is One then the given number is Odd
* else Even.
*/
```

This method is not only faster but also simpler at the machine level, as bitwise operations are generally quicker than arithmetic ones.

> Note : When I ran the benchmark, it didn't show much of a difference. (Python)

### Why Use These Bitwise Techniques?

You might wonder why you should bother with these bitwise techniques when more conventional methods work just fine. The answer lies in efficiency and understanding. Bitwise operations are faster because they operate directly on the binary representation of numbers. While the performance difference may be negligible in small-scale applications, in performance-critical code, such as in embedded systems or algorithms that must run on large datasets, these optimizations can make a significant impact.

Moreover, understanding bitwise operations and their applications can deepen your grasp of how computers perform basic operations. It’s a valuable skill for any programmer, helping you write more efficient code and better understand the low-level workings of your programs.

### Conclusion

Bitwise operations like XOR and AND might seem esoteric at first, but as we've seen, they can lead to elegant solutions for common problems. Whether you're swapping variables without a temporary placeholder or checking if a number is even or odd, these techniques can make your code more efficient and concise. So next time you're faced with one of these tasks, consider reaching for a bitwise operator—you might just find it’s the tool you need.

<p style="text-align: center;">fin.</p>
