# ➜ [blog.konstas.us](https://blog.konstas.us)
### Technical research, engineering archives, and publications at [**blog.konstas.us**](https://blog.konstas.us)

---

## Public-Key Cryptosystems Based on the Discrete Logarithm Problem

Public-key cryptography relies on the existence of **trapdoor one-way functions**—mathematical operations that are computationally efficient to evaluate in the forward direction, but computationally infeasible to invert without specific trapdoor information. Alongside integer factorization (RSA), the **Discrete Logarithm Problem (DLP)** forms the theoretical and practical foundation of modern asymmetric cryptography, key establishment protocols, and digital signature standards.

```
                    ┌────────────────────────────────────────────────────────┐
                    │               Discrete Logarithm Problem               │
                    │               Given (g, p, y), find x in:             │
                    │                    y ≡ gˣ (mod p)                      │
                    └───────────────────────────┬────────────────────────────┘
                                                │
       ┌────────────────────────┬───────────────┴───────────────┬────────────────────────┐
       ▼                        ▼                               ▼                        ▼
┌──────────────┐      ┌────────────────────┐          ┌────────────────────┐    ┌─────────────────┐
│ Multiplicative│     │   Diffie-Hellman   │          │      ElGamal       │    │ Elliptic Curves │
│ Group ℤₚ*    │      │    Key Exchange    │          │  Asymmetric Scheme │    │     E(𝔽ₚ)       │
│ • Order p-1  │      │ • Forward Secrecy  │          │ • IND-CPA Security │    │ • Chord & Tangent│
│ • Generators │      │ • CDH / DDH Proofs │          │ • Homomorphic Mult │    │ • ECDLP Hardness│
└──────────────┘      └────────────────────┘          └────────────────────┘    └─────────────────┘
```

---

### 1. The Multiplicative Group $\mathbb{Z}_p^*$ and the Discrete Logarithm Problem

#### Group Structure & Algebraic Properties
For any prime $p$, the set of integers coprime to $p$ forms a finite abelian group under modular multiplication, denoted $\mathbb{Z}_p^*$:

$$\mathbb{Z}_p^* = \{1, 2, 3, \dots, p-1\}$$

- **Group Order:** $|\mathbb{Z}_p^*| = \phi(p) = p - 1$.
- **Group Operation:** Binary modular multiplication $(a \cdot b) \bmod p$, satisfying closure, associativity, and commutativity.
- **Identity Element:** $e = 1$.
- **Multiplicative Inverses:** For every $a \in \mathbb{Z}_p^*$, there exists a unique $a^{-1} \in \mathbb{Z}_p^*$ satisfying $a \cdot a^{-1} \equiv 1 \pmod p$, computed efficiently via the Extended Euclidean Algorithm or Fermat's Little Theorem ($a^{p-2} \bmod p$).
- **Cyclic Structure & Generators:** $\mathbb{Z}_p^*$ is strictly cyclic. There exists at least one generator (primitive root) $g$ whose powers generate the entire group:
  $$\langle g \rangle = \{g^1, g^2, g^3, \dots, g^{p-1}\} = \mathbb{Z}_p^*$$
  The number of primitive roots in $\mathbb{Z}_p^*$ is $\phi(p - 1)$.

#### The Discrete Logarithm Problem (DLP)
Given a generator $g \in \mathbb{Z}_p^*$, a modulus $p$, and an element $y \in \mathbb{Z}_p^*$, find the unique integer $x \in \mathbb{Z}_{p-1}$ such that:

$$y \equiv g^x \pmod p$$

- **Forward Direction (Modular Exponentiation):** $y \equiv g^x \pmod p$ is computed in polynomial time $O(\log x \cdot \log^2 p)$ via repeated squaring (square-and-multiply).
- **Inverse Direction (Discrete Logarithm):** Extracting $x = \log_g y$ has no known polynomial-time classical algorithm for properly configured parameter groups.

---

### 2. Diffie-Hellman Key Exchange (DHKE)

Introduced by Whitfield Diffie and Martin Hellman in 1976, DHKE was the first published method allowing two parties to establish a shared cryptographic secret over an unencrypted, adversarial communication channel without transmitting the secret itself.

