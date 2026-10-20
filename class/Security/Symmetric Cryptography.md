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
The practice of protecting information and systems from improper, unauthorized, or accidental modification, deletion or destruction, by making the receiver aware if the message was compromised or not.

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

---
## Symmetric Cryptography
When we talk about symmetric cryptography the communicating parties share a secret key which is unknown to anyone else, and hard to guess. Moreover it can be used for just one session (session key) or for all future communication (long-term key).

Here’s the most important historical examples:
- Caesar cipher → mono-alphabetic shift
- Vigenère cipher → poly-alphabetic shift (only broken 300 years after its publication thanks to cryptoanalysis)
- Enigma → encryption machine used by Germany in WWII (similar to Vigenère cipher)

![[Pasted image 20261019151538.png]]

### One-Time Pad (OTP)
![[Pasted image 20261019151610.png|center|550]]

The OTP is perfectly secret, in fact the ciphertext reveals no information about the plaintext (other than its length) and even brute-forcing all keys does not help when the message is sufficiently short (there could be multiple plausible plaintext of the same ciphertext by using different keys).

>[!example]- Brute-Forcing the key
>![[Pasted image 20261019152659.png|400]]
>
>Iterating over all 5-byte keys generates all possible 5-byte strings, so it’s impossible to tell which key $k$ was the correct one.

In order to make this scheme secure, some conditions must be satisfied:
1. a key must be used for exactly one message
2. the key must be (at least) as long as the message

>[!bug] Key reuse leaks information (condition 1 missing)
>![[Pasted image 20261019154202.png|350]]
>
>Encrypting two different messages with the same key you would result in the XOR between the two plaintext messages.
>$$c_{1} \oplus c_{2}=(m_{1}\oplus k)\oplus (m_{2}\oplus k)=m_{1}\oplus m_{2}$$
>
>In this way an attacker can obtain lots information about the plaintext; if he knows a part of $m_{1}$ he can direclty obtain the corresponding part of $m_{2}$ etc.

>[!bug] Key looking leaks information (condition 2 missing)
>![[Pasted image 20261019154556.png|450]]
>
>By using a key that is shorter than the message an attacker can obtain information about the plaintext. In fact repeated patterns in the plaintext at a certain distance (the length of the key) would be encrypted in the same way, making this scheme vulnerable to frequency attacks
>

>[!info] Breaking a repeated key
>If $| k|=l$, every byte $m_{i+j\cdot l}$ was XORed with the same key byte $k_{i}$. There are three possible attacks:
>1. split the ciphertext into $l$ parts and run frequency analysis on each
>2. $c_{i}\oplus c_{j}=m_{i}\oplus m_{j}$ for two blocks of length $l$; guess parts of $m_{i}$ wether the resulting $m_{j}$ is sensible text
>3. If the plaintext uses only a character class (e.g. lowercase letters), each key byte $k_{i}$ must map all its ciphertext bytes into that class, which prunes the key space

---
## Block ciphers
### Naive encryption via block cipher
![[Pasted image 20261019160827.png|center|550]]
In the naive version of this cipher we have a key $k$ as long as message $m$ and the idea is to use a fresh random pad $r$, encrypted with $k$, that replaces the OTP key. In this way the key $k$ can be used for multiple messages. The drawback is that $(c,r)$ is twice as long as $m$.

However this naive implementation still remains unpractical to use, we would like to have:
- messages longer than one block
- keys much shorter than messages, and reusable for many messages
- ciphertext not (much) longer than plaintext
- efficient decryption (comparable to encryption)
- patterns in the plaintext must not be visible in the ciphertext
- use $F_{k}$ to encrypt one block and $F^{-1}_{k}$ to decrypt it (not used in naive and CTR)

We can describe $F_{k}(m)$ as a deterministic **pseudorandom permutation** (*PRP*) with key $k$ and $n$-bit input $m$ such that:
- **permutation** → a bijection on $\{0,1\}^{n}$, so the output is also $n$ bits long
- **pseudorandomness** → for a random key, the output looks random to every efficient algorithm that does not know the key

And a good PRP can be used as a block cipher (like in AES) and good block ciphers fulfill the *strict avalanche criterion* (flipping one input bit flips every output bit with probability $50\%$).

---
## Advanced Encryption Standard
The **Advanced Encryption Standard** (*AES*) is the most important block cipher in practice (standardized in 2001), and is commonly used to encrypt whole disks. It uses:
- block size → $128$ bits
- key size → $128$ ($10$ rounds), $192$ ($12$ rounds), $256$ ($14$ rounds)

