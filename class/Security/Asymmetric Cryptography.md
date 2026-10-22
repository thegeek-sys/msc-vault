---
Class: "[[Security]]"
---
---
## Index
- [[#Key exchange|Key exchange]]
- [[#Diffie-Hellman|Diffie-Hellman]]
	- [[#Diffie-Hellman#Security|Security]]
- [[#Asymmetric Cryptography|Asymmetric Cryptography]]
- [[#RSA|RSA]]
	- [[#RSA#Key generation|Key generation]]
	- [[#RSA#Security|Security]]
	- [[#RSA#Alternatives to RSA: elliptic curves|Alternatives to RSA: elliptic curves]]
- [[#Hybrid encryption|Hybrid encryption]]
- [[#Digital signatures|Digital signatures]]
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
**RSA** (Rivest, Shamir, Adleman, 1977) is the most famous asymmetric cryptography scheme. As already said each party has a pair of keys $(pk,sk)$ where $pk=(e,n)$ and $sk=(d,n)$ and $n$ is the RSA modulus, so that everybody can encrypt for you, but only you can decrypt.

![[Pasted image 20261021181435.png]]

$$
\begin{align}
\text{encrypt:}\quad c=m^{e} \text{ mod }n\\
\text{decrypt:}\quad m=c^{d}\text{ mod }n
\end{align}
$$
where $m<n$ and $d$ is the multiplicative inverse of $e \text{ mod } \varphi(n)$

>[!tldr] Number theory background
>- *Fermat’s little theorem* → $x^{p}\equiv x\pmod{p}$ for a prime $p$
>- then we have → $x^{p}\cdot x^{-1}\equiv x\cdot x^{-1}\pmod{p}\to x^{(p-1)}\equiv 1 \pmod{p}$ where $(p-1)$ is exactly $\varphi(p)$, follows the Euler’s theorem below
>- *Euler’s theorem* → $x^{\varphi(n)}\equiv 1\pmod{n}$ if $x$ and $n$ are coprime
>- for RSA moduli $n=pq$ (distinct primes $p$, $q$) → $\varphi(n)=\varphi(p)\cdot \varphi (q)=(p-1)(q-1)$
>- $x^{k\varphi(n)+1}\equiv x\pmod{n}$ for *all* $x$ and integers $k$

### Key generation
1. choose two large primes $p$ and $q$
2. compute $n=p\cdot q$ and $z=\varphi(n)=(p-1)(q-1)$
3. choose $e<z$ coprime to $z$ → *the public key is $(e,n)$*
4. find $d$ with $e\cdot d\equiv 1 \pmod{z}$ (using Euclidean algorithm) → *the private key is $(d,n)$*

>[!question] Why decryption works
>Seen that $e\cdot d\equiv 1\pmod{\varphi(n)}$ it means that $e\cdot d=k\cdot \varphi(n)+1$ for some $k$.
>
>Furthermore $x^{k\varphi(n)+1}= (x^{\varphi(n)})^{k}\cdot x \overset{\text{Euler}}{=}1^{k}\cdot x=x \pmod{n}$
>$$(m^{e})^{d}=m^{ed}=m^{k\varphi(n)+1}\equiv m\pmod{n}$$

>[!example]
>Key generation:
>- $p=5$, $q=11$ → $n=55$, $\varphi(n)=4\cdot 10=40$
>- $e=3$ (coprime to $40$)
>- $d=27$, since $3\cdot 27=81=2\cdot 40+1$ where $k=2$ and $\varphi(55)=40$
>
>Finally:
>- encrypt $m=7$ → $c=7^{3}\text{ mod }55=13$
>- decrypt $c=13$ → $m=13^{27}\text{ mod }55=7$
>
>In real-world keys $n$ has $2048$ bits or more

### Security
Encrypting with RSA is fairly easy ($c=m^{e}\text{ mod }n$), but decrypting without $d$ requires to compute $e$-th root modulo $n$, and for a composite $n$ (number which has at least one divider other $1$ and itself) this operation is hard (in the previous example would mean finding $x$ with $x^{3}\equiv 13 \pmod{55}$).

The best known method would be to factor $n$ into $p$ and $q$, and then generating the keys.

>[!info] Another method to find $p$ and $q$ using $\varphi(n)$
>However also knowing $\varphi(n)$ is equivalent as knowing the factorization of $n$; we have 
>$$
>\begin{align}
>\varphi(n)&=(p-1)(q-1)=pq-p-q+1= \\
>&=n-p-q+1\to \boxed{p+q=n-\varphi(n)+1}
>\end{align}
>$$
>At this point, it is possible to solve the following equation:
>$$
>t^{2}-(p+q)t+pq=(t-p)(t-q)=0
>$$
>
>So I just have to solve the equation for $t$ and I will obtain $p$ and $q$ (possible because I know both $p+q$ and $pq=n$).

For this reason it’s suggested to use a big $n$ in order to make it harder the factorization process (at least $2048$ bits, the biggest number ever factorized was $829$ bits in 2020).

Other then this **naive RSA is insecure**:
- it is *deterministic* → the same $m$ always give the same $c$ (chose plaintext attacks, attacker encrypts $m$ with $pk$ and then compares)
- it is *malleable* → $c\cdot r^{e}=(m\cdot r)^{e}$ (chosen ciphertext attacks, get $c\cdot r^{e}$ decrypted, receive $m\cdot r$, divide by $r$)
- small messages with small $e$ are dangerous → if $m^{3}<n$ and $e=3$, there is no modular reduction (it’s enough the $e$-th ordinary root, common choice today $e=65537$)

The solution is to use **randomized padding** (e.g. OAEP), that make the encryption probabilistic.

>[!question] Would you rather want a small e or a small d?
>Neither should be too small. A small $d$ can be recovered easily; a small $e$ allows taking the $e$-th root if no modular reduction occurred

### Alternatives to RSA: elliptic curves
So far we’ve just used cycle groups in modular arithmetic. The **elliptic curves** (*EC*) are another class of groups, whose elements are points in a curve.

For well chosen curves (weak curves makes the discrete logarithm easy, typically we use standardized curves like P-256, Curve25519/Ed25519, secp256k1) there is no known sub-exponential algorithms for the discrete logarithm. Furthermore it needs much smaller keys compared to RSA (a EC key is $256$ bit long and gives almost $128$ bits of security, comparable to a RSA key of $3072$ bit).

Elliptic curves cryptography uses the discrete logarithm problem generalized for the curves (e.g. ECDH, ECDSA, EdDSA, etc.).

>[!tldr] Outlook, post-quantum cryptography
>The Shor algorithm, on a sufficiently large quantum computer, efficiently computes both factorization and discrete logarithms, breaking:
>- RSA
>- Diffie-Hellman
>- elliptic curves
>- Symmetric crypto and hashes → are only mildly affected (sufficient to use $256$ bits keys)
>
>Even if a quantum-computer still doesn’t exists, as of today we need to protect ourselves against this attack due to “harvest now, decrypt later” (recorded traffic could be decrypted in the future).
>
>The solution is the *NIST standardized post-quantum schemes* of 2024, which presented *ML-KEM* (Kyber) for key exchange and *ML-DSA* (Dilithium) for signatures.
>
>However have been already deployed solutions in hybrid mode (classical + post-quantum) (e.g. in TLS key exchange).

---
## Hybrid encryption
Both symmetric and asymmetric encryption have advantages and disadvantages.

|            | Pro                          | Cons                                              |
| ---------- | ---------------------------- | ------------------------------------------------- |
| Symmetric  | fast                         | needs $n(n-1)/2$ keys to exchange in a secure way |
| Asymmetric | public keys can be published | slow                                              |
To overcome the limits of both parties the idea would be to combine both: encrypt symmetric key using asymmetric encryption, and the data symmetrically.

![[Pasted image 20261022142843.png|470]]

---
## Digital signatures
When using MACs (HMAC, CBC-MAC, etc.) they require a shared key (means $n(n-1)/2$ keys for $n$ parties) but there is no non-repudiation policy, so either party could have created the tag.

The solution is to use **digital signatures** based on asymmetric cryptography: use the $sk$ to sign and the $pk$ for the verification. In this way only the holder of the private key and sign (non-repudiation), and everyone can verify a signature using the signer’s public key.

>[!info] Using RSA for digital signatures
>Alice’s private key $(d,n)$ (and $p$, $q$), public key $(e,n)$, secure hash function $H$.
>
>In order to sign $\sigma=H(m)^{d}\text{ mod }n$, and to verify $H(m)\overset{?}{=}\sigma^{e} \text{ mod }n$.
>
>![[Pasted image 20261022144409.png]]
>
>In practice it corresponds to RSA-PSS (with randomized padding), ECDSA, Ed25519.
>
>>[!question] Why sign $H(m)$?
>>The first reason is *efficiency*: asymmetric cryptography is slow, so it is faster to only encrypt a short hash.
>>
>>Furthermore without hashing signature leak information (by using $\sigma=m^{d}$ anyone can obtain the message $m=\sigma^{e}$, as $e$ is public) and can be forged (given $\sigma=m^{d}$, then $\sigma^{x}=(m^{d})^{x}=(m^{x})^{d}$, so $\sigma^{x}$ is a valid signature of $m^{x}$).