```
       Alice (Private: a)                                  Bob (Private: b)
  ─────────────────────────────                       ───────────────────────────
   Public Parameters: (p, g)                           Public Parameters: (p, g)
   
   Computes: A = gᵃ mod p          ─── Public A ───►   Computes: B = gᵇ mod p
                                   ◄─── Public B ───
   
   Computes Shared Secret:                             Computes Shared Secret:
   K = Bᵃ mod p                                        K = Aᵇ mod p
     = (gᵇ)ᵃ = gᵃᵇ mod p                                 = (gᵃ)ᵇ = gᵃᵇ mod p
```

#### Protocol Mechanics
1. **Public Domain Parameters:** A large prime $p$ and a generator $g \in \mathbb{Z}_p^*$.
2. **Key Generation:**
   - Alice selects private key $a \in_R [2, p-2]$ and transmits public key $A = g^a \bmod p$.
   - Bob selects private key $b \in_R [2, p-2]$ and transmits public key $B = g^b \bmod p$.
3. **Shared Secret Derivation:**
   - Alice computes: $K_A = B^a \equiv (g^b)^a \equiv g^{ab} \pmod p$.
   - Bob computes: $K_B = A^b \equiv (g^a)^b \equiv g^{ab} \pmod p$.
   - Both derive identical secret $K = g^{ab} \pmod p$, passed to a Key Derivation Function (e.g., HKDF-SHA256).

#### Computational Assumptions
- **Discrete Logarithm Problem (DLP):** Given $(g, g^a)$, computing $a$ is hard.
- **Computational Diffie-Hellman (CDH):** Given $(g, g^a, g^b)$, computing $g^{ab}$ is hard without knowledge of $a$ or $b$.
- **Decisional Diffie-Hellman (DDH):** Distinguishing $g^{ab}$ from a uniformly random element in $\mathbb{Z}_p^*$ given $(g, g^a, g^b)$ is computationally infeasible.

#### Significance
- **Solved Key Distribution:** Replaced physical pre-shared key couriers in network engineering.
- **Perfect Forward Secrecy (PFS):** In Ephemeral Diffie-Hellman (DHE / ECDHE), session keys are generated per connection and discarded, protecting historical traffic against future server key compromise.
- **Protocol Foundation:** Underpins **TLS 1.3**, **SSH-2**, **IPsec / IKEv2**, **Signal Protocol**, and **WireGuard**.

---

### 3. ElGamal Public-Key Cryptosystem

Proposed by Taher Elgamal in 1985, this scheme extended Diffie-Hellman into full public-key encryption and digital signatures.

```
                                  ElGamal Encryption
  
   Public Key: (p, g, y = gˣ mod p)                    Private Key: x
  
   Sender (Alice):                                     Receiver (Bob):
   - Choose ephemeral random k                         - Receives (c₁, c₂)
   - Compute c₁ = gᵏ mod p                             - Computes shared mask: s = c₁ˣ mod p
   - Compute shared mask s = yᵏ mod p                  - Computes s⁻¹ mod p via EEA
   - Compute c₂ = m · s mod p                          - Recovers message: m = c₂ · s⁻¹ mod p
   
   Ciphertext: C = (c₁, c₂)
```

#### Protocol Specification
1. **Key Generation:** Private key $x \in_R [2, p-2]$, public key $y = g^x \bmod p$.
2. **Encryption:** Message $m \in \mathbb{Z}_p^*$, random nonce $k \in_R [2, p-2]$ with $\gcd(k, p-1) = 1$:
   $$c_1 \equiv g^k \pmod p, \quad c_2 \equiv m \cdot y^k \pmod p$$
3. **Decryption:** Using private key $x$:
   $$s = c_1^x \equiv g^{kx} \pmod p \implies m \equiv c_2 \cdot s^{-1} \pmod p$$

#### Properties
- **Semantic Security (IND-CPA):** Random ephemeral nonce $k$ guarantees distinct ciphertexts for identical messages under DDH.
- **Multiplicative Homomorphism:** $C_1 \cdot C_2 = (c_{1,1}c_{2,1}, \; c_{1,2}c_{2,2})$ decrypts directly to $m_1 \cdot m_2 \bmod p$, foundational in e-voting tallying and privacy-preserving MPC.
- **Signature Evolution:** Formed the basis for the NIST **Digital Signature Algorithm (DSA)**, **ECDSA**, and **Schnorr / BIP-340**.

---

### 4. Elliptic Curves over Finite Fields $E(\mathbb{F}_p)$

Elliptic Curve Cryptography (ECC) transfers the discrete logarithm problem from the multiplicative group $\mathbb{Z}_p^*$ to the additive group of geometric points on an algebraic curve.

