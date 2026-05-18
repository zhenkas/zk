# IPA with SIS

## Goal

Build a recursive IPA-like polynomial commitment / opening system over:

```text
R_Q = Z_Q[X] / (X^N + 1)
```

suitable for proving real PLONK-style polynomial evaluations:

```text
F(x) = y
```

while:

- keeping witness coefficients bounded,
- avoiding ECC/discrete-log assumptions,
- using recursive folding,
- obtaining soundness from:
  - Fiat-Shamir challenge entropy,
  - bounded representation / SIS-style hardness.

---

# 1. Important Structure

This construction works over **ring elements**, not scalar coefficients.

Each:

```text
a_i , A_i
```

is itself a polynomial in:

```text
R_Q = Z_Q[X] / (X^N + 1)
```

with degree:

```text
N - 1
```

Typical example:

```text
N = 1024
```

So commitment:

```text
C_a = Σ a_i A_i
```

is a sum of polynomial products inside the ring.

There may be millions of:

```text
a_i
```

inside one commitment.

---

# 2. Two Moduli

The actual PLONK polynomial lives over small modulus:

```text
q
```

Example:

```text
q = 2^16
```

But proof arithmetic is performed over much larger modulus:

```text
Q
```

where:

```text
Q >> q
```

The commitment/folding/proof system works entirely modulo:

```text
Q
```

Final evaluation relation:

```text
F(x) = Y_Q
```

is checked in:

```text
R_Q
```

and verifier finally reduces:

```text
y = Y_Q mod q
```

Thus:

- PLONK semantics remain over small field/ring `q`,
- proof security/boundedness happen over large modulus `Q`.

---

# 3. Why Big Q Matters

The entire bounded-SIS logic depends on:

```text
|a*| < Q / 4
```

or average/norm equivalent.

If folded witness remains much smaller than `Q`, then equality modulo `Q` behaves like true integer equality.

Thus fake openings require finding genuine bounded collisions:

```text
Σ Δ_i A_i = 0 mod Q
```

with small:

```text
Δ_i
```

This becomes SIS/module-SIS flavored hardness.

---

# 4. Standard IPA Problem

Classic IPA folding:

```text
a' = u a_L + u^-1 a_R
```

works well over ECC/scalars because arbitrary field inverses are cheap.

In bounded integer/ring systems this is problematic because:

- `u^-1` may be huge modulo `Q`,
- coefficient norms explode,
- final folded witness becomes unbounded.

Monomial challenge attempt:

```text
u = ± X^k
```

fixes inverse growth because:

```text
u^-1 = ± X^(N-k)
```

is still sparse.

But entropy per round is only:

```text
2N
```

For `N = 1024`:

```text
≈ 2^11
```

which is too small.

---

# 5. Signed-Permutation Folding

Instead of scalar challenge `u`, use two independent signed permutation operators:

```text
S_L , S_R
```

Each operator performs:

- random coefficient sign flips,
- optional random cyclic shift/rotation.

Example:

```text
(Sa)_i = σ_i a_(i-k)
```

where:

```text
σ_i ∈ {-1,+1}
```

and `k` is random shift.

These operators satisfy:

```text
S^-1 = S^T
```

and preserve norms:

```text
||Sa||_2 = ||a||_2
```

```text
||Sa||_∞ = ||a||_∞
```

---

# 6. New Folding Rule

Fold witness:

```text
a' = S_L a_L + S_R a_R
```

Fold public/evaluation vector:

```text
b' = S_L^-T b_L + S_R^-T b_R
```

Since signed permutations are orthogonal:

```text
S^-T = S
```

so practically:

```text
b' = S_L b_L + S_R b_R
```

---

# 7. Inner Product Expansion

Expanded folded inner product:

```text
<a', b'>
=
<a_L, b_L>
+
<a_R, b_R>
+
L + R
```

where:

```text
L = <S_L a_L , S_R^-T b_R>
```

```text
R = <S_R a_R , S_L^-T b_L>
```

These become the IPA cross terms.

Thus standard recursive IPA logic still works with operator challenges instead of scalar `u`.

---

# 8. Entropy

If challenge flips only 20 random signs:

Challenge count:

```text
C(N,20) * 2^20
```

For `N = 1024`:

```text
≈ 2^165
```

per round.

This is vastly stronger than monomial scalar challenges.

---

# 9. Coefficient Growth

Worst-case coefficient growth:

```text
|a'_i| ≤ |a_L,i| + |a_R,i|
```

So:

```text
||a'||_∞ ≤ 2b
```

if:

```text
|a_L,i| , |a_R,i| ≤ b
```

No inverse explosion occurs.

---

# 10. Statistical / Average Growth

