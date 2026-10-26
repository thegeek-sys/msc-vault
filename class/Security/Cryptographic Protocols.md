---
Class: "[[Security]]"
---
---
## Index
- [[#Introduction|Introduction]]
	- [[#Introduction#Threats to cryptographic protocols|Threats to cryptographic protocols]]
	- [[#Introduction#Notation|Notation]]
- [[#Reflection attack|Reflection attack]]
	- [[#Reflection attack#Prevention|Prevention]]
- [[#Replay attack|Replay attack]]
	- [[#Replay attack#Prevention|Prevention]]
		- [[#Prevention#Timestamps|Timestamps]]
		- [[#Prevention#Nonces|Nonces]]
	- [[#Replay attack#Quiz|Quiz]]
- [[#Needham-Schroeder mutual authentication protocol|Needham-Schroeder mutual authentication protocol]]
	- [[#Needham-Schroeder mutual authentication protocol#The attack|The attack]]
	- [[#Needham-Schroeder mutual authentication protocol#Needham-Schroeder-Lowe|Needham-Schroeder-Lowe]]
	- [[#Needham-Schroeder mutual authentication protocol#Designing and Implementing Protocols|Designing and Implementing Protocols]]
- [[#Password-based authentication|Password-based authentication]]
	- [[#Password-based authentication#Challenge-response authentication|Challenge-response authentication]]
	- [[#Password-based authentication#Password-Authenticated Key Exchange (background)|Password-Authenticated Key Exchange (background)]]
---
## Introduction
In an untrusted world where trusted principals exchange messages over a network with malicious entities, everybody can read, manipulate, drop, repeat, and inject messages in transit.

In this prospective cryptographic primitives provide integrity, authenticity, and confidentiality of messages, but how they are used depends on the security goals and the attacker’s capabilities. An attacker can break a protocol by intercepting, duplicating, and resending messages, without breaking any encryption.

### Threats to cryptographic protocols
Attacks are often surprising, but can be classified by how the attacker interacts with protocol sessions, he can:
- *passive* → eavesdrop
- *active* → modify, inject

While the attacks can violate:
- confidentiality
- integrity
- authenticity

### Notation
- $\{m\}_{k}$ → $m$ encrypted with symmetric key $k$ (only parties that know $k$ can decrypt)
- $\{m\}_{pk}$ → $m$ encrypted with public key $pk$
- $\{m\}_{sk}$ → $m$ encrypted (signed) with private key $sk$

---
## Reflection attack
This technique is commonly used to break those protocols where receiving the password encrypted with the right key, establishes a connection where the two parties can send plain messages (a sort of handshake).

![[Pasted image 20261026161922.png]]

In this case Eve sends Alice’s message back to Alice, who accepts it as coming from Bob (correct key $k$) because with a shared symmetric key, Alice cannot tell whether Bob or she herself created the ciphertext.

### Prevention
To break the symmetry the sender needs to include his identity so that Eve cannot reflect the message anymore.
![[Pasted image 20261026162157.png|center|550]]

---
## Replay attack
In the replay attack, Eve just listens to the message Alice sent, and replays to the same receiver.

>[!example] Car keys
>A possible practical application of this attack would be to sniff the radio signal used to open a car, and replay it (without having the real keys).

![[Pasted image 20261026163740.png|550]]

A valid message is recorded and sent again later (or delayed, or suppressed). The protocol in the above picture does not guarantee freshness, so that Eve is able to authenticate as Alice with Alice’s old message.

In this case a signature/MAC gives *data-origin authentication*, but *not freshness* (proving who wrote a message is not the same as proving it is current, entity authentication needs a nonce/timestamp)

### Prevention
#### Timestamps
A way could be to include a timestamp $t$, so that Bob accepts a message only recents whose timestamp has not been seen before.

![[Pasted image 20261026164315.png|450]]

However it requires synchronized clocks, requires storing recently seen messages and protocols may deliver messages in different order (see TCP).

#### Nonces
![[Pasted image 20261026164702.png|450]]

The technique is to challenge-response with a *nonce* $n$: chosen by the verifier (and accepts a response only once) a random number used in only one protocol session

### Quiz

>[!question] What, even if properly applied, does not protect against replay attacks?
>- **digital signatures**
>- nonces
>- accurate timestamps
>
>A signature gives data-origin authentication, not freshness: the signed message can simply be recorded and replayed.

>[!question] Which parties are authenticated in this protocol?
>![[Pasted image 20261026164904.png|400]]
>- Alice and Bob
>- only Alice
>- **neither**
>
>Both messages can be produced by anyone: $n$ is sent in the clear, and $\{Alice, m, n\}_{pk_{Bob}}$ uses Bob’s public key. Neither party proves knowledge of a secret.

>[!question] Whose identity is authenticated here? (message freshness aside)
>![[Pasted image 20261026165115.png|400]]
>- Alice and Bob
>- **only Bob**
>- neither
>
>$\{n\}_{sk_{Bob}}$ could come only from him, while $\{Alice, m, n\}_{pk_{Bob}}$ uses Bob’s public key, which anyone can do. (Freshness aside: Bob signs a nonce he chose himself, so an old signature could be replayed - genuine mutual authentication has Alice choose the challenge).

---
## Needham-Schroeder mutual authentication protocol
![[Pasted image 20261026165632.png|500]]

This protocol was proposed in 1978 and it’s goal to create was to guarantee secrecy and authenticity of the nonces $n_{A}$, $n_{B}$, later used to derive a shared session key.

### The attack
The attack to this protocol was found only in 1996 by Gavin Lowe (18 years after publication). The aim of this attack is to make Alice believe she is talking to Bob, while Bob is actually running a legitimate session with Eve, encrypting his messages with Eve’s public key.

![[Pasted image 20261026170516.png|500]]

In this case Eve is able to learn both nonces and compute the session key of Alice and Bob.

### Needham-Schroeder-Lowe
To address this issue Lowe added Alice’s identity to the second message, so that Bob starts the session with Eve, but the response names Alice, and he aborts the connection.

![[Pasted image 20261026170644.png|500]]

### Designing and Implementing Protocols
Designing cryptographic protocols is highly error-prone, even for experts. New settings keep requiring new protocols (e.g., social networks, cloud computing) (and implementing them is equally error-prone).

The following are the principles of creating a secure protocol:
- don’t reuse keys: e.g., separate signing and decryption keys, and one session key per direction
- include identities, and ensure freshness
- hash everything: each message includes a (signed) hash of all previous messages, which makes cut-and-paste attacks harder

---
## Password-based authentication
The main question is how can Alice prove to the server’s, which stores Alice’s password or its hash, her identity?

A negative example of how to do its is to just send in plaintext a message containing her identity and password $\{Alice, pw\}$. In fact, in this case, the hashed password can be eavesdropped and resent unchanged (furthermore knowing $h(pw)$ lets the attacker the $pw$ offline).

### Challenge-response authentication
A solution would be to use unilateral password authentication with a challenge-response protocol. In this way the server sends a fresh nonce, Alice answers with a MAC keyed with password, and because the server knows $pw$ and $n$, he can verify the MAC, making replay not possible. Example of this are: CHAP, CRAM-MD5.

![[Pasted image 20261026172744.png|360]]

However and eavesdropper can still guess the password offline ($n$ and the response are known).

>[!bug] Deriving secure keys from passwords
>Passwords have too little entropy to be used directly as keys, so that an adversary who knows $n$ can guess passwords offline, and knows when a guess decrypts correctly.

### Password-Authenticated Key Exchange (background)
The goal **Password-Authenticated Key Exchange** (*PAKE*) is to turn a low-entropy password into a high-entropy session key, preventing offline dictionaries on the password, MitM attacks by authenticating the key exchange with the password, and providing forward secrecy (a later password leak does not reveal old session keys).

Examples of this are DH-EKE (run DH with values encrypted under the password), or SPEKE (used between terminal and chip of the German ID card).