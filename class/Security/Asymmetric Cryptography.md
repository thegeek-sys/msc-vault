---
Class: "[[Security]]"
---
---
## Key exchange
Symmetric encryption and MACs need a shared secret for each pair of parties ($n$ parties need $n(n-1)/2$ keys), so for a real-world application it would mean storing thousands of different keys.

Furthermore a problem arises when we want to share a key; indeed with symmetric encryption the only two ways would be either through out-of-band channels (e.g. meeting in person, phone) but are impractical either by  sending the key over a public channel which would reveal it to the adversary.

For this reasons we need a secure way to exchanging keys. The idea would be to derive a secret key from a publicly exchanged information.

>[!info] Merkle’s puzzles (background)
>$A$ sends $B$ many puzzles (thousands) $p_{i}=\{ID_{i}, S_{i}\}w_{i}$ where:
>- $ID_{i}$ → the identifier of the puzzle
>- $S_{i}$ → the secret
>- $w_{i}$ → a weak key used to encrypt the $i$-th puzzle
>
>Then $B$ brute-forces one random puzzle $p_{j}$ and send back to $A$ the $ID_{j}$. In such a way $A$ and $B$ indirectly agreed to use the strong key $S_{j}$ ($A$ knows to which puzzle $ID_{j}$ corresponds).
>
>![[Pasted image 20261021163733.png|330]]
>
>A passive attacker in this case can see $ID_{j}$ but does not know which is the corresponding puzzle; in order to get this information he has to brute-force all the puzzles which takes $O(N\cdot m)$ for $m$ puzzles ($B$ needed just $O(N)$). However an active attacker can easily intercept $ID_{j}$ and send back to $A$ another $ID$ that he has brute-forced.
>
>Of course a quadratic gap is not enough for real security, but it inspired Diffie-Hellman.

---
## Diffie-Hellman

>[!tldr] Excursion: modular arithmetic
>- $x \mod n$ is the remainder when dividing $x$ by $n$
>- compute $+$, $-$, $\cdot$ on remainder → $(a\cdot b) \mod n=((a \mod n)\cdot(b\mod n))$ (same for $+$ and $-$)
>- multiplicative inverse $x$ of $a$ → $(a\cdot x)\equiv 1 \pmod{n}$
>- Euler’s totient $\varphi(n)$ → how many numbers $<n$ are coprime to $n$ (for a prime $p$, $\varphi(p)=p-1$)

Let’s analyze how it works:
- public parameters → prime modulus $p$ and base $g$
- each party picks a private exponent (each party has either $x$ or $y$) → $(x,y)$
- they exchange $X=g^{x} \text{ mod }p$ and $Y=g^{y}\text{ mod }p$ (not the exponents!)

In this way they are both able to compute the same key:
$$
k=Y^{x}=g^{xy}=X^{y} \pmod{p}
$$
![[Pasted image 20261021165628.png|center|600]]

>[!example]
>- public → $p=23$, $g=5$
>- Alice picks → $x=6$, $X=5^{6}\text{ mod 23}=8$
>- Bob picks → $y=15$, $Y=5^{15}\text{ mod }23=19$
>- Alice → $k=19^{6}\text{ mod }23=2$
>- Bob → $k=8^{15}\text{ mod 23}=2$
>
>An eavesdropper sees $p$, $g$, $X$, $Y$, but needs $x$ or $y$ (infeasible for big numbers).

### Security
Modular exponentiation is a one-way function, in fact computing the *discrete logarithm* (finding $x$ for $g^{x}\text{ mod }p$) is hard for large numbers making exhaustive search infeasible for large numbers. However Diffie-Hellman only protects againts a passive attacker; an active attacker can modify the exchanged $X$ and $Y$ compromising the communication between $A$ and $B$.


![[Pasted image 20261021171207.png|550]]

A machine-in-the-middle (MitM) can observe, drop, alter, and insert messages, and because DH has no type of authentication process it cannot be detected. The attack proceeds as follows:
1. Alice sends $X=g^{x}\text{ mod }p$ to Bob, but Mallory (MitM) intercepts it
2. Mallory chooses its own exponent $x'$ and sends to Bob $X'=g^{x'}\text{ mod }p$
3. Bob replies with $Y=g^{y}\text{ mod }p$, that Mallory intercepts, and sends to Alice $Y'=g^{y'}\text{ mod }p$

Now Mallory has access and can read all the messages that Alice and Bob exchange.

---
## Asymmetric Cryptography
An alternative to exchanging a key per pair is to use **asymmetric encryption**. This new paradigm consist in each party publishing information that let anyone communicate securely with it.

So each party creates a key pair $(pk,sk)$ where $pk$ is the *private key* (kept secret) and $pk$ is *public key* (published)

![[Pasted image 20261021172753.png]]

The encryption shown in the picture guarantees confidentiality (Alice encrypts using Bob’s public key, so that only Bob, using his private key, can decrypt it), but does not guarantee authentication (or integrity). In order to obtain authentication, Alice should encrypt using her own private key, and Bob decrypt using Alice’s public key (Bob will know for sure that the message was sent by Alice, as she only has her private key, but loses confidentiality as everyone can read the message using Alice’s public key).

---
## RSA
**RSA** (Rivest, Shamir, Adleman, 1977) is the most famous asymmetric cryptography scheme. As already said each party has a pair of keys, so that everybody can encrypt for you, but only you can decrypt.

>[!tldr] Number theory background
>- *Fermat’s little theorem* → $x^{p}\equiv x\pmod{p}$ for a prime $p$
>- *Euler’s theorem* → $x^{\varphi(n)}\equiv 1\pmod{n}$ if $x$ and $n$ are coprime
>- for RSA moduli $n=pq$ (distinct primes $p$, $q$) → $\varphi(n)=\varphi(p)\cdot \varphi (q)=(p-1)(q-1)$
>- $x^{k\varphi(n)+1}\equiv x\pmod{n}$ for *all* $x$ and integers $k$

### Key generation
1. choose two large primes $p$ and $q$
2. compute $n=p\cdot q$ and $z=\varphi(n)=(p-1)(q-1)$
3. choose $e<z$ coprime to $z$ → **the public key is $(e,n)$**
4. find $d$ with $e\cdot d\equiv 1 \pmod{z}$ (using Euler’s theorem itself) → *the private key is $(d,n)$*