Because sign patterns are random:

```text
E[ ||a'||_2^2 ]
=
||a_L||_2^2
+
||a_R||_2^2
```

Cross terms cancel in expectation.

After folding `m` witnesses:

Typical RMS growth:

```text
||a*|| ~ sqrt(m) * q
```

instead of worst-case:

```text
m * q
```

This allows significantly smaller modulus `Q`.

---

# 11. Statistical Modulus Formula

Using average/norm bounds:

```text
Q_bits
=
log2(q)
+
0.5 log2(m)
+
log2(kc)
+
3
```

where:

- `q` = coefficient modulus,
- `m` = number of folded witnesses,
- `k` = number of opened polynomials,
- `c` = mixing sparsity factor,
- `+3` = 2 safety bits + factor 2 margin.

Final folded witness size:

```text
a*_bits
=
log2(q)
+
0.5 log2(m)
+
log2(kc)
+
2
```

---

# 12. Proof Size Formula

Proof consists of:

- commitment `C`,
- evaluations `Y_i`,
- recursive `L,R` pairs,
- final folded witness `a*`.

Approximate size:

```text
Size_bits
=
N *
[
Q_bits * (1 + k + 2 log2(m))
+
a*_bits
]
```

Approximation:

```text
Size_bits
≈
N * Q_bits * (2 + k + 2 log2(m))
```

---

# 13. Example

Parameters:

```text
q = 2^16
m = 2^20
N = 1024
k = 1
```

Then:

```text
Q_bits ≈ 29
```

Estimated proof size:

```text
≈ 156 KiB
```

Important observation:

```text
N = 1024
```

means each ring element contains 1024 coefficients.

And:

```text
m = 2^20
```

means IPA recursively folds about one million ring elements.

Therefore total represented PLONK trace size is approximately:

```text
1024 * 2^20 ≈ 2^30
```

which is about:

```text
1 billion rows / coefficients
```

inside a single recursive opening proof.

# 14. Limb Decomposition

Split polynomial:

```text
F(X)
=
F_0(X)
+
16F_1(X)
+
16^2F_2(X)
+
16^3F_3(X)
```

with small limb coefficients:

```text
0 ≤ a_i < 16
```

Now:

```text
q_limb = 2^4
```

This reduces:

```text
Q_bits
```

and proof size significantly.

Example:

```text
m = 2^20
```

4-limb split:

```text
Q_bits ≈ 19
```

Proof size:

```text
≈ 109 KiB
```


# 14.5 Example Size Table

The following table shows approximate proof sizes using the statistical / average-growth model.

Assumptions:

```text
q = 2^16
N = 1024
c = 1
k = 1
```

For limb decomposition:

```text
F(X)
=
F_0(X)
+
16F_1(X)
+
16^2F_2(X)
+
16^3F_3(X)
```

with:

```text
q_limb = 2^4
k = 4
```

| PLONK rows | Folded ring elements `m` | Direct proof | 4-limb decomposition |
|---|---:|---:|---:|
| `2^20 ≈ 1 million` | `2^10` | `≈ 69 KiB` | `≈ 45 KiB` |
| `2^30 ≈ 1 billion` | `2^20` | `≈ 156 KiB` | `≈ 109 KiB` |

Explanation:

```text
PLONK rows ≈ N * m
```

where:

```text
N = 1024 = 2^10
```

So:

```text
m = 2^10  =>  2^20 rows
m = 2^20  =>  2^30 rows
```

Even billion-row PLONK traces remain within roughly:

```text
100–150 KiB
```

proof size range under this bounded recursive folding model.

---

# 15. Main Security Intuition

Security now comes from two independent layers.

## A. IPA Transcript Binding

Protected by high-entropy signed-permutation challenges.

## B. Bounded Representation Hardness

Attacker must find:

```text
Σ Δ_i A_i = 0 mod Q
```

with all:

```text
Δ_i
```

remaining bounded/small.

This is SIS/module-SIS flavored hardness.

---

# 16. Important Insight

Challenge entropy multiplies across rounds ONLY if:

- prover commits `L,R` before challenge,
- fake transcript cannot be repaired retroactively.

Weak scalar monomial challenges (`~2^11`) are insufficient.

Signed-permutation operators provide:

- huge entropy,
- bounded inverses,
- stable coefficient norms,
- practical recursive folding.

---

# 17. Intended Use

This system is intended as a polynomial commitment / opening layer for real PLONK-style proving systems.

The underlying PLONK polynomials remain standard low-modulus polynomials over:

```text
R_q
```

while recursive proof arithmetic and soundness amplification occur over:

```text
R_Q
```

using bounded recursive folding.
