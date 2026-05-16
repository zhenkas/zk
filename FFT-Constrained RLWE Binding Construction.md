# FFT-Constrained RLWE Binding Construction

## Overview

We study a modified RLWE/LWE-style public key construction in the negacyclic ring:

$$R_q = \mathbb Z_q[x]/(x^N + 1)$$

with large degree $$N = 1024$$.

The goal is to remove the usual requirement that the secret $$s$$ must be sampled from a small distribution, while still preserving strong binding properties.

The main idea is to use public polynomials whose FFT/NTT spectra contain structured zeros.

---

# Basic Construction

Standard RLWE form:

$$P = A s + e$$

where:

- $$A \in R_q$$ is public
- $$s \in R_q$$ is the secret
- $$e \in R_q$$ is a small bounded error

with coefficient bound:

$$|e_i| < B_e$$

---

# Initial Observation

Suppose the FFT/NTT spectrum of $$A$$ contains zeros on a subset:

$$Z_A \subset \{0,\dots,N-1\}$$

with:

$$|Z_A| = N/2$$

Then in Fourier space:

$$\widehat P_j = \widehat A_j \widehat s_j + \widehat e_j$$

and for all $$j \in Z_A$$:

$$\widehat P_j = \widehat e_j$$

because:

$$\widehat A_j = 0$$

This creates a large kernel:

$$\widehat s_j \text{ arbitrary on } Z_A$$

so many different secrets produce the same public value.

Therefore a single masked polynomial is insufficient.

---

# Dual-Masked Construction

We instead use two public equations:

$$P_1 = A s + e_1$$

$$P_2 = B s + e_2$$

where:

- $$A,B \in R_q$$
- $$A$$ and $$B$$ have complementary FFT zero sets

Specifically:

$$\widehat A_j = 0 \iff \widehat B_j \neq 0$$

and ideally:

$$\widehat B_j = 0 \iff \widehat A_j \neq 0$$

Thus every FFT coordinate of $$s$$ is constrained by at least one equation.

---

# Fourier-Space Structure

In FFT/NTT representation:

$$\widehat P_{1,j} = \widehat A_j \widehat s_j + \widehat e_{1,j}$$

$$\widehat P_{2,j} = \widehat B_j \widehat s_j + \widehat e_{2,j}$$

For each spectral coordinate:

- either $$\widehat A_j \neq 0$$
- or $$\widehat B_j \neq 0$$

Therefore no frequency component of $$s$$ is invisible.

---

# Binding Intuition

Suppose an attacker attempts to open the same public key with another secret:

$$s' = s + \Delta$$

Then:

$$e_1' = e_1 - A\Delta$$

$$e_2' = e_2 - B\Delta$$

To succeed, both new errors must remain small:

$$|e_1'| < B_e$$

$$|e_2'| < B_e$$

Because $$A$$ and $$B$$ cover complementary FFT halves, every spectral component of $$\Delta$$ is penalized somewhere.

Thus even tiny modifications of $$s$$ generally force at least one error vector to become large.

This removes the large kernel attack present in the single-equation version.

---

# FFT-Constrained Error Space

A key assumption is that the FFT zero locations are random-looking and not algebraically structured.

Consider the condition:

$$\widehat e[z_i] = 0$$

for $$N/2$$ random FFT coordinates.

This imposes roughly $$N/2$$ independent linear constraints on coefficient-space vectors.

Therefore the set:

$$\mathcal L_Z = \{ e \in \mathbb Z^N : \widehat e[z_i]=0 \}$$

forms a lower-dimensional lattice/subspace.

The probability that a random bounded vector satisfies all constraints is heuristically:

$$(2B_e+1)^{-N/2}$$

which is negligible for practical parameters.

Example:

- $$N=1024$$
- $$B_e=3$$

gives approximately:

$$7^{-512} \approx 2^{-1438}$$

---

# Security Intuition

The construction behaves like a constrained decoding problem.

Given:

$$P_1 = A s + e_1$$

$$P_2 = B s + e_2$$

find another valid opening:

$$(s',e_1',e_2')$$

such that both errors remain short.

Unlike standard RLWE, the security does not primarily rely on hiding a small secret. Instead, it relies on the difficulty of simultaneously satisfying:

1. spectral masking constraints,
2. coefficient shortness constraints,
3. consistency across both equations.

---

# Important Assumptions

The following assumptions are critical.

## 1. Complementary Spectral Coverage

There must be no coordinate where both vanish:

$$\widehat A_j = \widehat B_j = 0$$

otherwise that frequency becomes unconstrained.

---

## 2. Random-Looking Zero Positions

Zero sets should avoid simple algebraic patterns.

Structured patterns may create low-dimensional symmetries or easy short vectors.

---

## 3. Nonzero FFT Values Must Be Strong

All nonzero spectral coefficients should remain safely invertible/non-negligible modulo $$q$$.

Tiny spectral coefficients may create weakly constrained directions.

---

## 4. Large Minimum Distance

The constrained lattice:

$$\mathcal L_Z = \{ e : \widehat e_Z = 0 \}$$

should have shortest vector:

$$\lambda_1(\mathcal L_Z)$$

significantly larger than the allowed error bound.

This is likely the key quantitative security parameter.

---

# Open Questions

Important unresolved questions include:

- precise reduction to RLWE/LWE-like assumptions,
- spectral leakage effects,
- behavior under lattice reduction attacks,
- minimum distance estimates for random FFT-constrained lattices,
- optimal distribution for $$A,B$$,
- interaction with modular arithmetic and NTT domains,
- whether attacks can exploit the split spectral structure.

---

# Current Intuition

The construction appears promising as a binding-oriented primitive:

- changing $$s$$ locally is difficult,
- fake openings require solving a global constrained short-vector problem,
- random FFT masking may produce strong geometric constraints.

However, the system is not standard RLWE and likely requires a new dedicated security analysis.
