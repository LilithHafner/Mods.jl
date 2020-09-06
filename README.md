# Mods

Modular arithmetic for Julia.


## Quick Overview
This module supports modular values and arithmetic. The moduli are integers (at least 2)
and the values are either integers or Gaussian integers.

An element of $\mathbb{Z}_N$ is entered as `Mod{N}(a)` and is of type `Mod{N}`
element of $\mathbb{Z}_N[i]$ is entered a `Mod{N}(a+b*im)` and is of type 
`GaussMod{N}`. Both types are fully interoperable with each other and with 
(ordinary) integers and Gaussian integers.

```julia
julia> a = Mod{17}(9); b = Mod{17}(10);

julia> a+b
Mod{17}(2)

julia> 2a
Mod{17}(1)

julia> a = Mod{17}(9-2im)
Mod{17}(9 + 15im)

julia> 2a
Mod{17}(1 + 13im)

julia> a'
Mod{17}(9 + 2im)
```


---




The old version did not have a function to recover the value and the
modulus of a `Mod` value `x`. The kludge was to use `x.val` and `x.mod`.
In this version, the functions `value(x)` and `modulus(x)` are provided.
[`x.val` still works (although it should be avoided), but `x.mod` does not.]

## New Feature in 1.2.0

We allow the value of a `Mod` to be a Gaussian integer!
Specifically, we define a `GaussMod{N}` type that should be fully interoperable 
with `Mod{N}`. However, there is no reason for users to explicitly use `GaussMod`; 
just use `Mod{N}` with a complex argument:
```julia
julia> z = Mod{13}(4-3im)
Mod{13}(4 + 10im)

julia> z^12
Mod{13}(1 + 0im)

julia> 1/z
Mod{13}(9 + 10im)

julia> a = Mod{13}(1+5*im)
Mod{13}(1 + 5im)

julia> a*a'
Mod{13}(0 + 0im)

julia> inv(a)
ERROR: Mod{13}(1 + 5im) is not invertible
```


---

