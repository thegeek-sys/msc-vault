---
Class: "[[Security]]"
---
---
## Index
- [[#TLS’ structure|TLS’ structure]]
	- [[#TLS’ structure#Record protocol|Record protocol]]
	- [[#TLS’ structure#Handshake protocol|Handshake protocol]]
		- [[#Handshake protocol#Security capabilities|Security capabilities]]
		- [[#Handshake protocol#Server authentication and key exchange|Server authentication and key exchange]]
		- [[#Handshake protocol#Client authentication and key exchange|Client authentication and key exchange]]
		- [[#Handshake protocol#Finish|Finish]]
- [[#Forward secrecy|Forward secrecy]]
- [[#Traffic analysis|Traffic analysis]]
- [[#TLS 1.3|TLS 1.3]]
- [[#TLS 1.3|TLS 1.3]]
	- [[#TLS 1.3#Session resumption (PSK) and 0-RTT|Session resumption (PSK) and 0-RTT]]
- [[#Attacks on SSL/TLS|Attacks on SSL/TLS]]
	- [[#Attacks on SSL/TLS#Downgrade attacks|Downgrade attacks]]
	- [[#Attacks on SSL/TLS#Flame (2012, background)|Flame (2012, background)]]
	- [[#Attacks on SSL/TLS#Storm-0558 (2023, background)|Storm-0558 (2023, background)]]
	- [[#Attacks on SSL/TLS#Bad Randomness: Debian OpenSSL (2006-2008)|Bad Randomness: Debian OpenSSL (2006-2008)]]
	- [[#Attacks on SSL/TLS#Padding Oracles: POODLE (2014)|Padding Oracles: POODLE (2014)]]
	- [[#Attacks on SSL/TLS#Further SSL/TLS Vulnerabilities (background)|Further SSL/TLS Vulnerabilities (background)]]
- [[#HTTP Strict Transport Security|HTTP Strict Transport Security]]
---
## Introduction
**TLS** (*Transport Layer Security*), whose former versions were called Secure Sockets Layer (SSL), is de facto the standard for secure communication.

Its primary goal is to provide privacy and data integrity between two communication applications, and is deployed:
- web browsers (HTTPS), e-mail, VoIP
- VPNs
- payment terminals
- IOT devices

>[!done] Guaranees
>- end-to-end secure communication  in the presence of an active or passive network attacker
>- it’s built assuming that the attacker may owns the entire network (DHCP, DNS, and all Internet communication) and can read, modify, and inject any packet

>[!warning]
>Some of the classic public wifi login pages still use HTTP (no TLS)

---
## TLS’ structure
The core of TLS consists of two different protocols, used in the following order (in reality, there are more sub-protocols and extensions):
- **handshake protocol** → uses public-key cryptography to establish shared secret keys between client and server
- **record protocol** → uses these keys to protect confidentiality, integrity, and authenticity of the exchanged data

### Record protocol
![[Pasted image 20261029164929.png|center|408]](in the image it says MAC but it’s HMAC actually)

This protocol is used to encrypt, decrypt, and check the integrity of the data.
- confidentiality → granted via *symmetric encryption* with a shared key
- integrity → granted via *HMAC* with a different shared key

Both the keys are derived from the **master secret** of the handshake. This protocol is also used for fragmentation of application data and compression (optional, and a bad as it leaks data about the plaintext).

TLS up to version 1.2 uses [[Symmetric Cryptography#^2c6fc5|MAC-then-encrypt]] but it enabled attacks such as POODLE and Lucky 13. For further versions (TLS 1.3) it’s only used authenticated encryption ([[Symmetric Cryptography#Authenticated encryption with associated data (AEAD)|AEAD]]).

### Handshake protocol
The handshake protocol is used to:
- *negotiate* the protocol version and the cryptographic algorithms (*cipher suite*)
- *authenticate the server* and optionally the *client* using certificates to acknowledge the public keys and verify the identity (in HTTPS usually only the server is authenticated, in VPNs both)
- *establish a shared secret*

![[Pasted image 20261029170300.png|260]]

It consists of four phases:
1. **establish security capabilities**
2. **server authentication and key exchange**
3. **client authentication and key exchange**
4. **finish**

#### Security capabilities
During this phase client and server exchange nonces and agree on a TLS version

![[Pasted image 20261029171733.png|260]]

- **Client Hello**: the client sends
	- its highest supported TLS version
	- a client nonce
	- supported cipher suites (combination of algorithms it can use)
	- a session ID (zero for a new session)
- **Server Hello**: the server replies with
	- the chosen TLS version and cipher suite
	- a server nonce
	- the session ID (copied from the client, or a new one)

The nonces guarantee the *freshness* of the handshake, and are then used for the derivation of the master secret.

>[!info] Session and resumption
>Multiple TLS connections can share a session ID (and its key) so that resuming a session avoids the expensive public-key part of the handshake.
>
>This mechanism applies to HTTPS where many TCP connections to the same server share one TLS connection, so usually there is no client certificate (client is not authenticated, there is no client’s identity to reverify each time).

#### Server authentication and key exchange

![[Pasted image 20261029172633.png|260]]

This message contains:
- **ServerCertificate** → the server’s X.509 certificate (contains its $pk$, server name, and a CA signature)
- **ServerKeyExchange** → first part of the key exchange
	- DH/ECDH (elliptic curve DH) → the server’s DH parameters, signed with the certificate’s private key (ex-novo for each session); the public value $g^{y}$
	- RSA key exchange → nothing sent (removed in TLS 1.3)
- **CertificateRequests** → asks for the client’s certificate (only for client authentication)

#### Client authentication and key exchange

![[Pasted image 20261029172731.png|260]]

This message contains:
- **ClientCertificate** → only if requested
- **ClientKeyExchange** → second part of the key exchange
	- DH/ECDH → the client sends its public value $g^{x}$; now both can compute $g^{xy}$, that becomes the *pre-master secret* (**PMS**)
	- RSA → the client generates a random 48 byte PMS and sends it encrypted with the $pk$ of the server
- **CertificateVerify** → the client proves it owns the certificate by signing the handshake with its private key

Now both sides compute the *master secret* (**MS**)

>[!info] Deriving the keys (background)
>The pre-master secret depends on the key exchange
>- RSA → chosen by the client and sent encrypted
>- DH/ECDH → the shared DH value $g^{xy}$; the servers signs its DH contribution
>
>Master secret (48 bytes): $MS=PRF(PMS, \text{"master secret"}, N_{C}||N_{S})$
>From the MS, the PRF derives all further keys: MAC keys, encryption keys, and IVs for both directions.
>
>Key derivation function (KDF) is a pseudorandom function that derives (longer) keys from a secret value, usually built from a hash function (e.g. HMAC).
>
>To obtain the shared key using DH/ECDH the client has $x$ (he generated it), so acknowledges $g^{y}$ (sent by server) and computes $(g^{y})^{x}=g^{xy}$. Viceversa for the server.

#### Finish

![[Pasted image 20261029173631.png|260]]

This message contains:
- **ChangeCipherSpec** → both sides agree on switching to the negotiated keys and ciphers
- **Finished** → a MAC over all previous handshake messages, under the new keys (detects any manipulation of the plain handshake)

---
## Forward secrecy
Assuring **forward secrecy** means that compromising long-term secrets does not compromise past session keys (attacker may record all traffic of a serverm hoping to steal the server’s key later).

>[!question] Why is DH exchange better than RSA?
>With RSA key exchange the PMS travels encrypted with the server’s public key. If an attacker recorder the whole traffic, in future may obtain the private key and decrypt it.
>
>By using ephemeral DH (DHE/ECDHE) for every session are generated new DH values. In such way, the keys cannot be computed from the recorded DH values (due to the discrete log) and stealing the server’s key later does not help.
>
>For this reason RSA key exchange was removed in TLS 1.3 (has no forward secrecy)

---
## Traffic analysis
It’s possible to obtain quite few information by analyzing the traffic:
- **server identity**, through the certificate; in TLS 1.2 the certificare is sent in clear
- the **Server Name Indication** (SNI); in HTTPS the client declares to which domain it want to talk to because the same IP and port may host multiple services (can be found in the Client Hello)
- **message length**; the encryption hides the content but not the size: if you know all the media of a website, the sequence of the dimensions may reveal which media the client accessed

---
## TLS 1.3
In version 1.3 the handshake make just one roundtrip instead of two. In particular the client guesses the key exchange and sends its key share right away and the server replies with its share, and can encrypt immediately. Finally the *server certificate is sent encrypted* (even if the SNI is made public during Client Hello the more is encrypted the better it is).

![[Pasted image 20261029184647.png|289]]

RSA key exchange was removed so that only (EC)DHE is possible, making forward secrecy a default. Only AEAD ciphers are allowed (no compression, no static RSA, no CBC)

### Session resumption (PSK) and 0-RTT
After a full handshake the server sends a **session ticket** (session state encrypted with a server-only key) and a shared **PSK**. On the next connection the client presents the ticket, and certificate and signatures are skipped.

**0-RTT** (optional): the client sends data together with the ClientHello, without waiting for the server. Drawbacks:
- **no forward secrecy**: early data is encrypted only with the PSK-derived key.
- **replayable**: an attacker can resend the same message and the server accepts it again.

Use it only for idempotent requests (e.g. `GET`).

---
## Attacks on SSL/TLS
### Downgrade attacks
![[Pasted image 20261029185617.png|340]]

A MitM modifies the unencrypted hello messages to force weak cipher suites or old versions (available for interoperability, the server accepts the best it is offered). By forcing older version would be possible to use SSL 2.0 which has no Finished messages (which does the MAC of all the handshake, introduced with SSL 3.0).

However nowadays it’s not possible to downgrade TLS to such an old version like SSL 2.0 (removed from machines in 2011) neither to use vulnerable cipher suites. The most renowned attacks of this kind are FREAK and Longjam (2015) which forced the use of 512 bit “export-grade” keys, so weak that the attacker was able to break them during the handshake and to falsify even the Finished messages.

### Flame (2012, background)
Flame was cyber-espionage malware signed with a fake intermediate CA certificate that appeared to be issued by Microsoft, so Windows Update accepted it. The certificate was forged with an MD5 chosen-prefix collision against an obscure Microsoft certificate that still used MD5 and allowed code signing. The collision technique was new, hinting at state-level cryptanalysis.

### Storm-0558 (2023, background)
A China-linked actor obtained a Microsoft token-signing key, which had leaked years earlier in a crash dump. A validation flaw let the key sign both consumer and enterprise tokens, so the attackers forged authentication tokens and read the email of ~25 organizations, including US government agencies. Same lesson as Flame: one leaked or forgeable signing key breaks trust for everyone who relies on it, so signing keys must be protected (HSMs), rotated, and constrained in what they can sign.

### Bad Randomness: Debian OpenSSL (2006-2008)
A single line of code was commented out (`MD_Update(&m,buf,j); /* purify complains */`), so the random number generator was seeded only with the process ID. The seed had only ~15 bits of entropy (Linux default PID limit: 32768), collapsing the keyspace to a few thousand possible keys. All keys generated with Debian's OpenSSL in that period were predictable: SSH, OpenVPN, DNSSEC, TLS certificates and session keys.

> [!note] A Note on Randomness
>An Internet-wide scan of TLS and SSH keys (2012, Heninger et al.) found that vulnerable keys are surprisingly common: some keys are shared because of insufficient entropy, or are default "snake-oil" keys.
> 
>Researchers recovered the private keys of TLS and SSH hosts whose RSA moduli shared a prime factor. Causes: specific software, e.g. a boot-time entropy hole in the Linux random number generator (`/dev/urandom` needs to "warm up").
>
>Weak key-generation randomness has broken real Certificate Authorities too (see the Certificates lecture).

### Padding Oracles: POODLE (2014)
A **padding oracle** lets the attacker tell whether a manipulated ciphertext decrypts to valid padding (e.g. different error messages or timing). In SSL 3.0 the padding bytes can have arbitrary content except the last one, so the server's reaction leaks one byte per attempt, allowing plaintext recovery (e.g. cookies). Attackers first force a fallback to SSL 3.0 (downgrade). Fix: require and check specific padding content (TLS 1.0+); the real cure is AEAD, and TLS 1.3 removes CBC entirely.

### Further SSL/TLS Vulnerabilities (background)
- BEAST (2011) → chosen-plaintext attack on CBC in TLS 1.0, based on predictable IVs
- CRIME (2012) → compression oracle; mix attacker-chosen data with secrets and observe the compressed length
- Lucky 13 (2013) → timing-based padding oracle on CBC mode
- RC4 (2015) → shown to be weak, "only" 9 · 227 requests to steal a cookie
- DROWN (2016) → abuse servers that still support SSL 2.0 to decrypt TLS sessions

---
## HTTP Strict Transport Security
An active attacker does not necessary need to break TLS, he simply needs to drop all traffic to port 443 (used by HTTPS), so that the user is forced to use HTTP.

The solution is to use *HSTS*. With this the server sends an header in its HTTPS replies:
```
Strict-Transport-Security: max-age=<secondi>; includeSubDomains
```

In this way the client is required to never connect through HTTP for the next `max-age` seconds. If HTTPS does not work or the certificate is wrong, it doesn’t allow to continue.

However this header arrives only with an HTTPS reply, so the browser needs to have visited the website at least once in a safe way. For this reason the browsers implemented a list of domains that require HSTS, while mobile platforms use HTTPS by default for the app’s connections.

