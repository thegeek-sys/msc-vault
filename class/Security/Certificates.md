---
Class: "[[Security]]"
Sem: WiSe 26/27
---
---
## Introduction
The server (and optionally the client) is authenticated with a **certificate** which binds a public key to an identity (for the web it corresponds to it’s domain name). The certificate is essential for [[TLS]], indeed, without authenticity, confidentiality and integrity are “useless” (we might have a perfectly secure channel to the attacker).

The two feature we want from a certificate are:
- authenticate our communication partner
- obtain its public key

>[!question] What does it mean in HTTPS?
>- nobody can impersonate the web server
>- we can verify the server’s signature on its (EC)DH parameters (or encrypt the pre-master secret)

### Terminology
**Common Name** (*CN*) and **Subject Alternative Name** (*SAN*) are field of **X.509** used to determine for which domains a certificate is valid. The CN is contained in $Subject$ field of the certificate (e.g., `CN=cispa.de`), and it was typically used to put the domain name (today is legacy).

The SAN is an extension of the certificate with a list of valid names (e.g., `cispa.de`, `www.cispa.de`), wildcard (e.g., `*.cispa.de`), IP addresses, and email.

### X.509 (background)
![[Pasted image 20261103130808.png|435]]o

**What it is:** the standard (ITU-T, first version 1988) that defines the **format of public-key certificates**. The Internet profile is **RFC 5280**. It is used in TLS, IPsec, S/MIME, code signing and more.

**Versions:** v1 → v2 → **v3**. Version 3 adds **extensions** and is the one used today.

**Main fields of a certificate:**
- **Version** and **serial number**: identify the format version and this specific certificate
- **Signature algorithm**: the algorithm the CA used to sign
- **Issuer name**: the CA that issued the certificate
- **Validity**: `not before` / `not after`
- **Subject name**: who the certificate belongs to (e.g. CN)
- **Subject public key info**: the algorithm and the public key itself
- **Issuer / Subject unique identifier**: optional, rarely used (v2)
- **Extensions** (v3 only): e.g. **SAN** (the list of valid names), key usage
- **Signature**: made by the **CA's private key** over all the fields above

**Key point:** a certificate binds an identity (subject) to a public key, and the CA's signature vouches for that binding. The signature is the CA's, not the server's.

---
## Certificate Authorities and PKI
The idea is that you cannot trust every server of the world, but you can trust a small number of CA. Each CA signs the certificates of who is below him, so the trust goes from one signature to the other.

There are three typical levels:
1. Root CA
	- the certificate is self-signed (the issuer matches the subject)
	- it is the *trust anchor* → it isn’t verified with the signature, but it’s already inside the OS or browser
2. 