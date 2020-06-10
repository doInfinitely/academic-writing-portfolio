# LeetCode #29: Divide Two Integers
Given two integers *dividend* and *divisor*, divide two integers without using multiplication, division and mod operator.

I choose to address this problem because it was a medium difficulty challenge with one of the lowest rates of acceptance, 16.2%.

This  challenge can be seen as an exercise in understanding division via implementation. I proceeded by reconstructing certain “higher” “disallowed operations, in order to reconstruct division.

The first of these is multiplication by -1:

```py
    def flipsign(self, x):
        return 0-x
```

Next, multiplication by 10. We perform this via a digit shift:

```py
    def mult10(self, x):
        return int(str(x) + "0")
```

An approximation of division by 10 and the inverse of the previous operation: (we call it an approximation because it only gives the exact result when operating on numbers with a trailing zero):

```py
    def trunc(self, x):
        return int(str(x)[0:-1])
```

Next, we find integer log base 10. This operation, for inputs x and y, finds the largest natural number z such that x >= y * 10^z.

We also construct a method for convenience that finds y * 10^z by repeatedly calling the “multiply by 10” method.

The last helper method, *divpos(x, y)*, repeatedly subtracts y from x until x is less than zero, counting the whole number of times that y  fits in x.

```py
    def divpos(self, x, y):
        dividend = x
        divisor = y
        quotient = 0
        while dividend > 0:
            dividend -= divisor
            if dividend >= 0:
                quotient += 1
        return quotient
```

This method by itself is enough to solve the problem of integer division, however, using repeated subtraction without using the other helper methods would be O(n), where n is the dividend.  We can intuitively observe that linear complexity for this operation is pretty bad. For instance, we can instantly see that 2,000,000, divided by 1 is 2,000,000, but *divpos* actually has to count up to 2,000,000 to discover this.

Finally, we combine these methods to tackle the problem directly. In the beginning of *divide* we store the absolute value of minimum integer to handle the special case of division overflow. We also initialize the variables quotient and sign.

```py
    def divide(self, dividend, divisor):
        MAX = 2147483647
        quotient = 0
        sign = 1
        dend = dividend
        sor = divisor
```

Then we handle the cases where one or more of the inputs is negative. We reduce the problem to positive integer division and then at the end of the method use the *flipsign*  method and the value of the *sign* variable to correct the sign of the output.

```py
        if dend < 0:
            dend = self.flipsign(dend)
            sign = self.flipsign(sign)
        sor = divisor
        if sor < 0:
            sor = self.flipsign(sor)
            sign = self.flipsign(sign)
```

Next, we have
```py
while True:
            exp = self.findexp(dend, sor)
            quo = self.divpos(dend, self.multpow10(sor,exp))
            quotient += self.multpow10(quo,exp)
            for i in range(quo):
                dend -= self.multpow10(sor,exp)
            if exp == 0:
                break
```

In this part of the code, we multiply the divisor by 10 as much as possible such that it does not exceed the dividend, which is an O(log n) operation, where n is the size of the dividend. Then we repeatedly subtract this multiplied divisor from the dividend. We know this process can only be repeated a maximum of nine times (since if it can be done 10 or more times, the divisor would have been multiplied further), thus this step has constant time complexity. This process leaves a quotient and a remainder that is bounded to be lower than that the multiplied divisor. We treat this remainder as a new dividend and repeat the original process again to produce a new remainder and so on, each time adding the new quotient to the the previously computed ones. The algorithm terminates when the dividend is less than 10 times the divisor (i.e., the exponent, “exp,” is zero).
And finally, we integrate the sign information and handle the overflow edge cases.

In the worst case, all together this algorithm requires O(log^2 n) steps, where n is the numerical size of the dividend.