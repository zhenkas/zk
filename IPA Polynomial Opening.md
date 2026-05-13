# IPA Polynomial Opening (ECC Version)

## 1. Goal

Given polynomial:

    F(X) = a0 + a1 X + ... + a(N-1) X^(N-1)

prove:

    F(x) = y

with:
- proof size O(log N)
- prover work O(N)
- verifier work:
    - O(N) with arbitrary ECC generators
    - potentially lower with additional generator structure

This construction:
- does not require pairings
- does not require trusted setup
- works over arbitrary elliptic curves
- is an inner product argument specialized to polynomial evaluation
- as written, does not provide zero knowledge

Assume:

    N = 2^k

If necessary, coefficient vector is padded with zeros.

Let:
- ai, x, y ∈ F
- Gi, U ∈ G

where:
- F is the scalar field
- G is the elliptic curve group

Soundness requires:
- generators Gi are independent
- U is independent from all Gi
- no nontrivial linear relations are known among generators

Generators may be deterministically derived via hash-to-curve with domain separation so that no party knows discrete-log relations among them.

All Fiat-Shamir challenges ui must satisfy:

    ui != 0

All scalar arithmetic is performed in field F.


## Polynomial as Inner Product

Define coefficient vector:

    a = (a0, a1, ..., a(N-1))

Define evaluation vector:

    b(x) = (1, x, x^2, ..., x^(N-1))

Then:

    F(x) = <a, b>

So proving:

    F(x) = y

is equivalent to proving:

    <a, b> = y

The claimed evaluation y is public.


## Challenge Point Binding

This construction assumes the evaluation point x is bound to the commitment.

The prover first commits to the polynomial coefficients:

    C = Σ(ai * Gi)

Then x is derived from the commitment and transcript:

    x = H_to_field(domain, C, ctx...)

or is otherwise fixed before the prover can choose C.

Thus the prover cannot adapt C after learning x.

If the verifier receives x explicitly, the verifier must additionally check:

    x == H_to_field(domain, C, ctx...)


## 2. Setup

Public generators:

    G = (G0, G1, ..., G(N-1))

Introduce independent generator:

    U


## 3. Commitment

Prover computes commitment:

    C = Σ(ai * Gi)


## 4. Open

## Initial Proof State

Prover constructs:

    P0 = C + yU

Equivalently:

    P0 = <a, G> + <a, b> U

The recursive protocol preserves the invariant:

    P = <a, G> + <a, b> U


## Recursive IPA Folding

At each round split vectors into halves:

    a = (aL, aR)

    b = (bL, bR)

    G = (GL, GR)


## Cross Terms

Prover computes:

    L = <aL, GR> + <aL, bR> U

    R = <aR, GL> + <aR, bL> U

and sends:

    L, R


## Fiat-Shamir Challenge

In round i, define:

    u = ui

where the challenge is derived as:

    ui = H(domain, transcript, Li, Ri, ctx...)

where:
- H is a cryptographic hash function
- transcript includes prior proof state
- encodings are canonical
- domain provides protocol separation
- ui != 0

The Fiat-Shamir transcript should bind:
- domain separator
- curve/group identifiers
- N
- C
- x
- y
- U
- generator derivation or generator commitment
- all prior transcript elements
- all previous L/R pairs


## Folding

Fold vectors:

    a' = u aL + u^(-1) aR

    b' = u^(-1) bL + u bR

    G' = u^(-1) GL + u GR

Update proof state:

    P' = P + u^2 L + u^(-2) R

Expanded inner product:

    <a', b'> = <aL, bL> + u^2 <aL, bR> + u^(-2) <aR, bL> + <aR, bR>

Since:

    <a, b> = <aL, bL> + <aR, bR>

then:

    <a', b'> = <a, b> + u^2 <aL, bR> + u^(-2) <aR, bL>

Thus the inner product itself is not preserved.

However, the recursive update preserves the group invariant:

    P' = <a', G'> + <a', b'> U

Each round compresses two coordinates into one using verifier challenge u.

Repeat recursively.

After:

    log2(N)

rounds:

    a -> a*

    b -> b*

    G -> G*

The recursively accumulated commitment state becomes:

    P* = C + yU + Σ(ui^2 Li + ui^(-2) Ri)

Final relation:

    P* = a* G* + (a* b*) U

where:
- a* is the final folded coefficient scalar
- b* is the final folded evaluation scalar
- G* is the final folded generator point

Since vectors now have length 1:

    <a*, b*> = a* b*

After recursion, prover sends final scalar:

    a*


## Final Proof

Prover sends:

    Proof =
        C,
        y,
        (L1, R1),
        ...
        (LlogN, RlogN),
        a*


## 5. Verify

Verifier knows:
- generators Gi
- generator U

Verifier receives:
- commitment C
- claimed evaluation y
- recursive proof elements (Li, Ri)
- final scalar a*

Verifier derives:

    x = H_to_field(domain, C, ctx...)


## Reconstruct Transcript Challenges

Verifier computes:

    ui = H(domain, transcript, Li, Ri, ctx...)


## Recursive Reconstruction

Using the same folding rules as prover, verifier recursively updates:


Commitment state:

    P' = P + u^2 L + u^(-2) R

starting from:

    P0 = C + yU

After all recursive rounds, the accumulated folded commitment state is:

    P* = C + yU + Σ(ui^2 Li + ui^(-2) Ri)


Evaluation vector:

Starting from:

    b = (1, x, x^2, ..., x^(N-1))

verifier recursively folds:

    b' = u^(-1) bL + u bR

until final scalar:

    b*


Generator vector:

Starting from public generators:

    G = (G0, G1, ..., G(N-1))

verifier recursively folds:

    G' = u^(-1) GL + u GR

until final generator:

    G*

Equivalently:

    G* = Σ(wi * Gi)

where weights wi depend entirely on transcript challenges.


## Final Verification

Verifier checks:

    P* == a* G* + (a* b*) U

If equality holds, verifier accepts.


## 6. Security Statement

Under the discrete logarithm assumption in G, modeled in the random oracle model for Fiat-Shamir, and assuming generators Gi, U are sampled independently with no known linear relations, the protocol is a computationally sound argument for the existence of coefficients (a) satisfying:

    C = <a, G>

and:

    <a, b(x)> = y

where x is fixed before the prover commits to C.


## 7. Summary

## Proof Size

    2 log2(N)

group elements, plus:
- commitment C
- evaluation y
- final scalar a*


## Complexity

Prover:

    O(N)

because recursive work forms geometric series:

    N + N/2 + N/4 + ...

Verifier with arbitrary ECC generators:

    O(N)

because verifier must compute:

    G* = Σ(wi * Gi)

which is an N-sized multiscalar multiplication.

Computing b* is also O(N) by straightforward recursive folding, though the dominant verifier cost is typically the generator MSM.

Additional generator structure or preprocessing may reduce verifier complexity.


## Properties

Advantages:
- no pairings
- no trusted setup
- logarithmic proof size
- arbitrary elliptic curves supported
- transparent construction

Tradeoffs:
- verifier remains O(N) with arbitrary generators
- unlike KZG, verifier reconstructs folded generator state
- as written, protocol is not zero knowledge

Zero knowledge can be added by introducing random blinding terms, as done in standard IPA-based systems such as Bulletproofs and Halo.