[![Build Status](https://travis-ci.org/scheinerman/Mods.jl.svg?branch=master)](https://travis-ci.org/scheinerman/Mods.jl)

[![codecov.io](http://codecov.io/github/scheinerman/Mods.jl/coverage.svg?branch=master)](http://codecov.io/github/scheinerman/Mods.jl?branch=master)

## Basics

Objects of type `Mod` are a type of `Number`. In particular, `Mod{N}`
represents values in the set `{0,1,2,...,N-1}` with all operations
evaluated modulo `N`.

Construct a `Mod` object with `Mod{N}(val)`.  Both `val`
and `N` must be `Int` values.
```
julia> using Mods

julia> Mod{23}(4)
Mod{23}(4)

julia> Mod{23}(-1)
Mod{23}(22)

julia> Mod{12}(99)
Mod{12}(3)

julia> x = Mod(4,10)  # old style works, but please avoid
Mod{10}(4)
```

The smallest allowable modulus is 2:
```
julia> Mod{-4}(1)
ERROR: AssertionError: modulus must be at least 2
```

The functions `modulus` and `value` are used to recover the
relevant components of a `Mod` number
```
julia> x = Mod{10}(14)
Mod{10}(4)

julia> value(x)
4

julia> modulus(x)
10
```

With no arguments, `Mod{N}` creates a zero element of the given
modulus.
```
julia> Mod{17}()
Mod{17}(0)
```

The functions `zero`, `zeros`, `one`, and `ones` behave as for other
`Number` types:
```
julia> one(Mod{19})
Mod{19}(1)

julia> zeros(Mod{7},3,5)
3×5 Array{Mod{7},2}:
 Mod{7}(0)  Mod{7}(0)  Mod{7}(0)  Mod{7}(0)  Mod{7}(0)
 Mod{7}(0)  Mod{7}(0)  Mod{7}(0)  Mod{7}(0)  Mod{7}(0)
 Mod{7}(0)  Mod{7}(0)  Mod{7}(0)  Mod{7}(0)  Mod{7}(0)
```


## Operations

### The basic four

`Mod` objects can be added, subtracted, mulitplied, and divided with
one another. The two `Mod` operands must have the same modulus.
```
ulia> x = Mod{10}(8); y = Mod{10}(6);

julia> x+y
Mod{10}(4)

julia> y-x
Mod{10}(8)


julia> x*y
Mod{10}(8)

julia> Mod{10}(5) + Mod{11}(5)
ERROR: Cannot operate on two Mod objects with different moduli
```

Division can result in an error if the divisor is not invertible. A
`Mod` object `x` can be checked for invertibility using
`is_invertible(x)`. To find the inverse of `x` (assuming it exists)
use `inv(x)`.

```
julia> x = Mod{10}(8); y = Mod{10}(3);

julia> x/y
Mod{10}(6)

julia> y/x
ERROR: Mod{10}(8) is not invertible

julia> inv(y)
Mod{10}(7)
```

We also support unary minus.
```
julia> x = Mod{10}(3)
Mod{10}(3)

julia> -x
Mod{10}(7)
```

Note: The rational division operation `//` gives the same result
as ordinary division `/`.


#### Overflow safety

Integer operations on 64-bit numbers can give results requiring more than
64 bits. Fortunately, when working with modular numbers the results of
the operations are bounded by the modulus.
```
julia> N = 10^18                # this is a 60-bit number
1000000000000000000

julia> a = 10^15
1000000000000000

julia> a*a                      # We see that a*a overflows
5076944270305263616

julia> Mod{N}(a*a)              # this gives an incorrect answer
Mod{1000000000000000000}(76944270305263616)

julia> Mod{N}(a) * Mod{N}(a)    # but this is correct!
Mod{1000000000000000000}(0)
```


### Mixed Integer/Mod arithmetic

The basic four operations may also be performed between a `Mod` object
and an integer. The calculation proceeds as if the integer has the
same modulus as the `Mod` object.
```
julia> x = Mod{10}(3);

julia> x+9
Mod{10}(2)

julia> 4x
Mod{10}(2)

julia> 3-x
Mod{10}(0)

julia> x/7
Mod{10}(9)
```

### Rationals and Mods

The result of `Mod{N}(a//b)` is exactly
`Mod{N}(numerator(a)) / Mod{n}(denominator(b))`. This may equal
`Mod{N}(a)/Mod{N}(b)` if `a` and `b` are relatively prime to each other
and to `N`.

When a `Mod` and a `Rational` are operated with each other, the
`Rational` is first converted to a `Mod`, and then the operation
proceeds.

Bad things happen if the denominator and the modulus are not
relatively prime.





### Exponentiation

Use `x^k` to raise a `Mod` object `x` to an integer power `k`. If
`k` is zero, this always returns `Mod{m}(1)` where `m` is the modulus
of `x`. Negative exponentiation succeeds if and only if `x` is
invertible.
```
julia> x = Mod{100}(3)
Mod{100}(3)

julia> x^10
Mod{100}(49)

julia> x^-2
Mod{100}(89)

julia> x = Mod{100}(5)
Mod{100}(5)

julia> x^-3
ERROR: Mod{100}(5) is not invertible

julia> Mod{10}(0)^0
Mod{10}(1)
```


### Random numbers

The standard `rand` function returns a (pseudo)random `Mod` value. In
particular, `rand(Mod{N})` returns a value in `{0,1,...,N-1}`,
each with probability `1/N`.
```
julia> rand(Mod{20})
Mod{20}(16)
```

Random vectors and matrices can be created using `rand(Mod{N})(n)` and
`rand(Mod{N},n,m)`.



### Equality and hashing

Two `Mod` objects can be compared for equality with either `==` or
`isequal`.
```
julia> Mod{10}(3) == Mod{11}(3)
false

julia> Mod{10}(3) == Mod{10}(-7)
true
```

We can also compare `Mod` objects with integer objects:
```
julia> Mod{10}(3) == -7
true

julia> Mod{10}(3) == 7
false
```


We also define `hash` for `Mod` objects so they can be stored in sets
and used as keys in a dictionary.
```
julia> A = Set{Mod}()
Set{Mod} with 0 elements

julia> push!(A,Mod{10}(3))
Set{Mod} with 1 element:
  Mod{10}(3)

julia> push!(A,Mod{11}(3))
Set{Mod} with 2 elements:
  Mod{10}(3)
  Mod{11}(3)
```

The container can be narrowed to a particular `Mod` and then
we have this:
```
julia> B = Set{Mod{10}}()
Set{Mod{10}} with 0 elements

julia> push!(B,0)
Set{Mod{10}} with 1 element:
  Mod{10}(0)

julia> push!(B,11)
Set{Mod{10}} with 2 elements:
  Mod{10}(0)
  Mod{10}(1)

julia> push!(B,Mod{11}(3))
ERROR: MethodError: no method matching Mod{10}(::Mod{11})
```
The last input fails because `B` was defined to be a `Set`
that holds `Mod{10}` objects only.


### Chinese Remainder Theorem calculations

The Chinese Remainder Theorem gives a solution to the following
problem. Given integers `a, b, m, n` with `gcd(m,n)==1` find an
integer `x` such that `mod(x,m)==mod(a,m)` and
`mod(x,n)==mod(b,n)`. We provide the `CRT` function to solve this
problem as illustrated here with `a=3`, `m=10`, `b=5`, and `n=17`:

```
julia> s = Mod{10}(3); t = Mod{17}(5);

julia> CRT(s,t)
Mod{170}(73)
```

We find that `mod(73,10)` equals `3` and `mod(73,17)` equals `5` as
required. The answer is reported as `Mod{170}(73)` because any value of
`x` congruent to 73 modulo 170 is a solution.

The `CRT` function can be applied to any number of arguments so long
as their moduli are pairwise relatively prime.
