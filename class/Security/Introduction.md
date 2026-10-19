---
Class: "[[Security]]"
---
---
## What is security?
A system is _secure_ with respect to: 
- a **system model** → what the system is and does
- an **attacker model** → what the attacker can do
- **security goals** → what must be protected
	- **confidentiality** → only authorized parties can learn information
	- **integrity** → unauthorized modification is prevented or deleted
	- **availability** → the system or service is usable when needed

Other than the mentioned goals there are further goals:
- authentication → spits into entity authentication (who am I talking to) and data-origin authentication (who produced this message)
- non-repudiation → a party cannot deny an action later
- privacy → the ability of individuals to control information about themselves
- anonymity → an individual cannot be identified within a set of subjects (the anonymity set)

>[!warning] Authentication implies integrity
>Integrity alone does not identify a sender

---
## Model
Security arguments require a well-defined, open system, security properties, and attacker model (no “security by obscurity”).

### Attacker model
We can define two different types of attacker:
- **passive attacker** → observes, but does not change the system (e.g. eavesdropping)
- **active attacker** → modifies, injects, impersonates or disrupts

Of course the attacker will need an **attack surface** (where can the attacker interact with the system?). Let’s give some examples:
- network (protocol vulnerabilities, spoofing, intrusion)
- hardware (cold boot, probing, side channels, hardware trojans)
- software (vulnerabilities is OS or application code, logic flaws)
- humans (social engineering, errors, insiders)
- interfaces between layers (e.g. Rowhammer and Spectre attack hardware from software)

---
## Defenses and Security Guarantees
In order to avoid attacks it’s possible to adopt some protective measures:
- **prevention** → stop assets from being compromised
- **detection** → notice when, how, and by whom an asset was compromised
- **reaction** → recover from the damage (which may be irreversible)

However it’s not possible to guarantee 100% security, because security can only be proven with respect to an attacker model and because it can be falsified by a single successful attack.

