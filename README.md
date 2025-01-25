# NAME

Crypt::Primes - Provable Prime Number Generator suitable for Cryptographic Applications.  

# SYNOPSIS

```perl
# generate a random, provable 512-bit prime.

use Crypt::Primes qw(maurer);
my $prime = maurer (Size => 512); 

# generate a random, provable 2048-bit prime and report 
# progress on console.  

my $another_prime = maurer ( 
                     Size => 2048, 
                     Verbosity => 1 
                    );           


# generate a random 1024-bit prime and a group 
# generator of Z*(n).

my $hash_ref = maurer ( 
                Size => 1024, 
                Generator => 1, 
                Verbosity => 1
               );
```

# WARNING 

The codebase is stable, but the API will most definitely change in a future
release.  

# DESCRIPTION

This module implements Ueli Maurer's algorithm for generating large
_provable_ primes and secure parameters for public-key cryptosystems.  The
generated primes are almost uniformly distributed over the set of primes of
the specified bitsize and expected time for generation is less than the time
required for generating a pseudo-prime of the same size with Miller-Rabin
tests.  Detailed description and running time analysis of the algorithm can
be found in Maurer's paper\[1\].

Crypt::Primes is a pure perl implementation.  It uses Math::Pari for
multiple precision integer arithmetic and number theoretic functions.
Random numbers are gathered with Crypt::Random, a perl interface to
/dev/u?random devices found on most modern Unix operating systems.

# FUNCTIONS

The following functions are available for import.  They must be explicitly
imported.

- **maurer(%params)**  

    Generates a prime number of the specified bitsize.  Takes a hash as
    parameter and returns a Math::Pari object (prime number) or a hash reference
    (prime number and generator) when group generator computation is requested.
    Following hash keys are understood:

- **Size**

    Bitsize of the required prime number. 

- **Verbosity** 

    Level of verbosity of progress reporting.  Report is printed on STDOUT.
    Level of 1 indicates normal, terse reporting.  Level of 2 prints lots of
    intermediate computations, useful for debugging.

- **Generator** 

    When Generator key is set to a non-zero value, a group generator of Z\*(n) is
    computed.  Group generators are required key material in public-key
    cryptosystems like Elgamal and Diffie-Hellman that are based on
    intractability of the discrete logarithm problem.  When this option is
    present, maurer() returns a hash reference that contains two keys, Prime and
    Generator.

- **Relprime**

    When set to 1, maurer() stores intermediate primes in a class array, and
    ensures they are not used during construction of primes in the future calls
    to maurer() with Reprime => 1.  This is used by rsaparams(). 

- **Intermediates**

    When set to 1, maurer() returns a hash reference that contains
    (corresponding to the key 'Intermediates') a reference to an array of
    intermediate primes generated.

- **Factors**

    When set to 1, maurer() returns a hash reference that contains
    (corresponding to the key 'Factors') a reference to an array of
    factors of p-1 where p is the prime generated, and also (corresponding
    to the key 'R') a divisor of p.

- **rsaparams(%params)**

    Generates two primes (p,q) and public exponent (e) of a RSA key pair. The
    key pair generated with this method is resistant to iterative encryption
    attack. See Appendix 2 of
    \[1\] for more information.

    rsaparams() takes the same arguments as maurer() with the exception of
    \`Generator' and \`Relprime'.  Size specifies the common bitsize of p an q.
    Returns a hash reference with keys p, q and e.

- **trialdiv($n,$limit)**

    Performs trial division on $n to ensure it's not divisible by any prime
    smaller than or equal to $limit.  The module maintains a lookup table of
    primes (from 2 to 65521) for this purpose.  If $limit is not provided, a
    suitable value is computed automatically.  trialdiv() is used by maurer() to
    weed out composite random numbers before performing computationally
    intensive modular exponentiation tests.  It is, however, documented should
    you need to use it directly.

# IMPLEMENTATION NOTE

This module implements a modified FastPrime, as described in \[1\], to
facilitate group generator computation.  (Refer to \[1\] and \[2\] for
description and pseudo-code of FastPrime).  The modification involves
introduction of an additional constraint on relative size r of q.  While
computing r, we ensure k \* r is always greater than maxfact, where maxfact
is the bitsize of the largest number we can factor easily.  This value
defaults to 140 bits.  As a result, R is always smaller than maxfact, which
allows us to get a complete factorization of 2Rq and use it to find a
generator of the cyclic group Z\*(2Rq).

# RUNNING TIME

Crypt::Primes generates 512-bit primes in 7 seconds (on average), and
1024-bit primes in 37 seconds (on average), on my PII 300 Mhz notebook.
There are no computational limits by design; primes up to 8192-bits were
generated to stress test the code.  For detailed runtime analysis see \[1\].

# SEE ALSO

largeprimes(1), Crypt::Random(3), Math::Pari(3)

# BIBLIOGRAPHY

- 1 Fast Generation of Prime Numbers and Secure Public-Key Cryptographic
Parameters, Ueli Maurer (1994). 
- 2 Corrections to Fast Generation of Prime Numbers and Secure Public-Key
Cryptographic Parameters, Ueli Maurer (1996). 
- 3 Handbook of Applied Cryptography by Menezes, Paul C. van Oorschot
and Scott Vanstone (1997).
- Documents 1 & 2 can be found under docs/ of the source distribution.

# AUTHOR

Vipul Ved Prakash, <mail@vipul.net>

# LICENSE 

Copyright (c) 1998-2001, Vipul Ved Prakash. All rights reserved. This code
is free software; you can redistribute it and/or modify it under the same
terms as Perl itself.

# TODO

Maurer's algorithm generates primes of progressively larger bitsize using
a recursive construction method. The algorithm enters recursion with a
prime number and bitsize of the next prime to be generated. (Bitsizes of
the intermediate primes are computed using a probability distribution that
ensures generated primes are sufficiently random.) This recursion can be
distributed over multiple machines, participating in a competitive
computation model, to achieve close to best running time of the algorithm.
Support for this will be implemented some day, possibly when the next
version of Penguin hits CPAN.