It combines **permutation** (*diffusion*) with **substitution** (*confusion*); by not using complex mathematical operations (e.g. multiplication, square root, etc.) but just logic operation it can remain computationally lightweight while still needing the same effort for encryption and decryption.

>[!tldr] How AES works
>![[Pasted image 20261019164858.png|center|300]]
>
>**1. Inputs.** A 128-bit key and a 128-bit data block (the plaintext).
>**2. Key expansion (E).** The key is expanded into 11 different round keys of 128 bits each ($11 \times 128$ bits), called $K_0, \dots, K_{10}$. They are all derived from the original key and are not the same key repeated.
>**3. Initial AddRoundKey.** XOR the data block with $K_0$.
>**4. Rounds 1 to 10.** Each round applies four operations to the 4×4 byte state:
>- **SubBytes** → each of the 16 bytes is replaced via a fixed lookup table (the S-box). This is _confusion_, and it is the non-linear step.
>- **ShiftRows:** the rows of the state are rotated by different offsets, so bytes change position.
>- **MixColumns** → each 4-byte column is mixed so that every output byte depends on all 4 input bytes. Together with ShiftRows, this is _diffusion_.
>- **AddRoundKey** → XOR the state with the round key $K_i$ of the current round.
>(In real AES, the last round skips MixColumns. The image show a generic round.)
>
>**5. Output.** After round 10, the state is the 128-bit ciphertext.

In order to *decrypt* the ciphertext, each operation is inverted and applied in reverse order. Round keys are derived as for encryption, but applied in reverse order:
- AddRoundKey (XOR) is its own reverse
- MixColumns and ShiftRows can be inverted
- S-box can be inverted

---
## Modes of operation
Block ciphers work of fixed-size blocks (e.g. $128$ bits) and its **mode of operation** describe how to encrypt messages of arbitrary length (e.g. ECB, CBC, CTR, GCM, etc.). All the different modes does not differ on $F_{k}$ (the same as AES) but changes how it is used. 

>[!warning]
>Some modes leak plaintext information even if the block cipher is secure.

### Electronic Code Book (ECB)
![[Pasted image 20261019175336.png|center|480]]

The **Electronic Code Book** (ECB) encrypts each block separately but reuses the same key $k$ for each block permutation, so the encryption of $m_{k}$ is independent from any other $m_{j}$.

#### Drawbacks
The first drawback is that equal plaintext blocks result in equal ciphertext blocks, so similar plaintext have similar ciphertext (information about the plaintext leaks).

Another drawback is the fact that an attacker could interact and modify the single blocks independently, inserting targeted modifications for specific blocks leaving all the other blocks untouched.

Last bun not least there is the fact that blocks can be exchanged or deleted without affecting other blocks.

### Cipher Block Chaining (CBC)
![[Pasted image 20261019181528.png|center|350]]
In **Cipher Block Chaining** (CBC) each plaintext block is XORed with the previous ciphertext block before encryption, while the *initialization vector* (IV) is a radom, non-secret pad for the first block.

$$
\begin{align}
enc()\qquad &c_{i}\;:=F_{k}(m_{i}\oplus c_{i-1})\\
dec()\qquad &m_{i}\;:=c_{i-1}\oplus F^{-1}_{k}(c_{i})
\end{align}
$$
#### Properties
Advantages over ECB:
- identical plaintext blocks do not result in identical ciphertext blocks (not even the first one, due to the random IV)
- similar plaintext result in different ciphertext
- if an attacker flips a bit in $m_{i}$ the flip is propagates to $c_{i},\dots,c_{n}$
- if an attacker flips a bit in $c_{i}$ garbles $m_{i}$ and flips the same bit in $m_{i+1}$

However all that glitters is not gold, in fact the encryption process cannot be parallelized due to the strong data dependencies; contrary it is possible to parallelize the decryption process as the computation of $m_{i}$ does not depend from any other plaintext block.

>[!warning] CBC does not provide integrity
>Despite the error propagation, CBC cannot provide data integrity because the attacker can modify the plaintext in  a predictable way and the receiver has no way to determine whether the message is the original one or not.
>
>An attacker who intercepted $(c,IV)$ can:
>- flip bits in the IV (1), which flips the same bits in $m_{2}$ (2)
>- flip bits in $c_{i}$ (3), which flips the same bits in $m_{i+1}$ (4) and garbles $m_{i}$ (5)
>
>![[Pasted image 20261019183203.png]]

