# Bit Tricks



## detecting integer constant expressions in macros

https://lkml.org/lkml/2018/3/20/805

```

From	"Uecker, Martin" <>
Subject	detecting integer constant expressions in macros
Date	Tue, 20 Mar 2018 22:13:35 +0000
	

Hi Linus,

here is an idea:

a test for integer constant expressions which returns an
integer constant expression itself which should be suitable
for passing to __builtin_choose_expr might be:

#define ICE_P(x) (sizeof(int) == sizeof(*(1 ? ((void*)((x) * 0l)) :
(int*)1)))

This also does not evaluate x itself on gcc although this is
not guaranteed by the standard. (And I haven't tried any older
gcc.)

Best,
Martin
```

https://lkml.org/lkml/2018/3/21/817

```
From	Linus Torvalds <>
Date	Wed, 21 Mar 2018 15:27:49 -0700
Subject	Re: detecting integer constant expressions in macros
	

On Wed, Mar 21, 2018 at 2:10 PM, Rasmus Villemoes
<linux@rasmusvillemoes.dk> wrote:
>>
>> Oh, I think it's guaranteed by the standard that 'sizeof()' doesn't
>> evaluate the argument value, only the type.
>
> Even if there's some hypothetical scenario where x ends up containing
> something of variably-modified type, could one just use 0? instead of 1?
> . That doesn't affect the type of the whole expression.

Fair enough.

> I'm not really worthy of commenting or trying to improve on this, but I
> think it's important that 'git blame' will forever give Martin the
> credit for this

You're saying "please don't ever blame me".

Gotcha ;)

> two minor suggestions:
>
> - Cast (x) to (long) to allow x to have type u64 on a 32-bit platform
> without giving "warning: cast to pointer from integer of different
> size". That also allows x to have pointer type (it's theoretically
> useful to take max() of two pointers into an array), and not so
> important for the kernel, floating point type.

Fair enough.

> - Maybe use (int*)4 to avoid the compiler complaining about creating a
> non-aligned pointer.

Yes.

> I tried putting the below ugly test code through various gcc versions.
> In no cases could I get different optimization options to generate
> different results. I also couldn't get -Wall -Wextra to produce any
> warnings that wasn't just due to the silly input (i.e., yes, I know I
> have a comma expression with no side effects....). Most importantly,
> ICE_P was always accepted as an ICE itself. Everything from 4.6 onwards
> gives the same and expected output, namely 1 in the first four lines, 0
> otherwise. gcc 4.4 instead produces this:

Ok, so since we're going to get rid of 4.4 support anyway because of
our requirement of "asm goto", and since 4.5 was apparently never
shipped in any distros, I think the minimum compiler version we care
about is gcc-4.6.

So the above sounds fine, and I won't worry about the fact that
gcc-4.4 apparently does expression simplifications early.

> So it seems to accept/treat (x)*0l as NULL as long as x has no
> side-effects - where a function call is treated as having side effects
> unless the function is declared with the const attribute,

That's actually _largely_ the main thing we care about anyway in most
places, so even the 4.4 behavior is probably acceptable in practice.

For example, for "min/max()", it's true that constants are what we're
looking at, but at the same time, the expression simplification really
only cares about "no side effects".

In some other cases, we really do want to see constants, though (the
example I mentioned earlier with build-time simplification of bswap
comes to mind).

> comma-expressions are hard-coded as having side-effects [and what else
> would they be good for, so that's not totally unreasonable...]

Actually, I think one of our more common uses of a comma expression is
not for side effects, but because of things like type warnings.

I guess that is a "side effect" too, although it's really a build-time
one, not a run-time one.

> and statement expressions have side effects if they have more than one
> statement.

Ok, that's just odd semantics.

But I guess it's actually a very natural result of "do obvious
simplifications early", ie one of those expression simplifications
that gcc-4.4 does on the early parse-tree level is to just turn a
single-statement statement expression directly into just the
expression.

So I think the gcc-4.4 behavior is "understandable", even if it
happens to be very much non-standard, and happens to break the "x*0"
trick that Martin's masterpiece uses.

If we knew exactly what gcc ends up simplifying, we might be able to
work around it. For example, is the front-end tree simplification
smart enough to realize that

   1+(unsigned long)(x)+~(unsigned long)(x)

is also 0?

For a true *constant*, that's very easy for a compiler to see (just
evaluate the damn thing). For an expression that isn't constant, it's
a bit less obvious, and depends on knowing that "~x+1" is the same as
"-x" in 2's complement.

Of course, compilers *do* have that logic, but it's often deeper down
than at parse-tree time.

Doing some simple testing on godbolt, I see that gcc-4.1 actually
accepts this program:

  unsigned long x;
  void *ptr= (void *)(x-x);

and clearly considers "(void *)(x-x)" to be NULL.

But if you write it as

  unsigned long x;
  void *ptr= (void *)(1+x+~x);

it fails with 4.1.

But then it succeeds again with 4.4 and sees that it's just an odd way
to write 0.

Damn clever compiler writers to hell, they clearly started doing that
simplification early.

There are other weird ways to calculate 0, though. We're not *just*
limited to "*0" and trivial bit tricks. I bet we could come up with
something that always evaluates to zero, but is too hard for gcc to
turn into zero symbolically.

For example, gcc-4.4 does not realize that

  #define rightmost(x) ((x)&-(x))
  #define lowmask(x) (rightmost(x)-1)
  unsigned long x;
  void *ptr= (void *)(x&lowmask(x));

is a *really* weird way to write a constant zero. But if 'x' really is
a constant, it's immediately obvious because it just evaluates the
expression and get zero.

So if you use "#define x 0x1234", it turns into NULL again.

So we *could* make this work even for gcc-4.4.

          Linus
```

https://lore.kernel.org/lkml/dc86fdf7-3202-e836-6f71-af1e2458b105@rasmusvillemoes.dk/

```
From: Rasmus Villemoes <linux@rasmusvillemoes.dk>

...
Actually, since the first operand (the condition) is a non-zero number,
the _value_ of the whole expression is the value of the _second_
operand, but with a _type_ determined by the above rules. So the whole
ternary operator evalutes to either

  (void *)((void *)((long)(x) * 0l))

or

  (int *)((void *)((long)(x) * 0l))
```
