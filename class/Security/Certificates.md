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
The idea is that you cannot trust every server of the world, but you can trust a small number of CA. Each CA signs the certificates of who is below him, so the trust goes from one signature to the other. So a **public key infrastructure** (*PKI*) enables transitive trust.

![[Pasted image 20261103162843.png|403]]

### Chain of trust
In summary the root CA signs certificates of intermediate CAs, which sign lower-level certificates, but the CA must verify the requester’s identity and domain ownership.

![[Pasted image 20261103163106.png]]

There are three typical levels:
1. Root CA
	- the certificate is self-signed (the issuer matches the subject)
	- it is the *trust anchor* → it isn’t verified with the signature, but it’s already inside the OS or browser
2. Intermediate CA
	- the certificate is signed by the root (with private key)
3. End-entity certificate
	- it’s the server’s certificate (`cispa.de`) signed by the intermediate
	- this contains the SAN with the valid names

>[!question] What does the client do?
>- the server sends in $Certificate$ message its own certificate plus the intermediate ones
>- the client verified from bottom to top
>	- the leaf’s signature is valid with intermediate’s public key
>	- the intermediate’s signature is valid with root’s public key
>	- the root is in its own root store, so it stops
>- for each step it check also the expiration, revoke, and that the certificate is authorized to sign other certificates (it’s a CA)

### Issuing a new certificate
To issue a certificate, CAs have to validate the domain. There are two possible validations:
- **domain validation** (*DV*) → the requester proves control over the domain (e.g. Let’s Encrypt issues fully automatically after an HTTP- or DNS-based check)
- **extended validation** (*EV*) → stricter checks of the organization; the idea is to protect users from phishing domains with valid DV certificates (browsers used to highlight EV in the address bar, but stopped in 2019 - legacy today)

---
## Revocation
There could be multiple reasons why a certificate can be revoked:
- the public key was compromised
- the public key is weak (e.g. generated with low entropy)
- the owner stopped paying the CA
- the CA’s own certificate was compromised (e.g. the Flame attack)
- the domain was transferred to a new owner

Expiration can be considered as a form of revocation too, but many deployed systems do not bother checking revocation.

### Revocation mechanisms
#### CRLs

![[Pasted image 20261103171601.png|232]]

The **certificate revocation list** (*CRL*) is a signed list of revoked ceritificates published periodically by the CA. The certificate itself contain the URL of where to find the CRL, but we often talk about “delta CRLs” which only contain the updates of the previous CRL (with the newly revoked certificates).

However there are some issues:
- CRLs can be long (thousands of entries)
- are updated only every few days
- if the CRL cannot be fetched, clients usually accept the certificate (“soft fail”)

#### OCSP
With the **Online Certificate Status Protocol** (*OCSP*) is the client that asks the CA about one certificate (oc the response is signed).

Also for this method there are some drawbacks:
- high load on OCSP servers
- if the server does not answer, clients usually connect anyway
- the OCSP server learns which sites you visit (privacy leak)

This method is also used beyond TLS, e.g., to check app signatures in macOS (in 2020 an OCSP outage slowed down app launches worldwide).

#### OCSP stapling
The TLS server regularly fetches a signed OCSP response for its own certificate, and the response is valid for a limited amount of time (it is “stapled” to the TLS handshake).

Here’s the benefits:
- less load on the OCSP server
- no privacy leak

However we have a minor drawback: revocation takes effect with a slight delay. Furthermore, can be used a certificate extension called *Must-Staple* making a stapled response mandatory

>[!example]
>![[Pasted image 20261103172616.png|400]]

### The trend: short-lived certificates
However revocation has never worked well in practice as browsers do little online checking (e.g. Chrome uses its own curated revocation lists), and for example Let’s Encrypt even shut down its OCSP service in 2025 in favor of CRLs.

For these reasons the trend nowadays is basically to make certificates expire quickly, but of course this would require fully automated renewal (ACME).

---
## Rogue CAs and countemeasures
The weakest link in PKI are the rogue or compromised CAs:
- DigiNotar (2011): over 500 certificates issued to an unknown attacker, including `*.google.com`, used for MitM attacks in Iran
- TürkTrust (2012): accidentally issued intermediate CA certificates, used to issue certificates for `*.google.com`
- Comodo (2015): issued a certificate for `live.fi` (Microsoft) to an outsider
- Comodo (2016): issued certificates for foreign domains due to an OCR error when parsing WHOIS data
- Symantec (2015–2017): "test" certificates for google.com, up to 30,000 certificates without proper validation (Chrome stopped trusting Symantec roots - incl. VeriSign, Thawte - in 2018)
- Entrust (2024): repeated compliance failures, distrusted by Chrome for new certificates

There are few approach to avoid this:
- DNS Certificate Authority Authorization (CAA)
- DNS-base Authentication of Named Entities (DANE)
- Certificate pinning
- Certificate Transparency (CT)

### DNS Certificate Authority Authorization (CAA)
The CAA consist of a DNS record published by the domain’s owner listing which CAs may issue certificate for a domain.
- `0 issue "letsencrypt.org"` → Let’s Encrypt may issue certificates
- `0 issue ";"` → no CA may issue certificates
- `0 issuewild "letsencrypt.org"` → Let’s Encrypt may issue wildcard certificates
- `0 iodef "mailto:admin@domain.com"` → where to report policy violations

>[!warning]
>Records are additive: two `issue` entries allow both CAs

It falls back to the parent domain if no record exists, and CAs must check CAA but it cannot be enforced technically.

>[!example]
>```
>trust.cispa.de CAA 0 issue     "letsencrypt.org"
>cispa.de       CAA 0 issue     "pki.dfn.de"
>cispa.de       CAA 0 issuewild ";"
>nossl.cispa.de CAA 0 issue     ";"
>nossl.cispa.de CAA 0 iodef      "mailto:it@cispa.de"
>```
>
>1. Let’s Encrypt can issue regular certificates for `trust.cispa.de`
>2. DFN can issue regular certificates for `cispa.de`
>3. Nobody can issue a wildcard certificate for `cispa.de`
>4. Nobody can issue certificates for `nossl.cispa.de`
>5. Violations of (4) should be reported to it@cispa.de

### DNS-base Authentication of Named Entities (DANE)
DANE replaces reliance on CAs with reliance on DNS (required DNSSEC to sign DNS records, it’s an extention of DNS that adds digital signatures to verify authenticity and integrity).

The domain’s DNS publishes which certificate (or CA) is valid for the domain in a **TLSA record**, which is a specific leaf certificate or public key, or a specific CA and can also add custom trust anchors