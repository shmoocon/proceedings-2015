# Eliminating Timing Side-channels. A Tutorial

## Abstract

The traditional model of an attacker against a cryptographic primitive sees (and potentially controls) inputs and outputs of the computation. Side-channel attacks go beyond this model. The attacker now also sees some "leakage" of the internal state of the cryptographic computation. One class of leakage is timing: If the time taken by a computation depends on secret data, the attacker can measure time and obtain information about this secret data.  

The timing side channel is different than other side channels (such as power consumption or electromagnetic radiation) because it can be exploited remotely and without any specialized hardware or manual interaction.  It is also different because it is now well understood how to fully eliminate timing leakage. This tutorial explains how to write *constant-time* software, i.e., software that does not leak any secret information through timing.

## 1. Introduction

The last few decades of research in cryptography have produced various primitives and protocols that are widely accepted as secure. Examples are AES in CBC mode together with HMAC-SHA256 for authenticated encryption, RSA-2048 for public key encryption and ECDSA on the secp256k1 curve as used in Bitcoin for cryptographic signatures.  However, many *implementations* of those primitives and protocols have shown to be highly insecure as demonstrated by various devastating attacks. In 2006, Osvik, Shamir, and Tromer presented an attack that recovered the AES-256 key of Linux' dmcrypt hard-disk encryption in just 65ms [OST06]. The "Lucky-13" attack by AlFardan and Paterson recovered plaintext of CBC-mode encryptions in all common SSL/TLS implementations [AP13]. In 2014, Yarom and Falkner were able to recover on average 96.7\% of the bits of the RSA-2048 private key in GnuPG 1.4.13 by "observing a single signature or decryption round" [YF14]. Finally, the FLUSH+RELOAD attack by Benger, van de Pol,
Smart, Yarom, achieves a "reasonable level of success in recovering the secret
key" for OpenSSL ECDSA using secp256k1 "with as little as 200 signatures"
[BPSY14].

All of these attacks are so-called *timing attacks*. The general idea is quite
simple: If secret data influences the time taken by an implementation for
computations involving this secret data, then an attacker can measure the time
and deduce information about the secret data. This tutorial does not describe
the attack techniques in detail, but instead explains how to systematically
eliminate timing side channels from software. 

## 2. Secret branch conditions

The most obvious source for data-dependent timing variation in software are
conditional branches. Consider the following piece of code:

    if(secret)
    {
      do_A();
    }
    else
    {
      do_B();
    }

Obviously, if `do_A()` and `do_B()` take a different amount of time, this piece
of code will leak information about `secret` through timing. What is less
obvious is that this piece of code also leaks information about `secret` if
`do_A()` and `do_B()` take the same amount of time. One reason is that almost
all modern CPUs perform branch prediction. A CPU essentially guesses if `secret`
is 0 or 1 and fill the pipeline with the instructions that correspond to this
guess. A correct guess will result in faster execution of the code, a wrong
guess requires the pipeline to be flushed, which costs extra cycles. The other
reason are instruction caches: if instructions of `do_A()` are in cache and
instructions of `do_B()` are not, then fetching instructions of `do_A()` is
faster than fetching instruction of `do_B()`. See also Section 3.

Eliminating this kind of side channel requires to perform both computations,
`do_A()` and `do_B()` and then conditionally copying the result. Again, this
conditional copy must not be done through a branch, but through arithmetic. The
following piece of code is an example of a constant-time conditional move of a
32-bit integer from one memory location to another memory location, depending on
one secret bit. 

    /* secret has to be either 0 or 1 */
    void cmov(uint32_t *r, const uint32_t *a, uint32_t secret)
    {
      uint32_t t;
    
      secret = -secret; /* Now secret is either 0 or 0xffffffff */
      t = (*r ^ *a) & secret;
      *r ^= t;
    }

Note that this arithmetic approach for a conditional move can be
generalized to arbitrary data types thus be used to eliminate any conditional
branch.


## 3. Secret addresses

Secret branch conditions are only one of two major sources of input-dependent
timing variability in software; the other one is memory access. The easiest way
to understand why this is the case is to consider loads from cached memory.
Loading from cache (cache hit) is much faster than loading from main memory
(cache miss). The most famous examples of cache-timing attacks are attacks
against the Advanced Encryption Standard (AES). The original proposal by Daemen
and Rijmen already describes an implementation technique for AES, which is
making heavy use of secretly-indexed lookups from four tables of 1KB each. The
most basic timing attack against AES implementations that follows this apprach
goes as follows: After AES has processed some data, all tables are very likely
to be in cache. An attacker running code on the same CPU now loads some data and
thereby replaces some entries of the AES tables in cache with his own data.
Afterwards AES proceeds for a while and, depending on secret data, may again
replace some of the attackers data with table entries. When it is the attacker's
turn again, he loads his data and measures the time required for those loads.
If the loads are fast, his data was still in cache and AES has thus not accessed
the table entries that correspond to those cache lines; if it is slow, AES has
accessed those cache lines. 

Looking only at this basic attack one might conclude that secret addresses are
not a problem, as long as only the position within a cache line is secret but
the cache line is determined only from public information. This is not true as
pointed out by Bernstein in 2005 [Ber05] and Osvik, Shamir and Tromer in 2006
[OST06] and as demonstrated in principle by Bernstein and Schwabe in 2013
[BS13]. The reason is that the timing difference between a cache hit and a cache
miss is not the only way how addresses in load or store instructions influence
timing, other effects like failed store-to-load forwarding or cache-bank
conflicts influence timing depending on positions *within* a cache line; to
fully eliminate timing leakage, software must not access memory at
secret-data-dependent addresses. 

