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
![[Pasted image 20261103130808.png|435]]

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

>[!warning]
>![[bangdesk.gif]]
>
>Il CAA serve solamente ai CA per capire chi è autorizzato ad emettere il certificato (si tratta però di un “consiglio”, un CA malevolo può fare come gli pare), non al client per capire se il certificato ottenuto è providato da un CA autorizzato dal server.
>
>Per fare in modo che ci sia un controllo da parte del client, bisogna usare DANE

### DNS-base Authentication of Named Entities (DANE)
DANE replaces reliance on CAs with reliance on DNS (required DNSSEC to sign DNS records, it’s an extention of DNS that adds digital signatures to verify authenticity and integrity).

The domain’s DNS publishes which certificate (or CA) is valid for the domain in a **TLSA record**, which is a specific leaf certificate or public key, or a specific CA and can also add custom trust anchors. The clients verify the TLSA record with DNSSEC, and reject certificates that don’t match.

It’s commonly used for email (SMTP) between servers, but not supported by browsers.

>[!example]
>1. Il proprietario pubblica nel DNS (firmato con DNSSEC):
>	`_443._tcp.banca.it  TLSA  <hash del certificato giusto>`
>2. Il client si connette a banca.it e riceve un certificato nell'handshake
>3. Il client chiede il TLSA al DNS e verifica la firma DNSSEC
>4. Confronta: il certificato ricevuto corrisponde al TLSA?
>      sì → procede
>      no → rifiuta, anche se la catena di CA è valida

### Certificate pinning
The idea of certificate pinning is that the client remembers (“pins“) the expected certificate or public key of a server and rejects everything else (even if it’s a valid certificate signed by a trusted CA).

For websites we have **HTTP Public Key Pinning** (*HPKP*) which is used by the server to send pins in an HTTP header:
```
Public-Key-Pins: max-age=2592000;
	pin-sha256="E9CZ9INDbd+2eRQozYqqbQ2yXLVKB9+xcprMF+44U1g=";
	pin-sha256="LPJNul+wow4m6DsqxbninhsWHlwfp0JecwQzYpOLmCQ="
```

>[!note]
>The second pin is for backup; to change the key without blocking clients

However there are few drawbacks to this approach:
- **trust on first use** → by using HPKP the client trust whatever he receives the first time he connects; if in that moment he is talking with an attacker, the client pins the wrong key
- difficult to manage → if you lose the pinned key and you don’t have any backup, you block your own users until the expiration of the key
- can be abused by attackers → who temporarily compromises a server can pin it’s own key 

In Chrome it’s deprecated since 2018

### Certificate Transparency (CT)
The idea behind **Certificate Transparency** is to make every issued certificate publicly visible, so that a CA cannot issue a certificate for a domain without the domain owner being albe to see it (anyone can audit and monitor which certificates were issued, mistakenly or maliciously).

![[Pasted image 20261104111709.png|311]]

To do so, we use *public*, *append-only logs* based on Merkle hash trees. Each CA submits each certificate to CT log server, and the log returns a *Signed Certificate Timestamp* (SCT) and a promise to include the certificate in the log. The SCT is embedded in the certificate as an X.509v3 extension, so the proof of logging travels with the certificate; the CA first submits a precertificate to get the SCTs, then issues the final certificate containing them.

>[!info]
>Since April 2018, Chrome rejects new certificates without SCTs

Domain owners (or monitors) see new certificates for their domains (e.g. an unauthorized CA issues a certificate for their domain) and can react (e.g., request revocation). However CT **detects** rogue certificates; it does *not prevent* them.

#### Verifying the SCT
The clients know the log servers and their public keys (log lists, e.g., by Google, Apple) so they can verify the SCT signatures.

As a further measure of security usually policies *require multiple SCTs* from independent logs, to reduce the risk of a log colluding with a CA (e.g. Chrome requires 2 or 3 SCTs depending on the certificate lifetime).

Thanks to all of this suspect certificates should be detected quickly and revoked and auditors also check that logs behave correctly (append-only, consistent views).

>[!warning]
>Logs are only queried by monitors (to detect certificates issued for their domains) and auditors (to check the logs behave correctly), not by browsers during the handshake.
>
>Browsers just have to verify the signature using the log public key.

#### Privacy issues
However the public log reveals internal structure and services of a network (as a certificate is issued for a new URL it gets added to the log), so an attacker might try to compromise a server before the admin finished the setup.

---
## Certificate verification
The client is responsible for verifying the certificate: **TLS security depends on correct verification by the client**. The checks are:
- the chain leads to a trusted root CA, indeed each certificate is signed by the next one up and the root is pre-installed and self-signed
- all certificates are valid, not expired and not revoked
- the SAN (or CN) of the leaf matches the server contacted
- further requirements (e.g., key usage flags)

If there is **any error**, the verification fails.

>[!note] CA certificates must outlive the certificates they issue
>If the CA certificate expires before the end-entity (leaf) certificate, the chain is no longer valid from that moment, so the leaf gets rejected even if its own `not after` hasn't been reached yet. Revocation information (CRLs, OCSP responses) is signed by the CA, so it can no longer be verified either.
>
>That's why root and intermediate CA certificates have much longer validity (years) than end-entity ones (at most 398 days, and shrinking).

### Non-browser software
Browsers, as the primary way to access the web, have become very good at certificate verification, but non-browser software often does not:
- Amazon's EC2 Java library, PayPal's merchant SDKs, AdMob code, Chase mobile banking, and many more had **completely broken certificate validation**
- mobile apps are similarly broken

Typical causes: **checks disabled during development** (and never re-enabled) and **misunderstood library APIs**.
