---
Class: "[[Security]]"
Sem: WiSe 26/27
---
---
## Index
- [[#Static and dynamic analysis|Static and dynamic analysis]]
    - [[#Static and dynamic analysis#Soundness vs completeness|Soundness vs completeness]]
    - [[#Static and dynamic analysis#Static analysis|Static analysis]]
    - [[#Static and dynamic analysis#Dynamic analysis|Dynamic analysis]]
        - [[#Dynamic analysis#AddressSanitizer (ASan)|AddressSanitizer (ASan)]]
- [[#Fuzzing|Fuzzing]]
    - [[#Fuzzing#From dumb to smart|From dumb to smart]]
    - [[#Fuzzing#Coverage-guided fuzzing|Coverage-guided fuzzing]]
    - [[#Fuzzing#Fuzzing beyond userspace (background)|Fuzzing beyond userspace (background)]]
    - [[#Fuzzing#Rehosting embedded firmware|Rehosting embedded firmware]]
- [[#Symbolic execution|Symbolic execution]]
    - [[#Symbolic execution#Limits|Limits]]
        - [[#Limits#Concolic execution|Concolic execution]]
    - [[#Symbolic execution#angr|angr]]
- [[#Reverse engineering|Reverse engineering]]
    - [[#Reverse engineering#Binary diffing|Binary diffing]]
- [[#AI-assisted bug finding|AI-assisted bug finding]]
    - [[#AI-assisted bug finding#Autonomous agents|Autonomous agents]]
    - [[#AI-assisted bug finding#Caveats|Caveats]]
- [[#The bug-finding toolbox, in a glimpse|The bug-finding toolbox, in a glimpse]]
---
## Introduction
In the previous lectures we saw that attacks exploit bugs and that mitigations make exploitation harder; the question now is **how do we find the bugs in the first place?**

There are several approaches:
- **humans reading code** → manual audit (effective, but expensive and doesn't scale)
- **static analysis** → reason about the code without running it
- **dynamic analysis** → run the code and observe what happens (sanitizers, fuzzing)
- **symbolic execution** → reason about which inputs reach which program states
- **reverse engineering** and **binary diffing** → when only the binary is available
- increasingly, **AI-assisted tooling**

The stakes are high: critical memory-safety bugs still hide in mature and ubiquitous code.

> [!example] PHP2Shell (2026) CVE-2026-109618/-109619
> unauthenticated **remote code execution** caused by memory-lifetime bugs (use-after-free style) in the PHP runtime, reachable over the web and exploited **before** public disclosure (i.e. a 0-day).
> 
> This is exactly the bug class that sanitizers and fuzzing are built to catch.

---
## Static and dynamic analysis
**Program analysis** is the process of determining properties of a program's behavior (correctness, robustness, liveness, security, ...).

There are two broad families:
- **static analysis** → analyze the code **without executing it**
- **dynamic analysis** → analyze the program **while it runs**

Given a property $\varphi$ we want to check (e.g. "no out-of-bounds access"), deciding $\varphi$ exactly is in general **undecidable** (Rice's theorem), so every tool **approximates** and is necessarily wrong in one direction.

> [!info] Rice's theorem (background)
> Any non-trivial semantic property of programs (i.e. a property of _what the program does_, not of how it's written) is undecidable. It's a generalization of the halting problem: if we could decide "this program never accesses out of bounds", we could also decide whether a program halts.

### Soundness vs completeness
A tool checking property $\varphi$ can err in one of two ways:
- **sound** (_over-approximate_) → never misses a real violation (**no false negatives**), but may report bugs that can't actually happen (**false positives**, "may cry wolf")
- **complete** (_under-approximate_) → never reports a false alarm (**no false positives**), but may miss real bugs (**false negatives**)

| |Sound|Complete|
|---|---|---|
|approximation|over|under|
|guarantees|no false negatives|no false positives|
|downside|may cry wolf|may miss bugs|

You usually can't have both, so tools pick a side. In practice bug-finding tools often favor **few false positives**, since developers quickly ignore noisy tools.

> [!warning] Be careful
> In the literature the meaning of _sound_/_complete_ depends on the point of view (verification vs bug finding), and some papers use them the other way around. For this course: **sound = no missed bugs**, **complete = no false alarms**.

### Static analysis
**Static analysis** analyzes the source (or the binary) without running it, with techniques like:
- **data-flow analysis** → track how values propagate through the program
- **taint analysis** → mark data coming from untrusted _sources_ (e.g. `recv`, `argv`) and flag it if it reaches a dangerous _sink_ (e.g. `strcpy`, `system`, `printf` format) without sanitization
- **abstract interpretation** → execute the program over abstract domains (e.g. intervals: "`i` is in $[0, 10]$") instead of concrete values
- **model checking** → exhaustively explore the state space of a model of the program

It finds uninitialized variables, null-pointer dereferences, tainted-input flows and some overflows.

Pros and cons:
- finds bugs **early** (at development time) and can in principle cover **all paths**
- can produce **false positives** and misses bugs that depend on runtime values
- often requires the **source code**

Examples: compiler warnings (`-Wall -Wextra`), `clang-tidy`, Coverity, Infer, CodeQL

> [!info] Capiamo
> ![[bangdesk.gif]]
> 
> Esempio di taint analysis:
> 
> ```c
> char buf[64];
> char *name = getenv("USER");   // SOURCE: dato controllato dall'utente (tainted)
> strcpy(buf, name);             // SINK: copia senza controllo della lunghezza → WARNING
> ```
> 
> L'analizzatore non sa quanto sarà lunga `name` a runtime, ma vede che un dato "sporco" arriva in un sink pericoloso senza passare per un controllo, quindi lo segnala.
> 
> Il problema è che se in mezzo ci fosse un controllo che l'analisi non riesce a capire (es. una funzione di validazione complessa), il warning sarebbe un **falso positivo**.

### Dynamic analysis
**Dynamic analysis** analyzes the program while it runs on **concrete inputs**:
- it only sees the paths actually executed (**under-approximate**)
- **no false positives**: a reported crash is a real crash

The main problem is that it **needs inputs that trigger the buggy behavior**, so it is combined with **fuzzing** to generate those inputs.

**Sanitizers** instrument the program to detect errors at runtime that otherwise would go unnoticed.

#### AddressSanitizer (ASan)
**AddressSanitizer** (_ASan_) is a memory-error detector for C/C++, implemented as a compiler pass (LLVM/GCC, enabled with `-fsanitize=address`).

It detects:
- **use-after-free** (dangling-pointer dereference)
- **buffer overflows** on heap, stack and globals
- **use-after-return**, **double-free**, **memory leaks**

It works with **shadow memory** and **poisoned redzones**:
- each 8 bytes of application memory map to 1 **shadow byte** that marks them valid or poisoned
- allocations are surrounded by **redzones** that are poisoned, and freed memory is poisoned too (and kept in a _quarantine_ for a while before being reused)
- the compiler inserts a check of the shadow byte before **every** load/store, and an access to poisoned memory aborts with a detailed report

The overhead is ~**2x slowdown** (plus extra memory), so it's a **testing tool**, not a production defense (compare with [[Mitigations#ARM Memory Tagging Extension (MTE)|MTE]], which does a similar job in hardware and can stay on in production).

Siblings:
- **MSan** → reads of uninitialized memory
- **UBSan** → undefined behavior (signed integer overflow, invalid shifts, misaligned pointers, ...)
- **TSan** → data races between threads

> [!info] Capiamo
> ![[bangdesk.gif]]
> 
> Perché serve ASan? Perché molti bug di memoria **non fanno crashare** il programma: un overflow di 1 byte sull'heap sovrascrive il chunk vicino e il programma va avanti come se niente fosse. Senza sanitizer il bug resta invisibile (anche al fuzzer!).
> 
> Il mapping shadow è semplicissimo:
> 
> ```
> shadow_addr = (addr >> 3) + OFFSET
> ```
> 
> e il valore del byte shadow dice:
> 
> - `0` → tutti gli 8 byte sono accessibili
> - `k` (1..7) → solo i primi `k` byte sono accessibili
> - negativo → memoria avvelenata (redzone, memoria liberata, ...)
> 
> ```c
> int *p = malloc(10 * sizeof(int));
> free(p);
> return p[3];   // use-after-free
> ```
> 
> Con ASan:
> 
> ```
> ==1234==ERROR: AddressSanitizer: heap-use-after-free on address 0x6040000000dc
> READ of size 4 at 0x6040000000dc thread T0
>    #0 main test.c:5
> freed by thread T0 here: ...
> previously allocated by thread T0 here: ...
> ```
> 
> Il report dice cosa è successo, dove, e anche dove la memoria era stata allocata e liberata.

---
## Fuzzing
**Fuzzing** is automated testing: feed **invalid, unexpected or random input** to the program and watch for crashes:
- cheap, scalable and needs little knowledge of the program
- a crash (segfault, ASan report, failed assertion) flags a bug

The term was coined by **Miller** (1988): random "fuzz" input (noise on a dial-up line during a storm gave him the idea) crashed a large fraction of common Unix utilities.

Today it's the **workhorse** for finding memory-safety bugs: Google's **OSS-Fuzz** runs fuzzers continuously on open-source projects and has found tens of thousands of bugs.

To fuzz a library you write a **harness**: a small function that takes the fuzzer's bytes and passes them to the code under test.

```c
// libFuzzer harness
int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    parse_image(data, size);   // function under test
    return 0;
}
```

> [!warning]
> Fuzzing and sanitizers go together: the fuzzer generates the inputs, the sanitizer turns silent memory corruption into a visible crash. Fuzzing without ASan misses most of the small overflows and UAFs.

### From dumb to smart
- **dumb fuzzing** → random or blindly mutated bytes; trivial to set up, but rarely gets past input parsing (a random file almost never has a valid header/checksum)
- **mutational fuzzing** → mutate a **corpus** of valid inputs (bit flips, byte insertions, splicing two inputs, ...); needs a good **seed corpus** and doesn't know which mutations matter
- **generational fuzzing** → build inputs from a **format/grammar** specification; reaches deep code, but needs a model of the input format (written by hand)

The trade-off: the smarter the fuzzer, the more setup it needs.

### Coverage-guided fuzzing
**Coverage-guided fuzzing** is the key idea behind modern fuzzers (AFL, libFuzzer, honggfuzz).

The program is **instrumented** to measure code coverage (which basic blocks/edges are executed):
- a mutated input is **kept** if it reaches **new code**
- this creates an **evolutionary loop**: inputs that explore more become new seeds

```
corpus = seeds
loop:
    input  = pick(corpus)
    mutant = mutate(input)
    cov, crashed = run(mutant)        // instrumented binary
    if crashed:       save(mutant)    // bug found
    if cov is new:    corpus.add(mutant)
```

Coverage feedback steers the fuzzer deep into the program **without a format model**, turning blind mutation into a **guided search**.

> [!info] Capiamo
> ![[bangdesk.gif]]
> 
> Prendiamo:
> 
> ```c
> if (buf[0] == 'P')
>  if (buf[1] == 'N')
>    if (buf[2] == 'G')
>      bug();
> ```
> 
> Un fuzzer stupido deve indovinare 3 byte giusti **insieme**: $1/256^3 \approx 1/16$ milioni.
> 
> Un fuzzer coverage-guided invece, appena una mutazione azzecca `'P'`, vede un ramo nuovo e **salva** quell'input; poi muta a partire da lì finché non azzecca `'N'`, e così via. Il problema diventa 3 tentativi da $1/256$, cioè qualche centinaio di esecuzioni.
> 
> Il limite: confronti "tutto o niente" come `if (x == 0xdeadbeef)` (32 bit in un colpo solo) non danno feedback parziale, e il fuzzer resta bloccato. Qui torna utile l'esecuzione simbolica (vedi sotto), e i fuzzer moderni usano trucchi come lo _splitting_ dei confronti o fuzzer ibridi (fuzzing + concolic).

### Fuzzing beyond userspace (background)
**Nyx** → hypervisor-based fuzzing of whole systems (kernels, hypervisors):
- uses **Intel PT** (Processor Trace, hardware tracing of the executed branches) for coverage, so no recompilation is needed
- uses VM **snapshots** for speed: after each test case the whole VM is reset to a clean state in a fraction of a millisecond

### Rehosting embedded firmware
Embedded firmware is hard to fuzz: there's often **no source** and **no OS** to run it on, and the code constantly talks to hardware peripherals through **MMIO** (memory-mapped I/O registers).

**Rehosting** means running the firmware in an **emulator** instead of on the real device:
- **Fuzzware** / **Hoedur** → model the peripheral (MMIO) behavior, so the firmware runs in the emulator and the fuzzer can feed its input through the "peripherals"

This connects to the research area of fuzzing embedded/ICS (Industrial Control Systems) firmware.

---

## Symbolic execution
The goal of **symbolic execution** is to find the **input that reaches a certain program location**.

The program is interpreted with **symbolic inputs** (variables like $\alpha$, $\beta$) instead of concrete values:
- each branch adds a **path constraint** on the symbolic input
- at the target location, the accumulated constraints are given to an **SMT solver** (e.g. Z3), which returns a **concrete input** satisfying them

It explores many paths **systematically**, not by luck (unlike fuzzing).

A classic example is recovering a valid serial number by solving the checks done in `checkSerial()`.

> [!info] Capiamo
> ![[bangdesk.gif]]
> 
> ```c
> void f(int x, int y) {
>    if (x > 10) {
>        if (y == 2 * x + 3)
>            bug();
>    }
> }
> ```
> 
> Diamo come input simboli: $x = \alpha$, $y = \beta$. L'esecuzione simbolica segue **entrambi** i rami di ogni `if`, accumulando vincoli:
> 
> |Path|Constraint|
> |---|---|
> |`x <= 10`|$\alpha \le 10$|
> |`x > 10`, `y != 2x+3`|$\alpha > 10 \land \beta \ne 2\alpha+3$|
> |`x > 10`, `y == 2x+3` → **bug()**|$\alpha > 10 \land \beta = 2\alpha+3$|
> 
> Per raggiungere `bug()` si chiede al solver di risolvere l'ultimo vincolo, e lui risponde ad es. $\alpha = 11, \beta = 25$: ecco l'input che fa scattare il bug.
> 
> Un fuzzer avrebbe dovuto indovinare per caso una coppia con $y$ esattamente uguale a $2x+3$, cosa praticamente impossibile.

### Limits
Symbolic execution is powerful, but it **does not scale**:
- **path explosion** → every branch potentially doubles the number of paths ($n$ branches → up to $2^n$ paths), and loops with symbolic bounds create infinitely many
- **loops and complex constraints** (hashes, crypto, floating point, non-linear arithmetic) blow up the solver
- **modeling the environment** (syscalls, libraries, files, network) is hard: the engine has to know what `read()` or `malloc()` do

#### Concolic execution
**Concolic** (_concrete + symbolic_) execution mixes the two to tame these problems:
1. run the program on a **concrete** input, and record the symbolic constraints along the path actually taken
2. **negate** one of the branch conditions and ask the solver for an input that takes the other side
3. repeat with the new input

When something is too hard to handle symbolically (e.g. a library call), the concrete value is used instead, so execution never gets stuck. Examples: KLEE, SAGE (Microsoft), and hybrid fuzzers like Driller (AFL + angr).

### angr
**angr** is a Python framework for binary analysis and symbolic execution.

Example: find the valid verification number of a binary (find the address that prints "Number validated!", avoid the one of the failure branch):

```python
import angr

project = angr.Project('main.elf')
state   = project.factory.full_init_state()
simgr   = project.factory.simgr(state)

simgr.explore(find=GOOD_ADDR, avoid=BAD_ADDR)   # symbolic exploration

s = simgr.found[0]
print("Valid number: %s" % s.posix.dumps(0)[:12])   # stdin that reaches GOOD_ADDR
```

(the slide also adds constraints so that each of the 12 input bytes is a digit `'0'..'9'`)

```
$ python solve.py
Valid number: 430009016964        # ~26 s
$ ./main.elf
Enter verification number 430009016964
Number validated!
```

---

## Reverse engineering
**Reverse engineering** recovers understanding (and sometimes source) from a binary, in order to:
- find bugs or hidden features when no source is available
- analyze malware, proprietary protocols, firmware

Tools:
- **disassemblers / decompilers** (static) → IDA Pro, Ghidra, Binary Ninja, radare2; get the assembly, reconstruct the control flow (CFG) and recover pseudo-C
- **debuggers** (dynamic) → gdb, and dynamic instrumentation frameworks like **Frida** and **PIN**, which let you hook functions and observe/modify arguments at runtime

> [!warning]
> Decompilation is lossy: variable names, types, structs and comments are gone after compilation, so the pseudo-C must be "cleaned up" by hand (renaming, retyping). That's where most of the reverse engineering time goes.

### Binary diffing
Vendors ship security patches for closed-source products with **little detail** about the bug, but the patch itself **reveals where the bug was**.

**Binary diffing** compares the patched and unpatched binaries:
- functions/blocks are matched by name, structure (CFG shape) or decompiled code
- the **changed blocks** point at the vulnerability (e.g. a newly added bounds check)

This enables **1-day exploits**: reverse the patch, then exploit the systems that are **still unpatched**. Tools: BinDiff, Diaphora

This is what motivates **fast patch deployment**: the window between patch release and exploit is getting shorter.

> [!info] 0-day vs 1-day vs n-day
> 
> - **0-day** → vulnerability unknown to the vendor (no patch exists)
> - **1-day** → vulnerability already patched, but recently (many systems still vulnerable)
> - **n-day** → old known vulnerability, still exploitable on systems that never patched
> 
> Example: after Microsoft's "Patch Tuesday", attackers diff the patches and the day after is jokingly called "Exploit Wednesday".

---
## AI-assisted bug finding
LLMs are now woven across the whole bug-finding pipeline:
- **harness generation** → write fuzzing harnesses automatically (Google's _OSS-Fuzz-Gen_ increased coverage on real projects); writing harnesses is the most time-consuming manual step of fuzzing
- **triage and root-causing** → summarize crashes, point at the faulty line, draft patches
- **variant analysis** → "find other instances of this bug pattern in the codebase"

The 2024–2026 leap: **autonomous agents** that find real, previously-unknown bugs; the agent drives the whole loop itself: **hypothesize → test → confirm**.

These **augment** the classic tools (fuzzers, sanitizers, symbolic execution), they don't replace them.

### Autonomous agents
- **Google Big Sleep** (DeepMind + Project Zero) → LLM agent auditing real code
    - 2024: first known real-world 0-day found by an AI agent (in SQLite)
    - Jul 2025: caught CVE-2025-6965 **before** attackers used it, the first AI pre-emption of an in-the-wild 0-day
    - then 20 open-source bugs (FFmpeg, ImageMagick) and 5 in Apple WebKit (2025)
- **XBOW** (B. Dolan-Gavitt et al.) → autonomous penetration-testing agent
    - first non-human to top HackerOne's US leaderboard, 285 reported bugs (2025)
    - **deterministic validation**: it proves the exploit actually runs instead of trusting the LLM, so almost no false positives
- **DARPA AIxCC** (DEF CON 2025) → competition of find-and-patch systems, won by Team Atlanta
    - 18 real open-source vulnerabilities found and **auto-patched**, about 45 min and 152 USD each
    - all 7 finalists open-sourced their systems

> [!info] Capiamo
> ![[bangdesk.gif]]
> 
> L'idea chiave di XBOW si collega a soundness/completeness: un LLM da solo è come un analizzatore statico rumoroso, "crede" di vedere bug che non esistono (falsi positivi). Aggiungendo un passo di **validazione deterministica** (es. il payload deve davvero far apparire una stringa segreta, o il programma deve davvero crashare) si ottiene la stessa garanzia dell'analisi dinamica: se viene segnalato, è vero.

### Caveats
AI is powerful, but not magic:
- LLMs still **hallucinate** bugs (false positives) and **miss** real ones, hence XBOW's "prove it" validation
- they are best used **with** fuzzers, sanitizers and symbolic execution, not instead of them

It's also a **double-edged sword**:
- the same agents cut the attacker's cost too: faster 1-days (from a patch) and 0-days, cheaply and at scale
- defenders must run these tools **first**, and the value of **memory-safe languages** and **fast patching** rises

> [!warning] For this course
> 
> - LLMs may be used in the exercises (state which one and how)
> - the exam tests the **underlying techniques**, which is what lets you judge when the tool is wrong

---

## The bug-finding toolbox, in a glimpse

|Technique|Runs the code?|Errs towards|Main limit|
|---|---|---|---|
|Static analysis|no|false positives (sound)|noise, needs source|
|Sanitizers (ASan)|yes|false negatives (complete)|only executed paths, ~2x slowdown|
|Coverage-guided fuzzing|yes|false negatives|magic values, deep states|
|Symbolic execution|interpreted|—|path explosion, environment|
|Reverse engineering / diffing|both|—|manual effort|
|AI agents|both|hallucinations|need validation|

Take-home messages:
- **static analysis** reasons without running (sound: no missed bugs, but false positives)
- **dynamic analysis** runs the code (complete: real crashes, but only executed paths); **sanitizers + fuzzing** are the practical workhorses
- **coverage-guided fuzzing** steers mutation into deep code; **rehosting** reaches firmware
- **symbolic execution** solves for inputs, but suffers path explosion (angr, concolic)
- **reverse engineering** and **binary diffing** find bugs in binaries (and enable 1-days)
- **autonomous AI agents** now find (and patch) real 0-days, accelerating both defenders and attackers