#### Short Weierstrass Form
Over a prime field $\mathbb{F}_p$ ($p > 3$), an elliptic curve $E(\mathbb{F}_p)$ is defined by:

$$y^2 \equiv x^3 + ax + b \pmod p$$

with non-singularity condition $\Delta = 4a^3 + 27b^2 \not\equiv 0 \pmod p$, along with the point at infinity $\mathcal{O}$ acting as the identity element.

```
        Point Addition (P ≠ Q)                      Point Doubling (P = Q)
   
        y │       . (Curve)                         y │       . (Curve)
          │      / \                                  │      / \
        P ┼─────/───\─── Q                          P ┼─────*   (Tangent Line)
          │    /     \                                │    / \
          │   /       \                               │   /   \
        ──┼──/─────────\────── x                    ──┼──/─────\────── x
          │ /           \                             │ /       \
      P+Q ┼* (Reflect -R)                         2P  ┼* (Reflect -R)
          │                                           │
```

#### The Group Law (Chord-and-Tangent)
For points $P = (x_1, y_1)$ and $Q = (x_2, y_2)$:
1. **Identity:** $P + \mathcal{O} = P$, and $P + (-P) = \mathcal{O}$ where $-P = (x_1, -y_1 \bmod p)$.
2. **Slope $\lambda$:**
   $$\lambda = \begin{cases} 
   \frac{y_2 - y_1}{x_2 - x_1} \pmod p & \text{if } P \neq Q \text{ (Secant line)} \\ 
   \frac{3x_1^2 + a}{2y_1} \pmod p & \text{if } P = Q \text{ (Tangent line)} 
   \end{cases}$$
3. **Point Addition $R = P + Q = (x_3, y_3)$:**
   $$x_3 \equiv \lambda^2 - x_1 - x_2 \pmod p$$
   $$y_3 \equiv \lambda(x_1 - x_3) - y_1 \pmod p$$

#### Elliptic Curve Discrete Logarithm Problem (ECDLP)
Given a base point $P \in E(\mathbb{F}_p)$ of prime order $n$ and a public point $Q = [k]P = \underbrace{P + P + \dots + P}_{k \text{ times}}$, finding the scalar integer $k \in \mathbb{Z}_n$ is the **ECDLP**.

---

### 5. Algebraic Correspondence & Cryptanalytic Landscape

| Mathematical Property | Multiplicative Group $\mathbb{Z}_p^*$ | Elliptic Curve Group $E(\mathbb{F}_p)$ |
| :--- | :--- | :--- |
| **Elements** | Integers $\{1, 2, \dots, p-1\}$ | Points $(x, y) \in \mathbb{F}_p \times \mathbb{F}_p$ plus $\mathcal{O}$ |
| **Group Operation** | Modular Multiplication ($a \cdot b \bmod p$) | Point Addition ($P + Q$) via Chord-and-Tangent |
| **Identity Element** | $e = 1$ | Point at infinity $\mathcal{O}$ |
| **Inverse** | $a^{-1} \pmod p$ via Extended Euclidean | $-P = (x, -y \bmod p)$ (Reflection across x-axis) |
| **Repeated Operation** | Exponentiation: $g^k \bmod p$ | Scalar Multiplication: $[k]P$ |
| **Discrete Logarithm** | Find $x$ in $y \equiv g^x \pmod p$ | Find $k$ in $Q = [k]P$ (**ECDLP**) |
| **Best Classical Attack** | **Subexponential:** Index Calculus ($L_p[1/3]$) | **Exponential:** Pollard's $\rho$ ($\mathcal{O}(\sqrt{n})$) |
| **NIST 128-bit Security Key** | **3072 bits** | **256 bits** (Curve25519, secp256r1) |

#### Hardening Requirements
- **Finite Fields:** Safe prime generation ($p = 2q + 1$) to neutralize Pohlig-Hellman subgroup attacks.
- **Elliptic Curves:** Strict subgroup validation ($n \cdot P = \mathcal{O}$) and prime-order curve selection to prevent invalid curve / small-subgroup attacks.
- **Side-Channel Defense:** Constant-time scalar multiplication (Montgomery Ladder) to eliminate timing and power analysis (SPA/DPA).
- **Post-Quantum Transition:** Shor's algorithm solves DLP/ECDLP in polynomial time $O((\log p)^3)$, driving global adoption of NIST Post-Quantum standards (**ML-KEM** / **ML-DSA**).

---

Also a musician — new releases and updates at [**pulseintimetunes.agency**](https://pulseintimetunes.agency)
