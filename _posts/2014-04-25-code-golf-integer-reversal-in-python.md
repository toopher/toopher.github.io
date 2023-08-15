---
layout: post
category: 
tags: []
author: Seth
display_title: Python Code Golf - Reverse a 32b Integer
---

We were talking recently and I brought up bit operations. I said I
thought I had done bit operations more often in interviews than in years of
real coding. This led to a discussion about the merits of bit
operations, ultimately centering on how to reverse the bits in a 32b
integer. From there, we geeked out on how to write the shortest 32b
integer bit reversal.  Our approaches can roughly be classified as 
*bit manipulation*, *reduce*, *recursive lambdas*, and *string reversal*. 
It unfolded something like this:

It started with us simply trying to get a correct solution with Google
and our own ingenuity. A couple *bit manipulation* approaches came out: 
shifting with magical constants and shift-and-mask.

The [less-than-obvious bit twiddling solution from StackOverflow](http://stackoverflow.com/a/746203/553403):

    def reverse_bits(x):
      x = (((x & 0xaaaaaaaa) >> 1) | ((x & 0x55555555) << 1))
      x = (((x & 0xcccccccc) >> 2) | ((x & 0x33333333) << 2))
      x = (((x & 0xf0f0f0f0) >> 4) | ((x & 0x0f0f0f0f) << 4))
      x = (((x & 0xff00ff00) >> 8 ) | ((x & 0x00ff00ff) << 8))
      return((x >> 16) | (x << 16))

We ultimately decided this solution was cheating since we could not rederive it easily.
So, we dove into writing our own and came up with the more readable shift-and-mask:

    def reverse(input_, width=32):
      output = 0
      for i in xrange(1, width+1):
        mask = 1 << (width - i)
        bit = input_ & mask
        output = output | (bit << i - 1)
      return output
    
Then we started to get terse with this 116 character *reduce* solution:

    # We start to get tricky
    def bitrev(n):
        return reduce(lambda result, x: result + 
        (1 << x if n & (1 << (31-x)) else 0), xrange(0, 32), 0)

From here, golf began and the race was on. We can get under 100 characters, 
right? Yep. By shortening variables to a single-letter, and tweaking a bit 
more we got to a 72 character solution:

    b=lambda n:reduce(lambda a,x:a+(1<<x)*int(n&(1<<(31-x))and 1),range(32))

After a few more iterations, we're down to 60 characters:

    b=lambda n:reduce(lambda a,r:a|(n>>r&1)<<31-r,range(0,32),0)

"Hey, doesn't `range` default to starting at `0`?" you say. Yep, so we
shave off another couple characters:

    b=lambda n:reduce(lambda a,r:a|(n>>r&1)<<31-r,range(32),0)

Then, a new approach entered the competition: *recursive lambdas*. That
gets us down to 52 characters:

    b=lambda n,r=31:r!=-1 and(n>>r&1)<<31-r|b(n,r-1)or 0

A slightly different tack gets us to 49 characters:

    # WTH?! 32and is valid syntax? 
    # Yes, because Python variables cannot start with a number
    b=lambda n,r=0:r<32and(n>>r&1)<<31-r|b(n,r+1)or 0

And this gets whittled down to 45 characters:

    b=lambda n,r=0:r<32and(n>>r&1)<<31-r|b(n,r+1)

But why stop there? How about 42 characters:

    b=lambda n,r=0:n and(n&1)<<31-r|b(n/2,r+1)

Then *string reversals* enter the competition with a 42 character solution:

    b=lambda n:int(format(n,'#034b')[:1:-1],2)

We felt like 42 characters might be a lower bound, but we kept hacking
and got to 40 characters:

    b=lambda n:int(format(n,'032b')[::-1],2)

Since string reversals are kind of cheating, we got a 40 character
recursive lambda:

    b=lambda n,r=31:n and(n&1)<<r|b(n/2,r-1)

Tied at 40 characters we broke the tie with disassembling. The recursive
lambda came out larger than the string reversal and it was slower.
It seems standard library functions are robust.

Much later, an elegant 37 character string reversal solution surfaced:

    b=lambda n:int(bin(2**32+n)[:2:-1],2)

This approach proved faster than the string reversal using
`format`, although the disassembly was a hint longer.

We haven't found a shorter approach. Is 37 character the lower limit?
Perhaps, although we think the theoretical winner would import a minified 
library that contains a bit reversal method:

    import r;b=lambda n:r.r(n)

Shorter solutions are likely possible in other languages as several
characters are dedicated to converting Python integers to 32b (Python
helpfully converts `int`s to `bignum`s under the cover, so you don't
easily overflow, which is really nice until you specifically need a 4 byte
integer).

This ended up being a fun learning experience.  It was fascinating to see
how we all approached the problem and how we fed off one another's
optimizations. Long live integer bit reversals!

Do you see any optimizations? Which approach is most readable?

---

## Appendix

### Testing
To test, try these simple testcases:
 
    print b(0) == 0
    print b(1) == 2147483648
    print b(1909035918) == 1909035918
    print b(b(1)) == 1

### Disassembly output

    # Recursive Lambda
    1         0 LOAD_FAST                0 (n)
              3 JUMP_IF_FALSE_OR_POP    38
              6 LOAD_FAST                0 (n)
              9 LOAD_CONST               1 (1)
             12 BINARY_AND          
             13 LOAD_FAST                1 (r)
             16 BINARY_LSHIFT       
             17 LOAD_GLOBAL              0 (d)
             20 LOAD_FAST                0 (n)
             23 LOAD_CONST               2 (2)
             26 BINARY_DIVIDE       
             27 LOAD_FAST                1 (r)
             30 LOAD_CONST               1 (1)
             33 BINARY_SUBTRACT     
             34 CALL_FUNCTION            2
             37 BINARY_OR           
             38 RETURN_VALUE


    # String reversal with format
    1         0 LOAD_GLOBAL              0 (int)
              3 LOAD_GLOBAL              1 (format)
              6 LOAD_FAST                0 (n)
              9 LOAD_CONST               1 ('032b')
             12 CALL_FUNCTION            2
             15 LOAD_CONST               0 (None)
             18 LOAD_CONST               0 (None)
             21 LOAD_CONST               2 (-1)
             24 BUILD_SLICE              3
             27 BINARY_SUBSCR       
             28 LOAD_CONST               3 (2)
             31 CALL_FUNCTION            2
             34 RETURN_VALUE

    # String reversal with bin
    1         0 LOAD_GLOBAL              0 (int)
              3 LOAD_GLOBAL              1 (bin)
              6 LOAD_CONST               4 (4294967296)
              9 LOAD_FAST                0 (n)
             12 BINARY_ADD
             13 CALL_FUNCTION            1
             16 LOAD_CONST               0 (None)
             19 LOAD_CONST               1 (2)
             22 LOAD_CONST               3 (-1)
             25 BUILD_SLICE              3
             28 BINARY_SUBSCR
             29 LOAD_CONST               1 (2)
             32 CALL_FUNCTION            2
             35 RETURN_VALUE

### Timing results

    # Recursive lambda
    timeit b(b(1))
    100000 loops, best of 3: 11.7 us per loop

    # String reversal with format
    timeit b(b(1))
    100000 loops, best of 3: 2.95 us per loop
    
    # String reversal with bin
    timeit b(b(1))
    100000 loops, best of 3: 2.86 µs per loop