The general technique to rewrite table lookups without leaking timing
information is to load all elements and use conditional moves to write the
requested element to the output. For a table containing `TABLE_LENGTH` entries
of type `uint32_t`, this looks as follows:

    uint32_t table[TABLE_LENGTH];

    uint32_t lookup(size_t pos)
    {
      size_t i;
      int b;
      uint32_t r = table[0];
      for(i=1;i<TABLE_LENGTH;i++)
      {
        b = isequal(i, pos);
        cmov(&r, &table[i], b);
      }
      return r;
    }

Note that this code is using a function `isequal` for constant-time comparison
instead of a simple `==` operator. The reason is that compilers may use
conditional branches when facing `==`. A general approach for constant-time
equality comparison in C looks as follows; note that it can easily be adapted to
other data types by changing the types of inputs and the `sizeof` argument:

    int isequal(uint32 a, uint32 b)
    {
      size_t i; uint32 r = 0;
      unsigned char *ta = (unsigned char *)&a;
      unsigned char *tb = (unsigned char *)&b;
      for(i=0;i<sizeof(uint32);i++)
      {
        r |= (ta[i] ^ tb[i]);
      }
      r = (-r) >> 31;
      return (int)(1-r);
    }

Obviously, even such an xor-based test for equality is not a *guarantee* that
compilers do not introduce funny optimizations that violate constant-time
behavior; however, for current compilers this code seems to be safe. 

For the case of AES, using this general approach for constant-time lookups
results in horrible performance. Implementations of AES that are at the same
time fast and secure use either bitslicing (see, for example [KS09]) or
vector-permute instructions as described in [Ham09]. Both techniques are rather
complex and do not offer good performance across platforms. This is one reason
that Intel, AMD, and ARM decided to include hardware support for AES in their
recent lines of processors.

## 4. Variable-time arithmetic

Secret branch conditions and secret addresses are the two major sources of
timing variation, but some processors expose a third source for potential timing
leakage: variable-time arithmetic instructions, i.e., arithmetic instructions
that take a different amount of time depending on their inputs. Examples include
`DIV`, `IDIV`, and `FDIV` on pretty much all Intel/AMD CPUs, various math
instructions like `FSIN`, `FCOS` etc. on Intel and AMD CPUs, and multiplication
instructions like `MUL`, `MULHW`, `MULHWU` on many PowerPC CPUs or `UMULL`,
`SMULL`, `UMLAL`, and `SMLAL` on ARM Cortex-M3.
The easiest way to eliminate timing side channels stemming from those
instructions is not to use these instructions. For software written in C or
other compiled languages this requires some understanding of what compilers do
to the source code; it is obviously much easier to avoid those instructions when
writing software directly in assembly.


## References

[AP13]   AlFardan, Paterson, 2013. Lucky Thirteen: Breaking the TLS and DTLS Record Protocols.    
         IEEE Symposium on Security and Privacy,  IEEE Computer Society (2013), pp 526--540.    
         [http://www.isg.rhul.ac.uk/tls/Lucky13.html](http://www.isg.rhul.ac.uk/tls/Lucky13.html)

[BPSY14] Naomi Benger, Joop van de Pol, Nigel P. Smart, Yuval Yarom, 2014. "Ooh Aah... Just a Little Bit": A small amount of side channel can go a long way.    
         Cryptographic Hardware and Embedded Systems -- CHES 2014, LNCS vol. 8731, Springer (2014), pp 75--92.    
         [http://eprint.iacr.org/2014/161/](http://eprint.iacr.org/2014/161/)

[Ber05]  Daniel J. Bernstein, 2005. Cache-timing attacks on AES.    
         [http://cr.yp.to/papers.html#cachetiming](http://cr.yp.to/papers.html#cachetiming)

[BS13]   Daniel J. Bernstein, Peter Schwabe, 2013. A word of warning. CHES 2013 rump-session talk.    
         [https://cryptojedi.org/peter/data/chesrump-20130822.pdf](https://cryptojedi.org/peter/data/chesrump-20130822.pdf)
         [software]({https://cryptojedi.org/peter/data/cacheline.tar.bz2)

[Ham09]  Michael Hamburg, 2009: Accelerating AES with Vector Permute Instructions.    
         Cryptographic Hardware and Embedded Systems -- CHES 2009, LNCS vol. 5745, Springer (2009), pp 1--17.    
         [http://mikehamburg.com/papers/vector_aes/vector_aes.pdf](http://mikehamburg.com/papers/vector_aes/vector_aes.pdf)

[KS09]   Emilia Käsper, Peter Schwabe. Faster and Timing-Attack Resistant AES-GCM.    
         Cryptographic Hardware and Embedded Systems -- CHES 2009, LNCS vol. 5745, Springer (2009), pp 1--17.    
         [http://cryptojedi.org/papers/#aesbs](http://cryptojedi.org/papers/#aesbs)

[OST06]  Dag-Arne Osvik, Adi Shamir, Eran Tromer, 2006. Cache Attacks and Countermeasures: the Case of AES.
         Topics in Cryptology -- CT-RSA 2006, LNCS vol. 3860, Springer (2006), pp 1--20.    
         [http://eprint.iacr.org/2005/271/](http://eprint.iacr.org/2005/271/)

[YF14]   Yuval Yarom, Katrina Falkner, 2014. FLUSH+RELOAD: a High Resolution, Low Noise, L3 Cache Side-Channel Attack.    
         Proceedings of the 23rd USENIX Security Symposium, USENIX Association (2014), pp 719--732.    
         [http://eprint.iacr.org/2013/448/](http://eprint.iacr.org/2013/448/)

Primary Author Name: Peter Schwabe
Primary Author Affiliation: Radboud University, Nijmegen, The Netherlands
Primary Author Email: peter@cryptojedi.org
Keywords/Tags: Timing attacks, constant-time software, secret branches, secret addresses
