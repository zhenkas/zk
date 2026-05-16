# RLWE-Based Polynomial Evaluation Proof

## Negacyclic Fourier-Syndrome Commitment Version

## Status

This document describes a research-oriented polynomial evaluation proof.

The goal is to prove statements of the form:

$$
F(x_i)=y_i
$$

for multiple public points $x_i$, while keeping the polynomial $F$ hidden.

The construction uses an RLWE-style commitment:

$$
V = A F + E
$$

but modifies the public polynomial $A$ so that it has random zero positions in the Fourier/NTT domain.

This makes the commitment binding even when $F$ is not bounded, because the bounded error $E$ is forced to satisfy public Fourier-syndrome constraints.

This document is not a formal security proof. It is a design sketch and security argument.

---

# 1. Goal

The prover has a private polynomial:

$$
F \in R_q
$$

and wants to prove claims:

$$
F(x_1)=y_1,\quad F(x_2)=y_2,\quad \dots,\quad F(x_m)=y_m
$$

without revealing $F$.

The verifier should learn only the claimed evaluations $(x_i,y_i)$, not the full polynomial.

---

# 2. Ring

We work in the negacyclic ring:

$$
R_q = \mathbb{F}_q[X]/(X^N+1)
$$

where:

- $N$ is a power of two
- $q$ supports a negacyclic NTT
- multiplication is modulo $X^N+1$

Thus:

$$
X^N=-1
$$

and multiplication in $R_q$ is negacyclic convolution.

---

# 3. Public Parameters

The public parameters include:

- modulus $q$
- degree parameter $N$
- error bound $B$
- public polynomial $A\in R_q$
- random Fourier-zero set $Z\subseteq\{0,\dots,N-1\}$

The polynomial $A$ is generated so that:

$$
\widehat A_j=0 \quad \forall j\in Z
$$

where $\widehat A$ denotes the negacyclic NTT of $A$.

For frequencies outside $Z$, $\widehat A_j$ should be nonzero and random-looking.

The set $Z$ should be sampled randomly and fixed globally. Structured choices such as all odd frequencies or arithmetic progressions must be avoided.

---

# 4. Commitment to the Polynomial

To commit to $F$, the prover samples a bounded error polynomial:

$$
E = \sum_{i=0}^{N-1} e_i X^i
$$

with:

$$
|e_i|\le B
$$

and computes:

$$
V = AF + E \in R_q
$$

The commitment is:

$${Com}(F;E)=V$$

The polynomial $F$ is hidden by the RLWE-style masking term $AF+E$.

The polynomial $E$ is kept private.

---

# 5. Why Bounds on F Are Not Required

If $A$ were invertible in $R_q$, then for any chosen bounded $E$, one could solve:

$$
F=A^{-1}(V-E)
$$

So a commitment with unrestricted $F$ would not be binding.

The Fourier-zero construction prevents this.

In the NTT domain:

$$
\widehat V_j = \widehat A_j\widehat F_j + \widehat E_j
$$

For every zero frequency $j\in Z$:

$$
\widehat A_j=0
$$

so:

$$
\widehat V_j=\widehat E_j
$$

Therefore the public commitment $V$ fixes a Fourier syndrome of the bounded error $E$:

$$
\widehat E_Z=\widehat V_Z
$$

A malicious prover cannot freely choose arbitrary bounded $E$. They must find a bounded polynomial matching the public syndrome.

This is the mechanism that allows $F$ to remain unrestricted.

---

# 6. Binding Condition

Suppose the same commitment $V$ can be opened to two different polynomials:

$$
V = AF + E = AF' + E'
$$

Subtract:

