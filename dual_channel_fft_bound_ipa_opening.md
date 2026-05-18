# Dual-Channel FFT-Bound IPA Opening (Draft)

## 1. Setup

Work in negacyclic ring:

\[
R=\mathbb F[X]/(X^N+1)
\]

Polynomial to open:

\[
f(z)=\sum_{i=0}^{m-1} a_i z^i
\]

where:

\[
a_i\in R
\]

Evaluation point:

\[
x\in R
\]

Claimed value:

\[
y=f(x)
\]

---

# 2. Dual Generator Channels

Construct two generator families:

\[
A_i \in R
\]

\[
B_i \in R
\]

with FFT-domain zero masks.

Let:

\[
\Omega=\{0,\dots,N-1\}
\]

Choose two disjoint global sets:

\[
T_A,T_B\subset\Omega
\]

such that:

\[
|T_A|=|T_B|=N/4
\]

\[
T_A\cap T_B=\varnothing
\]

For each coefficient \(i\):

Choose random subset:

\[
R_i\subset\Omega\setminus(T_A\cup T_B)
\]

with:

\[
|R_i|=N/4
\]

Define FFT zero masks:

\[
Z(A_i)=T_A\cup R_i
\]

\[
Z(B_i)=\Omega\setminus Z(A_i)
\]

Thus:

\[
|Z(A_i)|=|Z(B_i)|=N/2
\]

and:

\[
Z(B_i)=\overline{Z(A_i)}
\]

Meaning every FFT coordinate of every coefficient is captured by exactly one channel.

---

# 3. Base Commitments

Define:

\[
C_1=\sum_i a_i A_i
\]

\[
C_2=\sum_i a_i B_i
\]

Shared evaluation term:

\[
P_1=C_1+yU
\]

\[
P_2=C_2+yU
\]

where:

\[
U\in R
\]

is a public random ring element.

---

# 4. Precommit Folding Phase

Before seeing evaluation folding randomness, derive Fiat-Shamir challenges:

\[
c_0,\dots,c_{\log m-1}
\]

Fold recursively:

\[
C_1^\star=\operatorname{Fold}_{c}(C_1)
\]

\[
C_2^\star=\operatorname{Fold}_{c}(C_2)
\]

where folding weights are standard IPA-style:

\[
\alpha_i(c)
\]

Thus:

\[
C_1^\star
=
\sum_i \alpha_i(c)a_iA_i
\]

\[
C_2^\star
=
\sum_i \alpha_i(c)a_iB_i
\]

Publish:

\[
(C_1^\star,C_2^\star)
\]

before evaluation folding begins.

---

# 5. Evaluation Folding Phase

After observing:

\[
(C_1^\star,C_2^\star,x,y)
\]

derive two new folding challenge sequences:

\[
p^{(1)}_0,\dots,p^{(1)}_{\log m-1}
\]

\[
p^{(2)}_0,\dots,p^{(2)}_{\log m-1}
\]

such that:

\[
\widehat{p^{(1)}_k}[T_A]
=
\widehat{c_k}[T_A]
\]

\[
\widehat{p^{(2)}_k}[T_B]
=
\widehat{c_k}[T_B]
\]

while remaining FFT coordinates are fresh random values.

Thus each channel shares half of its FFT frequencies with the precommit fold.

---

# 6. IPA Folding

Fold evaluation channels independently:

\[
P_1^\star
=
\operatorname{Fold}_{p^{(1)}}(P_1)
\]

\[
P_2^\star
=
\operatorname{Fold}_{p^{(2)}}(P_2)
\]

Define folded witnesses:

\[
a_1^\star
=
\sum_i \alpha_i(p^{(1)})a_i
\]

\[
a_2^\star
=
\sum_i \alpha_i(p^{(2)})a_i
\]

Folded evaluation vectors:

\[
b_1^\star
=
\operatorname{Fold}_{p^{(1)}}(b(x))
\]

\[
b_2^\star
=
\operatorname{Fold}_{p^{(2)}}(b(x))
\]

Folded generators:

\[
A_1^\star
=
\sum_i \alpha_i(p^{(1)})A_i
\]

\[
B_2^\star
=
\sum_i \alpha_i(p^{(2)})B_i
\]

---

# 7. Final Verification Equations

Verifier checks:

## Channel 1 IPA equation

\[
P_1^\star
\stackrel{?}{=}
a_1^\star A_1^\star
+
(a_1^\star b_1^\star)U
\]

## Channel 2 IPA equation

\[
P_2^\star
\stackrel{?}{=}
a_2^\star B_2^\star
+
(a_2^\star b_2^\star)U
\]

---

# 8. FFT Overlap Binding Checks

Verifier additionally checks:

## Channel 1 overlap

\[
\widehat{
P_1^\star
-
C_1^\star
-
(a_1^\star b_1^\star)U
}[T_A]
=
0
\]

because:

\[
\widehat A_i[T_A]=0
\]

for all \(i\).

---

## Channel 2 overlap

\[
\widehat{
P_2^\star
-
C_2^\star
-
(a_2^\star b_2^\star)U
}[T_B]
=
0
\]

because:

\[
\widehat B_i[T_B]=0
\]

for all \(i\).

---

# 9. Intended Security Intuition

The prover commits to:

\[
(C_1^\star,C_2^\star)
\]

before evaluation randomness is known.

After evaluation point \(x\) is revealed:

- evaluation folding uses fresh challenges;
- but half of FFT frequencies remain tied to the precommit fold.

Since:

\[
Z(B_i)=\overline{Z(A_i)}
\]

the two overlap checks together cover all FFT frequencies.

A cheating prover attempting to alter folded witness values after seeing \(x\) must solve incompatible constraints on complementary FFT supports.

The construction attempts to replace ECC discrete-log binding with FFT-support noninvertibility across dual complementary channels.