### Counter (CTR)
The **Counter** (CTR) mode turns a block cipher into a stream cipher, by encrypting a nonce (one for each message, not one per block) to produce a keystream and XORs it with the plaintext.

$$
c_{i}:=m_{i}\oplus F_{k}(\text{nonce}\mid\mid i)
$$

Because it uses one single nonce for each message it’s necessary to concatenate the message with the index of the current block otherwise $F_{k}$ would be identical for each block (and it would result exactly in the OTP).

#### Properties
- encryption and decryption can be parallelized, and allow random access
- only $F_{k}$ is needed, never $F^{-1}_{k}$
- it is the basis of GCM

>[!warning]
>Reusing a nonce under the same key would result in the same keystream (the same problem as reusing a one-time pad).

---
## Message Authentication Code (MAC)
A MAC provides **data-origin authentication** (and hence integrity), using a secret key. In particular the message goes through the $\text{MAC}_{k}$ function and generates an authentication tag $t$; the tag $t$ is attached to the message, and the receiver can apply the same $\text{MAC}_{k}$ function to the message and compare the newly generated $t_{\text{new}}$ with the received $t$. If $t_{\text{new}}==t$ the receiver can be sure that message was not altered by anyone (the only for an attacker to compromise the integrity would be to edit the message and the tag, but in order to generate a valid tag he should have the correct key).

![[Pasted image 20261019190746.png]]

In contrast with a plain checksum (e.g. CRC or bare hash) MAC is able to detect an adversary while checksums just accidental corruption (noise). A MAC detects adversarial modification, and identifies the origin.

>[!warning] Integrity vs. Authentication
>- **Integrity** → *has data been altered?* (a property of the *message*)
>- **Authentication** → *who produced it?* (a property of the *origin*)
>
>Furthermore we need to distinguish between:
>- *entity authentication* → who am I talking to?
>- *data-origin authentication* → who produced this message? (the one provided by MAC)
>
>| Mechanism               | Integrity?            | Origin?               |
>| ----------------------- | --------------------- | --------------------- |
>| Hash/CRC (no key)       | vs. accidental change | no                    |
>| MAC/signature (keyed)   | vs. an adversary      | yes                   |
>| Password login (entity) | not per message       | who, at session start |
>
>Authentication implies integrity, but integrity alone does not prove who sent the data. If I know that a message was produced by Alice, I also know that it isn’t been altered (because an altered message does not come from Alice); while an unaltered message may come from anyone.

### Naive MAC from a block cipher
![[Pasted image 20261019192036.png]]

In this case an adversary who does not know $k$ cannot forge a valid tag (Bob’s verification succeeds only if Alice generated the message). However it’s not practical because the message must fit into one block.

---
## Hash functions
A cryptographic hash function $H(m)$ is deterministic, takes input of any length and is:
- **compressing** → fixed-length output for arbitrary-length input
- **collision resistant** → it is practically impossible to find $m\neq m'$ with $H(m)=H(m')$
- **preimage resistant** (one-way) → given $H(m)$ for a random $m$, it is infeasible to find $m$ except by trying all candidates
- **second-preimage resistant** → given $m$, it is infeasible to find $m'\neq m$ with $H(m')=H(m)$

>[!info] The birthday paradox
>Making an hash function collision resistant is the harder challenge due to the birthday paradox.
>
>With 23 people in a room, the chance that two share a birthday is $>50\%$ because the number of pairs $n(n-1)/2$ grows quadratically.
>
>For this reason in hash functions a $d$-bit hash, a generic attack needs about $2^{d/2}$ messages to find a collision so the output must not be too short. A practical example is the SHA-1 (160 bits) where the best attacks need about $2^{61}-2^{63}$ operations instead of the generic $2^{80}$.
>
>The current standard for hash function is SHA-2 (e.g. SHA-256) and SHA-3 (Keccak).
>
>>[!tldr]- Security of hash functions over time
>>![[Pasted image 20261019203611.png]]
 
To sum it up:

| Property | Attacker's task | Generic cost |
|---|---|---|
| Collision resistance | Find any $m \neq m'$ with $H(m) = H(m')$ | $2^{n/2}$ |
| Preimage resistance | Given $H(m)$, find $m$ | $2^n$ |
| Second-preimage resistance | Given $m$, find $m' \neq m$ with $H(m') = H(m)$ | $2^n$ |

