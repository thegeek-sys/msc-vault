---
Class: "[[System Security]]"
---
---
## Reasoning about security
### What is a system?
A system is an interconnected collection of components organized to accomplish some function while operating within a defined trust boundary. It interacts with other systems and its environment: hardware, software, humans and the physical world.

![[Pasted image 20261024212146.png]]

Components within a system can also interact through a service interface. One example of this are the `Userland` programs that can request services from the kernel through `syscalls` (e.g., open a file).

### What does security mean?
We can divide security into three categories:
- **assets** → what we must protect; can include data or capabilities
- **risk** → measures the likelihood and consequences (impact) of an asset losing protection
- **risk analysis** → goes beyond technical bugs to include economic, legal, and mission factors

>[!info] Core rule
>You cannot secure a system until you identify its assets and their value

#### Computer security
Computer security is about protection of (information) assets. We must study how they can be compromised. Various other definitions exist. For example, computer security can be seen as “absence of unauthorized access to, or handling of, system state.”

#### Prevention, detection, reaction
- **prevention** → stop assets from being compromised, stolen, or modified (es. keep the private key unpredictable and hidden)
- **detection** → notice when, how, and by whom an asset was compromised (es. Monitor logs for unusual key usage)
- **reaction** → Contain the breach and restore the system to a secure state (es. replace the key and revoke trust)

>[!warning]
>Some harm is permanent: a leaked secret cannot be made secret again

### Threat modeling: the attacker

>[!question] Against whom? (who)
>- script kiddies → low skill, opportunistic 
>- cyber criminals → profit-driven
>- nation states, intelligence agencies

>[!question] With which capabilities? (how)
>- listen on the network or change traffic?
>- run code on the same machine?
>- physical access, time, money?

>[!info]
>Define the attacker model before designing defenses

### Security objectives: CIA
- **Confidentiality** → “Preserving authorized restrictions on information access and disclosure.”
- **Integrity** → “Guarding against improper information modification or destruction.”
- **Availability** → “Ensuring timely and reliable access to and use of information.”

There are also other secondary objectives:
- *reliability* → continuity of correct service, despite accidental failures
- *safety* → no catastrophic consequences for users and environment, even in adverse conditions
- *security* → prevent and detect unauthorized actions, in the presence of attacks (assumes an intelligent adversary)

#### Dependability and security
Unlike security, **dependability** concerns service that can justifiably be trusted

![[Pasted image 20261024213554.png]]

### Correct service, failure, error, fault
A system delivers **correct service** when it implements the desired system function. A **failure** is the event when the delivered service *deviates* from correct service. An **error** is the deviating part of the internal state. An error can remain invisible at the interface. A **fault** is the cause of an error, such as a bug or a malicious input. A dormant fault becomes active when it **causes an error**.

![[Pasted image 20261024213740.png]]