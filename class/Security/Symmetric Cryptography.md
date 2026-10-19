---
Class: "[[Security]]"
---
---
## Introduction
### Goals of Cryptrography

>[!warning] Composabilty
>Confidentiality and integrity are independent goals.
>
>In fact encryption alone does not detect modifications while integrity protection alone does not hide the content
>
>![[Pasted image 20261019102120.png]]
#### Confidentiality
Only authorized parties can learn information
![[Pasted image 20261019101406.png]]

#### Integrity
The practice of protecting information and systems from improper, unauthorized, or accidental modification, deletion or destruction.

![[Pasted image 20261019101717.png]]

---
## Kerckhoff’s Principle

>[!cite]
>”A cryptosystem should be secure if everything about the system, **except** the key, is public knowledge”

This principle is nowadays considered the standard, thanks to widely analyzed secure cryptosystems.

In fact security through algorithm obscurity is a bad idea because:
- it’s easier to hide a key than an algorithm
- it’s easier to change a key for a single than an algorithm for everybody (containment of beaches)
- it’s easier to communicate if everyone uses the same algorithms

#### Cryptorgraphic keys
If a key is a bit string of length $n$, the number of possible keys is $2^n$. So enumerating all of them takes time exponential in $n$.

According to the Kerckoff’s principles there should be no more efficient way to break a cryptosystem then trying all possible keys

---
## Randomness in security
Security very often reduces to how many bits an attacker must guess in order to compromise a cryptosystem, and if the attacker can predict the value, the mechanism is broken, no matter how strong is the algorithm.

This “guessing budget” is the *entropy* available to the defender (e.g. $n$ uniformly random bits → the attacker needs up to $2^n$ guesses)

>[!info] Entropy definition (background)
>Informally, entropy measures the attacker’s uncertainty about a value.
>
>For security, the useful notion is **min-entropy** ($H_{\infty}$): the attacker’s best single guess
>$$H_{\infty}=-\log_{2}(\max_{x}\text{P}[X=x])$$
>
>(among all the possible keys select the one with the highest probability of being the right one, while $-\log_{2}p=\log_{2} \frac{1}{p}$ is the number bits necessary to write that number)
>
>So if the most likely value has probability $p$, that values it $-\log_{2}p$ bits. A uniform $n$-bit value has $n$ bits of min-entropy while a skewed distribution has less.
>
>>[!example]
>>Let’s suppose that $1\%$ of the users select `123456` as a password. In this case the most probable value has $p=0.01$, so $H_{\infty}=-\log_{2}0.01 = \log_{2} \frac{1}{0.01}=\log_{2} 100\approx 6.6$ bits.
>
>>[!question] Why min-entropy and not Shannon entropy?
>>The attacker attacks the most likely case first. This is exactly why human-chosen passwords are weak: the distribution is far from uniform

### True Randomness vs. Pseudorandomness
**True randomness** come from a physical entropy source (e.g. thermal noise, timing jitter, hardware RNGs etc.) and it is slow, has a limited supply, but it’s genuinely unpredictable.

On the other hand **pseudorandomness** (PRNG) is a deterministic algorithm that stretches a small seed into a long stream. It is fast and unlimited, but fully determined by the seed; so anyone who learns (or guesses) the seed can reproduce the entire stream.

However a stream can pass statistical tests and still be *predictable* to an attacker (e.g. `rand()`/`Math.random()` are meant only for simulations, not security).
Security needs a **Cryprographically Secure PRNG** (CSPRNG), so that even seeing many outputs, an attacker cannot predict the next bit or recover the state. For this reason typically in security we use the OS source `/dev/urandom`, `getrandom()`, `CryptGenRandom` and not the language’s default `random`.

>[!hint] Rule of thumb
>If a value protects something, it must come from a CSPRNG, well seeded