>[!tldr]- Hash vs. MAC
>The hash have no key, so everyone can compute it (even the attacker). So if an attacker intercepts a message, modify it and recomputes the hash, the receiver will never know that message has been modified.
>
>The only use of the has is to address an accidental corruption, not an adversary

### Hash-and-MAC
The [[Symmetric Cryptography#Naive MAC from a block cipher|naive MAC]] only works if the message fits into a block. In order to overcome this limit, first of all we compress the message using an hash function (creating a fixed length hash) and then the $\text{MAC}_{k}$ is computed on the hash. In this way it’s possible to manage messages of any length.

![[Pasted image 20261019200102.png]]

>[!error] The wrong way
>Someone could ask “why don’t we directly hash key and data together, $t=H(k\mid\mid m)$?”, but with Merkle-Damgard hashes (SHA-1, SHA-256) it’s vulnerable to **length extension attacks**.
>
>The final digest coincide with the internal state after the last block. An attacker that knows $H(k\mid\mid m)$ and the length of $k\mid\mid m$ can then resume the computation from the exact state where the original computation stopped and add some data.
>
>In particular this attack proceeds as it follows:
>1. **Compute the padding** → the padding appended to $k\mid\mid m$ depends only on the length, which the attacker knows. Call it $\text{pad}$.
>2. **Build the forged message** → $m \mid\mid \text{pad} \mid\mid m'$, where $m'$ is the data the attacker wants to append. The receiver will compute $H(k \mid\mid m \mid\mid pad \mid\mid m')$. The first part, $k\mid\mid m\mid\mid \text{pad}$, is exactly what produced $t$, so the state after those blocks is $t$. 
>3. **Resume the computation** → initialize the hash with state $= t$ (instead of the IV), process $m'$, and add the final padding based on the total length. The result is:
>
>$$t' = H(k \mid\mid m \mid\mid pad \mid\mid m')$$
>
>This is exactly the tag the server computes for $m \mid\mid pad \mid\mid m'$ (which is the message that the attacker is going to send to the receiver. $k$ was never needed.
>
>SHA-3 uses a “sponge” construction instead, which prevents extension attacks.

>[!success] The correct way, HMAC
>HMAC is proven secure (in contrast to simple keyed hashes). The encryption consists in two separate hash computations with an inner and an outer key.
>
>$$\text{HMAC}(k,m)=H((k'\oplus opad)\mid\mid H((k'\oplus ipad)\mid\mid m))$$
>
>where $k'$ is the key padded with zeros to the hash’s block size (e.g. 512 bits for SHA-256).
>
>In this case the external hash hides the internal state, so the length extension attack is not feasible.
>
>Currently it is still considered secure even with some weaknesses in the hash function (e.g. HASH-SHA1 is still considered secure).

To sum it up in order to use the naive MAC (there are other types of MAC functions that make you use arbitrarily-long messages) you can either:
- hash the message and then apply the MAC function
- use the HMAC

---
## Authenticated encryption
In order to obtain confidentiality and integrity with symmetric encryption you need to combine encryption and a MAC, but not every combination is secure. Let’s analyze all the possible combinations:
- **encrypt-and-MAC** → $c:=\text{End}_{k_{e}}(m), t:=\text{MAC}_{k_{m}}(m)$, the MAC is not designed for secrecy and may leak information about $m$ (e.g. repeated messages)
- **MAC-then-encrypt** → $t:=\text{MAC}_{k_{m}}(m), c:=\text{Enc}_{k_{e}}(m\mid\mid t)$, prone to padding-oracle attacks (successfully used against TLS)
- **encrypt-then-MAC** → $c:=Enc_{k_{e}}(m), t:=\text{MAC}_{k_{m}}(c)$, sound with independent keys $k_{e}\neq k_{m}$

The general rule is that different instances of cryptographic primitives should use independent keys.

### Authenticated encryption with associated data (AEAD)
There are modes that provide confidentiality and integrity in one operation (e.g. Galois/Counter Mode - GCM, which is a CTR-mode encryption plus an authentication tag computed during encryption, it is easy to parallelize and any bit flip in the ciphertext is detected).

For *associated data* we refer to authenticated but not encrypted data (e.g. packet headers).

AES-GCM and ChaCha20-Poly1305 are the standard choices today (e.g., in TLS 1.3), but like CTR it’s not possible to reuse a nonce under the same key.