$$
A(F-F') = E'-E
$$

Let:

$$
D = E'-E
$$

Since both $E$ and $E'$ are bounded by $B$, we have:

$$
|D_i|\le 2B
$$

For every $j\in Z$:

$$
\widehat A_j=0
$$

therefore:

$$
\widehat D_j=0
$$

So two different openings exist only if there is a nonzero bounded polynomial:

$$
D\ne0
$$

such that:

$$
\widehat D_Z=0
$$

The core binding assumption is:

$$
ker({NTT}_Z)\cap[-2B,2B]^N=\{0\}
$$

or at least that finding such a nonzero $D$ is infeasible.

---

# 7. Heuristic Security Estimate

For parameters:

- $N=1024$
- $|Z|=512$
- $q=2^{16}$
- $B=2^5=32$

The number of possible bounded differences is approximately:

$$
(4B+1)^N = 129^{1024}
$$

If using the simpler bound for one error vector:

$$
(2B+1)^N = 65^{1024}
$$

The probability that a random vector satisfies $|Z|=512$ independent Fourier-zero equations is:

$$
q^{-512}=2^{-8192}
$$

For bounded error vectors, the heuristic expected number is:

$$
\frac{65^{1024}}{(2^{16})^{512}}
\approx 2^{-2025}
$$

For bounded differences, the estimate is:

$$
\frac{129^{1024}}{(2^{16})^{512}}
\approx 2^{-1000}
$$

Both are extremely small under the random-subspace heuristic.

This is not a proof. It assumes the selected Fourier rows behave like random linear constraints on bounded coefficient vectors.

---

# 8. Why Random Z Is Important

The number $|Z|$ is not enough. The positions matter.

Bad example:

$$
Z=\{1,3,5,\dots,N-1\}
$$

Then:

$$
D(X)=1+X^{N/2}
$$

has very small coefficients and satisfies:

$$
\widehat D_Z=0
$$

So structured choices of $Z$ can destroy binding.

Good practice:

- choose $Z$ uniformly at random
- reject obvious structured sets
- test for low-weight annihilators such as $1\pm X^r$
- experimentally search for short kernel vectors

---

# 9. Evaluation Proof

After committing:

$$
V=AF+E
$$

The prover wants to prove:

$$
F(x_i)=y_i
$$

for public points $x_i$.

The prover must convince the verifier that the hidden committed polynomial $F$ evaluates correctly at these points, without revealing $F$.

For one point $x$, the condition:

$$
F(x)=y
$$

is equivalent to saying:

$$
F(X)-y
$$

is divisible by:

$$
X-x
$$

So there exists a witness polynomial $W_x(X)$ such that:

$$
F(X)-y=(X-x)W_x(X)
$$

For multiple points, define:

$$
P(X)=\prod_{i=1}^m (X-x_i)
$$

and let $Y(X)$ be the interpolation polynomial satisfying:

$$
Y(x_i)=y_i
$$

Then the evaluation statement is:

$$
F(X)-Y(X)=P(X)W(X)
$$

for some witness polynomial $W$.

---

# 10. Combining Commitment and Evaluation

The proof must show knowledge of polynomials:

$$
F,E,W
$$

such that:

$$
V=AF+E
$$

$$
F-Y=PW
$$

and:

$$
|E_i|\le B
$$

The verifier should learn neither $F$ nor $E$.

The public data are:

- $A$
- $V$
- points $x_i$
- claimed values $y_i$
- zero set $Z$
- bound $B$

---

# 11. Fiat-Shamir / Sigma-Style Sketch

A possible proof sketch follows the standard commit-challenge-response pattern.

The prover samples random masking polynomials:

$$
R_F, R_E, R_W
$$

with $R_E$ bounded or sampled from a distribution compatible with zero-knowledge.

The prover sends masked commitments corresponding to the two relations:

$$
T_1 = A R_F + R_E
$$

$$
T_2 = R_F - P R_W
$$

The verifier derives challenge $c$ by Fiat-Shamir:

$$
c = H(A,V,Z,x_i,y_i,T_1,T_2)
$$

The prover responds:

$$
Z_F = R_F + cF
$$

$$
Z_E = R_E + cE
$$

$$
Z_W = R_W + cW
$$

The verifier checks:

$$
A Z_F + Z_E = T_1 + cV
$$

and:

$$
Z_F - PZ_W = T_2 + cY
$$

The verifier also checks that $Z_E$ is within the allowed response bound, using rejection sampling or another zero-knowledge-preserving technique.

This proves that the same hidden $F$ is used both in the commitment relation and in the evaluation relation.

---

# 12. Hiding of F(x_i)

If the values $y_i$ are public, then the verifier learns those values by definition.

If the goal is to prove a hidden evaluation relation without revealing $y_i$, then $Y$ must also be committed, and the proof should show:

$$
F(x_i)=Y_i
$$

or some predicate over $Y_i$, without opening the values.

For example, the prover may commit to $Y$ or to individual values $y_i$, and prove consistency:

$$
F-Y=PW
$$

without revealing $Y$.

This gives two modes:

## Public evaluation mode

The verifier learns:

$$
F(x_i)=y_i
$$

## Hidden evaluation mode

The verifier learns only that committed values are consistent with evaluations of $F$, but not the values themselves.

---

# 13. Important Zero-Knowledge Caveat

The response:

$$
Z_F = R_F+cF
$$

can leak information about $F$ unless $R_F$ is sampled from a sufficiently wide distribution and rejection sampling is used.

Similarly:

$$
Z_E = R_E+cE
$$

can leak information about $E$.

A complete construction must specify:

- response bounds
- masking distributions
- rejection sampling
- simulator strategy
- challenge size
- soundness error

Without these details, this remains a protocol sketch, not a complete zero-knowledge proof.

---

# 14. What Changed From the Original Version

The original idea used:

$$
V=AF+E
$$

as an RLWE-style commitment.

The issue was that if $F$ is unrestricted and $A$ is invertible, then a prover can choose bounded $E$ and solve for $F$.

The updated version fixes this by making $A$ non-invertible in a controlled way:

$$
\widehat A_Z=0
$$

This forces:

$$
\widehat E_Z=\widehat V_Z
$$

and makes bounded $E$ nontrivial to fake.

So the commitment becomes binding because of the bounded Fourier-syndrome condition, while $F$ can remain unrestricted.

---

# 15. Open Problems

## 15.1 Formal Binding Proof

Need to prove or assume hardness of finding:

$$
D\ne0,\quad |D_i|\le2B,\quad \widehat D_Z=0
$$

for random $Z$.

## 15.2 Efficient PLONK Implementation

Need an efficient way to enforce:

$$
\widehat E_Z=\widehat V_Z
$$

and coefficient bounds on $E$.

Options include:

- in-circuit NTT
- partial NTT constraints
- custom gates
- lookup-assisted butterflies

## 15.3 Zero-Knowledge

Need a full simulator and rejection sampling analysis.

## 15.4 Hidden Evaluation Values

Need a clear construction for the mode where $F(x_i)$ is proven correct but not revealed.

---

# 16. Summary

The construction is a polynomial evaluation proof based on a negacyclic RLWE-style commitment:

$$
V=AF+E
$$

where:

- $F$ is the hidden polynomial
- $E$ is a bounded hiding/error polynomial
- $A$ has random Fourier zeros
- $V$ binds $F$ through the bounded Fourier syndrome of $E$

The prover proves:

$$
F(x_i)=y_i
$$

by showing:

$$
F-Y=PW
$$

while also proving consistency with the commitment:

$$
V=AF+E
$$

The key new idea is that random Fourier zeros in $A$ remove the need to bound $F$, because any alternative opening would require a nonzero bounded error difference whose FFT vanishes on the selected zero set.

This turns the binding problem into a bounded Fourier-kernel problem.
