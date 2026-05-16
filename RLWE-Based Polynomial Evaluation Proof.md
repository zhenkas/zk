# RLWE-Based Polynomial Evaluation Proof

## 1. Overview

This document describes a compact RLWE-based proof system for proving hidden polynomial evaluations inside PLONK-style protocols.

The construction combines:

- RLWE commitments,
- Fiat-Shamir lattice proofs,
- sparse polynomial challenges,
- multi-point polynomial openings.

The main goal is:

- prove evaluations of a hidden polynomial,
- avoid interpolation attacks,
- preserve compact proof size,
- maintain strong soundness over small finite fields.

---

# 2. Base Ring

We work over the ring:

$$R_q = \mathbb{F}_q[X]/(X^n-1)$$

Example parameters:

$$n = 1024$$

$$q \approx 2^{16}$$

The hidden witness polynomial is:

$$F(X)$$

The prover samples a small error polynomial:

$$E(X)$$

---

# 3. RLWE Commitment

The prover commits to the witness using:

$$V(X) = A(X)F(X) + E(X)$$

where:

- $$A(X)$$ is public,
- $$F(X)$$ is hidden,
- $$E(X)$$ is small.

This commitment binds the witness polynomial to the RLWE relation.

---

# 4. Sparse Polynomial Challenge

A sparse Fiat-Shamir challenge polynomial is derived:

$$C(X)$$

Example structure:

- degree $$n-1$$,
- only 20 nonzero coefficients,
- coefficients in $$\{-1,+1\}$$.

The challenge is derived from:

$$C(X)=H(t_v,V,\text{context})$$

The sparse structure keeps response norms bounded while providing a very large challenge space.

---

# 5. Response Polynomials

The prover samples masking polynomials:

$$W_f(X)$$

$$W_e(X)$$

and computes:

$$t_v(X)=A(X)W_f(X)+W_e(X)$$

Responses are:

$$z_f(X)=W_f(X)+C(X)F(X)$$

$$z_e(X)=W_e(X)+C(X)E(X)$$

All operations are performed inside:

$$R_q$$

---

# 6. Main Verification Equation

Verifier checks:

$$A(X)z_f(X)+z_e(X)=t_v(X)+C(X)V(X)$$

because:

$$A(W_f+CF)+(W_e+CE)=AW_f+W_e+C(AF+E)$$

which equals:

$$t_v + CV$$

This proves consistency with the RLWE commitment.

---

# 7. Smallness Verification

Verifier checks coefficient bounds:

$$|z_e[i]| < B$$

for all coefficients.

This prevents arbitrary response forgery and preserves RLWE hardness assumptions.

---

# 8. Evaluation Points

Evaluation points are derived from the committed state:

$$x_i = H(V,\text{context},i)$$

Importantly:

- the prover knows $$x_i$$ before constructing openings,
- but cannot freely forge $$z_f$$,
- because $$z_f$$ is bound to the RLWE commitment relation.

This removes the interpolation attack that exists without RLWE binding.

---

# 9. Polynomial Openings

The prover wishes to prove:

$$F(x_i)=y_i$$

for multiple points.

For each point:

$$t_i = W_f(x_i)$$

Verifier checks:

$$z_f(x_i)=t_i+C(x_i)y_i$$

because:

$$z_f(x_i)=W_f(x_i)+C(x_i)F(x_i)$$

and:

$$F(x_i)=y_i$$

The verifier must additionally ensure:

$$C(x_i)\neq0$$

so the challenge does not vanish at the evaluation point.

---

# 10. Multi-Point Soundness

A single opening equation over:

$$\mathbb{F}_q$$

provides approximately:

$$1/q$$

soundness.

Since:

$$q \approx 2^{16}$$

multiple independent opening points are used.

Example:

- 10 independent points,
- each checked independently.

This yields approximately:

$$2^{-160}$$

opening soundness.

---

# 11. Why Interpolation Attacks No Longer Work

Without RLWE commitments, a prover could construct an arbitrary polynomial passing through all checked points.

After introducing:

$$V=AF+E$$

the response polynomial:

$$z_f$$

is no longer freely selectable.

To forge openings, the prover would need to produce another valid decomposition:

$$V=AF'+E'$$

with bounded:

$$E'$$

which is assumed hard under RLWE.

Thus the interpolation attack disappears.

---

# 12. Security Intuition

The protocol derives security from two layers:

## RLWE Binding

The commitment:

$$V=AF+E$$

prevents arbitrary witness modification.

## Fiat-Shamir Challenges

The sparse polynomial challenge:

$$C(X)$$

provides a massive challenge space while preserving bounded response norms.

---

# 13. Proof Structure

The final proof contains:

- commitment polynomial:

$$V(X)$$

- masking commitment:

$$t_v(X)$$

- response polynomials:

$$z_f(X), z_e(X)$$

- opening pairs:

$$(t_i,y_i)$$

for each evaluation point.

---

# 14. Open Questions

The construction still requires formal analysis regarding:

- exact soundness bounds,
- zero-knowledge leakage,
- challenge distribution,
- sparse polynomial challenge security,
- extraction guarantees,
- recursive composition,
- rejection sampling requirements.

The protocol should currently be viewed as an experimental RLWE-based polynomial evaluation proof system.
