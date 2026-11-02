---
Class: "[[System Security]]"
Sem: WiSe 26/27
---
---
# SysSec – x86-64 & Tools Cheatsheet

> [!info] Contesto Lecture 2 (x86-64 Assembly I), Tutorial 2, Assignment 1 (`intro`, `environment` su REF). Le note sono ordinate per uso pratico: tool → registri → calling convention → stack/memoria → encoding → workflow REF.

---

## 1. objdump – disassemblare e guardare byte

```bash
objdump -d -M intel prog                          # disassembla le sezioni di codice (Intel syntax)
objdump -d -M intel --disassemble=func prog       # solo una funzione
objdump -d -M intel -j .text prog                 # solo una sezione
objdump -D -M intel -j .rodata prog               # decodifica DATI come se fossero codice
objdump -s -j .rodata prog                        # hexdump di una sezione (hex + ASCII)
objdump -t prog                                   # tabella dei simboli
objdump -d --no-show-raw-insn -M intel prog       # solo istruzioni, senza byte
objdump -d -M intel --start-address=0x1129 --stop-address=0x113f prog
```

> [!tip] Estrarre tutti i byte di una funzione come stringa hex
> 
> ```bash
> objdump -d --disassemble=secret intro \
>   | grep -P '^\s+[0-9a-f]+:' | cut -f2 | tr -d ' \n'; echo
> ```
> 
> `cut -f2` prende la colonna dei byte (colonne separate da tab). **Controlla sempre** a mano che l'ultima istruzione inclusa sia il `ret` e che non ci siano byte di padding (`nop`, `66 0f 1f ...`, `cc`) dopo.

**Ogni riga di output ha 3 parti:** `indirizzo: byte macchina mnemonic + operandi`.

> [!warning] Il disassembler non sa cosa è codice Dopo il `ret` objdump continua a decodificare qualsiasi byte, anche stringhe/dati → istruzioni senza senso (`gs`, `insb`, `outsd`…). Se vedi roba strana: converti i byte in ASCII (vedi [[#8. Tabella ASCII rapida]]).

---

## 2. readelf – struttura del file ELF

```bash
readelf -h prog          # ELF header: Class (ELF64/ELF32), Type (EXEC/DYN=PIE), Entry point
readelf -S -W prog       # sezioni: nome, indirizzo, offset, size, flag
readelf -s -W prog       # simboli: indirizzo, size, tipo (FUNC/OBJECT), nome
readelf -l -W prog       # segmenti (program headers): cosa viene caricato dove e con che permessi
readelf -p .rodata prog  # stringhe in una sezione
readelf -x .text prog    # hexdump di una sezione
```

### Sezioni da conoscere

|Sezione|Contenuto|Flag tipici|
|---|---|---|
|`.text`|codice delle funzioni|`AX` (alloc, exec)|
|`.plt` / `.plt.sec`|stub per chiamare funzioni di libc|`AX`|
|`.init` / `.fini`|codice di inizio/fine|`AX`|
|`.rodata`|costanti, stringhe letterali (`"area = %d\n"`)|`A`|
|`.data`|globali inizializzate|`WA`|
|`.bss`|globali a zero (non occupa spazio nel file)|`WA`|
|`.got` / `.got.plt`|indirizzi risolti a runtime|`WA`|
|`.symtab` / `.strtab`|simboli e nomi (sparisce con `strip`)|–|

> [!tip] Leggere una riga di `readelf -S -W` `[Nr] Name Type Address Off Size ES Flg Lk Inf Al` **Address** = indirizzo virtuale (relativo al load base se PIE), **Off** = offset nel file, **Size** = dimensione in byte (hex). readelf stampa gli indirizzi con zeri iniziali (`0000000000001060`): se un formato chiede "senza leading zeros", toglili. Per capire in che sezione sta una funzione: prendi il suo indirizzo da `readelf -s` e verifica in quale intervallo `[Address, Address+Size)` cade.

Altri tool utili: `file prog`, `nm prog`, `strings prog`, `checksec --file=prog` (se installato), `xxd prog | less`.

---

## 3. GDB

### Avvio e breakpoint

```gdb
gdb -q ./prog                    # -q = niente banner
gdb -q -x cmds.gdb ./prog        # esegue i comandi del file (uno per riga)
gdb -q --args ./prog 6 7         # passa direttamente gli argomenti

set disassembly-flavor intel
break *secret                    # bp sulla PRIMA istruzione (prima del prologo)
break secret                     # bp DOPO il prologo (differenza importante!)
break *0x555555555159            # bp su indirizzo
tbreak main                      # breakpoint temporaneo
info breakpoints / delete N / disable N
run 5                            # avvia con argomenti
run < input.txt                  # stdin da file
starti                           # ferma alla primissima istruzione (_start / loader)
```

### Eseguire

|Comando|Short|Cosa fa|
|---|---|---|
|`stepi`|`si`|1 istruzione, **entra** nelle `call`|
|`nexti`|`ni`|1 istruzione, **esegue tutta** la `call`|
|`step` / `next`|`s` / `n`|come sopra ma per riga di sorgente (serve debug info)|
|`finish`|`fin`|fino al return della funzione corrente (mostra il valore di ritorno)|
|`continue`|`c`|fino al prossimo breakpoint|
|`until *addr`||fino a un indirizzo|
|`kill` / `quit`|`k` / `q`||

### Ispezionare

```gdb
disassemble secret               # disassembla una funzione (=> indica dove sei)
x/10i $rip                       # prossime 10 istruzioni
x/i $pc                          # istruzione che verrà eseguita ORA
display/i $pc                    # la mostra automaticamente a ogni step
info registers                   # tutti i registri   (i r)
info registers rdi rsi eflags
p/d $eax   p/x $rax   p/t $al    # decimale / hex / binario
p $rsp                           
x/2dw $rbp-8                     # 2 word (4B) a rbp-8, in decimale
x/4gx $rsp                       # 4 giant (8B) a rsp, in hex
x/2wx $esp                       # (32-bit) 2 word in cima allo stack
x/s 0x402004                     # stringa
info frame                       # info sul frame corrente
backtrace                        # (bt) catena di chiamate
info proc mappings               # memory map (vmmap in GEF)
show environment                 # variabili d'ambiente passate al programma
set $eax = 42                    # modificare un registro
```

### Formato di `x/NFU addr`

- **N** = quanti elementi
- **F** = formato: `x` hex, `d` decimale, `u` unsigned, `t` binario, `c` char, `s` stringa, `i` istruzione
- **U** = unità: `b` 1B, `h` 2B (half), `w` 4B (word), `g` 8B (giant)

> [!warning] "word" in GDB ≠ "word" in Intel In GDB `w` = **4 byte**. In terminologia Intel: word = 2B, dword = 4B, qword = 8B.

### File di comandi (risparmia tempo)

```gdb
# cmds.gdb
set disassembly-flavor intel
break *secret
run 5
info registers edi
```

`gdb -q -x cmds.gdb ./intro`. Un file `~/.gdbinit` viene caricato automaticamente a ogni avvio (e un `.gdbinit` nella directory corrente se l'auto-load lo permette).

> [!note] GEF Se si carica: `context`, `vmmap`, `telescope $rsp`, `checksec`. Se all'avvio vedi un'eccezione Python, GEF non è caricato: i comandi normali funzionano, usa `p/d` per i decimali.

---

## 4. Registri x86-64

### Registri general purpose e sottoregistri (tutti e 16)

Nomi delle dimensioni (terminologia Intel): **qword** = 64 bit, **dword** = 32 bit, **word** = 16 bit, **byte** = 8 bit.

|64 bit (qword)|32 bit (dword)|16 bit (word)|8 bit low (byte)|8 bit high (bit 8–15)|Ruolo (System V)|Salvato da|
|---|---|---|---|---|---|---|
|`rax`|`eax`|`ax`|`al`|`ah`|return value, nr. syscall|caller|
|`rbx`|`ebx`|`bx`|`bl`|`bh`|generico|**callee**|
|`rcx`|`ecx`|`cx`|`cl`|`ch`|arg 4, contatore (`rep`, shift per `cl`)|caller|
|`rdx`|`edx`|`dx`|`dl`|`dh`|arg 3, parte alta di `mul`/`div`|caller|
|`rsi`|`esi`|`si`|`sil`|–|arg 2, source (string ops)|caller|
|`rdi`|`edi`|`di`|`dil`|–|arg 1, destination (string ops)|caller|
|`rbp`|`ebp`|`bp`|`bpl`|–|base/frame pointer|**callee**|
|`rsp`|`esp`|`sp`|`spl`|–|stack pointer|**callee** (deve tornare uguale)|
|`r8`|`r8d`|`r8w`|`r8b`|–|arg 5|caller|
|`r9`|`r9d`|`r9w`|`r9b`|–|arg 6|caller|
|`r10`|`r10d`|`r10w`|`r10b`|–|temp (arg 4 nelle syscall)|caller|
|`r11`|`r11d`|`r11w`|`r11b`|–|temp (distrutto da `syscall`)|caller|
|`r12`|`r12d`|`r12w`|`r12b`|–|generico|**callee**|
|`r13`|`r13d`|`r13w`|`r13b`|–|generico|**callee**|
|`r14`|`r14d`|`r14w`|`r14b`|–|generico|**callee**|
|`r15`|`r15d`|`r15w`|`r15b`|–|generico|**callee**|

> [!note] Nomi alternativi
> 
> - In alcuni testi `r8b` si scrive `r8l` (stessa cosa).
> - Le sigle storiche: **a**ccumulator, **b**ase, **c**ounter, **d**ata, **s**ource **i**ndex, **d**estination **i**ndex, **b**ase **p**ointer, **s**tack **p**ointer. Per i registri "vecchi" il 32 bit ha la `e` (extended), il 64 la `r`; per `r8–r15` si usano i suffissi `d`/`w`/`b`.
> - Solo `rax rbx rcx rdx` hanno anche il byte alto (`ah bh ch dh`): `rsi rdi rbp rsp` e `r8–r15` no.

### Quale sottoregistro prende un `mov`

|Istruzione|Byte scritti|Il resto del registro a 64 bit|
|---|---|---|
|`mov rdi, …`|8|–|
|`mov edi, …`|4|**azzerato** (bit 32–63 = 0)|
|`mov di, …`|2|invariato|
|`mov dil, …`|1 (bit 0–7)|invariato|
|`mov ah, …`|1 (bit 8–15)|invariato|

Esempio (come negli appunti): `mov edi, 0x401180` → `rdi = 0x0000000000401180`. Il compilatore usa `edi` invece di `rdi` quando il valore entra in 32 bit: encoding più corto, stesso risultato.

Speciali: `rip` (instruction pointer, in GDB anche `$pc`), `rflags`, segmenti (`cs ds es ss fs gs` – `fs` usato per TLS/stack canary), `xmm0`–`xmm15` (float/SIMD).

```
 63                              31              15       7       0
 ┌───────────────────────────────┬───────────────┬─────────┬───────┐
 │             rax               │      eax      │   ax    │       │
 │                               │               │ ah │ al │       │
 └───────────────────────────────┴───────────────┴─────────┴───────┘
```

> [!important] Regole di scrittura sui sottoregistri
> 
> - Scrivere un registro **a 32 bit** (`mov ecx, …`) **azzera i 32 bit alti** del registro a 64 bit (zero-extension).
> - Scrivere a **8 o 16 bit** (`al`, `ah`, `ax`) **lascia invariato** tutto il resto.
> - `ah/bh/ch/dh` (byte alto dei 16 bit) esistono solo per a/b/c/d. `sil/dil/bpl/spl` e `r8b…` richiedono un prefisso REX, e con REX non si possono usare `ah/bh/ch/dh` nella stessa istruzione.
> 
> Esempio (valore diverso da quello del tutorial): `rax = 0x1111222233334444`
> 
> - `mov al, 0xff` → `0x11112222333344ff`
> - `mov ah, 0x00` → `0x11112222333300ff`
> - `mov ax, 0xbeef` → `0x111122223333beef`
> - `mov eax, 0x1` → `0x0000000000000001` ← metà alta azzerata!

> [!tip] Perché `mov eax, 1` invece di `mov rax, 1` Risultato identico grazie alla zero-extension, ma l'encoding è più corto (5 byte vs 7: niente prefisso REX.W, immediato da 32 bit). Stessa idea per `xor eax, eax` per azzerare.

### RFLAGS (i flag che servono davvero)

|Flag|Significato|Settato da|
|---|---|---|
|`ZF`|risultato = 0|`cmp`, `test`, aritmetica|
|`SF`|risultato negativo||
|`CF`|carry / borrow unsigned||
|`OF`|overflow signed||

Salti: `je/jz`, `jne/jnz`; **signed**: `jl jle jg jge`; **unsigned**: `jb jbe ja jae`.

### 32 bit (IA-32)

Stessi nomi senza `r` e senza `r8–r15`: `eax ebx ecx edx esi edi ebp esp eip eflags`.

---

## 5. Dove finiscono argomenti e valori di ritorno

### x86-64 Linux – System V AMD64 ABI

| |Registro|
|---|---|
|Arg 1|`rdi` (int → `edi`)|
|Arg 2|`rsi` (`esi`)|
|Arg 3|`rdx` (`edx`)|
|Arg 4|`rcx` (`ecx`)|
|Arg 5|`r8`|
|Arg 6|`r9`|
|Arg 7+|sullo stack (da destra a sinistra)|
|Float/double|`xmm0`–`xmm7`|
|**Return**|`rax` (`eax` per int), 128 bit in `rdx:rax`, float in `xmm0`|

> Mnemonico: **D**iane's **S**ilk **D**ress **C**osts **8**-**9** $ → `rdi rsi rdx rcx r8 r9`

- **Caller-saved** (la funzione chiamata può distruggerli): `rax rcx rdx rsi rdi r8–r11`
- **Callee-saved** (devono essere preservati): `rbx rbp r12–r15` (e `rsp`)
- Stack allineato a 16 byte prima di una `call`; _red zone_: 128 byte sotto `rsp` usabili da funzioni foglia senza spostare `rsp` (per questo `area` scrive a `[rbp-0x4]` senza `sub rsp`).
- Funzioni variadiche (`printf`): `al` = numero di registri vettoriali usati.

**Syscall x86-64** (`syscall`, bytes `0f 05`): numero in `rax`, args in `rdi rsi rdx r10 r8 r9`, ritorno in `rax`; `rcx` e `r11` vengono distrutti.

### x86 32 bit – cdecl

- Argomenti **sullo stack**, pushati da destra a sinistra (arg1 è il più vicino alla cima).
- `call` pusha l'indirizzo di ritorno sopra gli argomenti.
- Return in `eax` (64 bit in `edx:eax`), float in `st(0)`.
- Caller pulisce lo stack (`add esp, N` dopo la call).
- Caller-saved: `eax ecx edx`; callee-saved: `ebx esi edi ebp`.

```
All'ingresso della funzione         Dopo push ebp; mov ebp, esp
(bp su *func)

  [esp+8]  arg2                       [ebp+12] arg2
  [esp+4]  arg1                       [ebp+8]  arg1
  [esp]    return address             [ebp+4]  return address
                                      [ebp]    vecchio ebp   ← esp = ebp
```

> Lo spostamento di offset tra `esp` all'ingresso e `ebp` nel corpo è dovuto proprio al `push ebp`: +4 byte.

**Syscall 32 bit** (`int 0x80`): numero in `eax`, args in `ebx ecx edx esi edi ebp`.

> [!tip] Come verificarlo in GDB
> 
> - x86-64: `break *f` → `info registers rdi` = primo argomento. `finish` → il valore di ritorno è in `rax`.
> - 32-bit: `break *f` → `x/2wx $esp`: il primo è l'indirizzo di ritorno (`x/i <addr>` → cade dentro `main`, subito dopo la `call`), il secondo il primo argomento.

---

## 6. Stack frame e prologo/epilogo

```asm
push rbp            ; 55        salva il frame pointer del chiamante
mov  rbp, rsp       ; 48 89 e5  nuovo frame
sub  rsp, N         ; (opz.)    spazio per le locali
...                 ; locali a [rbp-x], (32-bit) args a [ebp+8], [ebp+12]...
leave               ; c9        = mov rsp, rbp ; pop rbp
ret                 ; c3        pop rip
```

Con `-O0` gcc copia gli argomenti dai registri nello stack (`mov [rbp-0x4], edi`) e li rilegge → codice verboso ma facile da seguire.

- Lo stack cresce **verso indirizzi bassi**: `push` = `rsp -= 8; [rsp] = val`.
- `call f` = `push rip_next; jmp f`.

---

## 7. Layout della memoria di un processo

```
indirizzi alti
┌──────────────────────────────┐
│ stringhe env, argv, path     │  ← cima dello stack all'avvio
│ auxv, envp[], argv[], argc   │
│ stack ↓                      │
│ ...                          │
│ librerie condivise (mmap)    │
│ ...                          │
│ heap ↑                       │
│ .bss / .data                 │
│ .rodata                      │
│ .text                        │
└──────────────────────────────┘
indirizzi bassi
```

- **PIE**: gli indirizzi in objdump/readelf sono offset dal load base. In GDB (ASLR disattivato di default, `set disable-randomization on`) il base 64-bit è `0x555555554000` → `0x1129` diventa `0x555555555129`. Per binari 32-bit PIE il base tipico è `0x56555000`.
- Sotto GDB lo stack è deterministico; fuori, con ASLR attivo, cambia a ogni esecuzione (`cat /proc/sys/kernel/randomize_va_space`; `setarch -R ./prog` lo disattiva per un'esecuzione).

> [!note] Perché gli indirizzi sullo stack cambiano tra GDB e shell (Assignment 1, Task 3) Le variabili d'ambiente, `argv` e il path del programma sono copiati **in cima allo stack**. Se cambia la loro lunghezza totale, tutto ciò che sta sotto (frame, array locali) si sposta. Cose da confrontare:
> 
> - `env` nella shell vs `show environment` in GDB (GDB tende ad aggiungerne alcune, es. `LINES`, `COLUMNS`)
> - il valore di `argv[0]` / la variabile `_`: GDB avvia il programma con il **path assoluto**
> - comandi utili: `show environment`, `unset environment VAR`, `set environment VAR=val`, `x/s *((char**)environ)` e `x/20s` vicino alla cima dello stack

---

## 8. Encoding delle istruzioni

- **x86 = CISC, lunghezza variabile**: da 1 a 15 byte (`push rbp` = 1 byte, `imul` con memoria = 4 byte, `mov` con imm64 = 10 byte).
- **ARM64**: sempre 4 byte. **RISC-V**: 4 byte (2 con l'estensione compressa C).
- Conseguenza: su x86 devi sapere **dove inizia** un'istruzione per decodificare il resto; partendo da un offset sbagliato ottieni istruzioni diverse.

### Prefisso REX (0x40–0x4F) in 64-bit mode

`0100 W R X B`

- `W=1` → operandi a 64 bit. `48` = REX.W: `89 e5` = `mov ebp, esp`, `48 89 e5` = `mov rbp, rsp`.
- `R/X/B` → estendono i campi registro per accedere a `r8–r15`.

### Opcode da riconoscere a occhio

|Byte|Istruzione|
|---|---|
|`55` / `5d`|`push rbp` / `pop rbp`|
|`50+r` / `58+r`|push / pop registro|
|`c3`|`ret`|
|`c9`|`leave`|
|`e8 xx xx xx xx`|`call rel32`|
|`e9` / `eb`|`jmp rel32` / `jmp rel8`|
|`74` / `75`|`je` / `jne` (rel8)|
|`90`|`nop`|
|`cc`|`int3` (breakpoint)|
|`0f 05`|`syscall`|
|`cd 80`|`int 0x80`|
|`b8+r imm32`|`mov r32, imm32`|
|`31 c0`|`xor eax, eax`|
|`f3 0f 1e fa`|`endbr64` (CET; sparisce con `-fcf-protection=none`)|

### Little endian

Il byte meno significativo sta all'indirizzo più basso: `0x12345678` in memoria = `78 56 34 12`. Gli immediati nelle istruzioni sono anche little endian (`b8 01 00 00 00` = `mov eax, 1`).

---

## 9. Tabella ASCII rapida

|Hex|Char|Hex|Char|Hex|Char|
|---|---|---|---|---|---|
|`0a`|`\n`|`30–39`|`0–9`|`41–5a`|`A–Z`|
|`20`|spazio|`21`|`!`|`61–7a`|`a–z`|
|`00`|NUL (fine stringa)|`2e`|`.`|`5f`|`_`|

Lettera maiuscola ↔ minuscola: differenza `0x20`. Conversione rapida in shell:

```bash
echo 48656c6c6f | xxd -r -p         # hex → testo
printf 'Hi' | xxd                   # testo → hex
python3 -c "print(bytes.fromhex('48656c6c6f'))"
```

---

## 10. AT&T vs Intel (se vedi output senza `-M intel`)

| |Intel|AT&T|
|---|---|---|
|Ordine|`mov dst, src`|`mov src, dst`|
|Registri|`rax`|`%rax`|
|Immediati|`1`|`$1`|
|Memoria|`[rbp-0x4]`|`-0x4(%rbp)`|
|Dimensione|`dword ptr`|suffisso `movl`, `movq`|

---

## 11. Workflow REF / Assignment

```bash
task check          # verifica i file di consegna
task submit         # consegna
echo -n 'flag' > /home/user/flag_a     # -n: niente newline/spazi extra
cat -A /home/user/flag_a               # controlla caratteri invisibili
```

- Formato flag **esatto**: niente spazi, virgolette, `0x` dove non richiesto, hex in minuscolo.
- Per i flag di GDB, prepara un `cmds.gdb` e rilancialo invece di ridigitare tutto.
- Per ricostruire una funzione (`secret.c`): usala come oracolo con vari input (`run 6`, `run 10`, `run 20`, … + `finish`), confronta con la tua implementazione, e prova anche **valori negativi e 0**: il test la compila sia a 64 che a 32 bit e confronta molti input.
- Compilare per prova: `gcc -O0 -o t t.c` e `gcc -m32 -O0 -o t32 t.c`.
- Il binario in GDB e quello in objdump sono lo stesso: se un indirizzo non torna, ricorda l'offset del base PIE.

> [!question] Domande di ripasso (Tutorial 2)
> 
> - Cosa sono le tre colonne di un output objdump?
> - Quanto è lunga una funzione? → indirizzo dopo il `ret` − indirizzo di inizio.
> - Cosa cambia il byte `48` davanti a un'istruzione?
> - Dopo `mov ecx, 0xffffffff`, cosa c'è nei 32 bit alti di `rcx`?
> - `bpl` a che registro appartiene? `rsi` ha un byte alto separato?

---

## 12. Appunti dal Tutorial 2 (utili per i prossimi assignment)

### Leggere bene una riga di objdump

```
112a:   48 89 e5        mov rbp, rsp
 │        │               └─ istruzione leggibile (mnemonic + operandi)
 │        └─ byte macchina: opcode (89) + prefisso (48) + ModRM (e5)…
 └─ indirizzo/offset dell'istruzione (relativo al load base se PIE)
```

> [!warning] Errore facile La prima colonna è l'**indirizzo** dell'istruzione, non c'entra con `rbp`. E la colonna dei byte non è solo "opcode": contiene anche prefissi, byte ModRM, displacement (`fc` = −4) e immediati (`01`).

- Lunghezza di una funzione = indirizzo dopo il `ret` − indirizzo di inizio (oppure la colonna **Size** di `readelf -s`).
- Displacement negativi in complemento a 2: `fc` = −4, `f8` = −8 → `[rbp-0x4]`, `[rbp-0x8]`.

### Dati decodificati come codice
Dopo il `ret` i byte `48 65 6c 6c 6f 21 0a` diventano istruzioni senza senso (`gs`, `insb`, `outsd`), ma in realtà sono una stringa ASCII. Segnali che stai guardando dati e non codice:

- istruzioni rare/privilegiate (`in`, `out`, `ins`, `outs`, `hlt`), prefissi di segmento strani (`gs`, `fs`, `es:`)
- sequenze di byte tutti in `0x20–0x7e` (stampabili) che finiscono con `00` o `0a`
- `objdump -s` mostra la colonna ASCII a destra: usala per riconoscere stringhe.

### `mov eax, imm` vs `mov rax, imm`
Due motivi, entrambi da saper dire:
1. **Perché è corretto**: scrivere un registro a 32 bit azzera i 32 bit alti, quindi `rax` diventa esattamente 1.
2. **Perché è preferito**: encoding più corto (niente prefisso REX.W, immediato da 4 byte) → codice più compatto, cache più efficiente.

### ELF header (`readelf -h`)
- **Class**: `ELF64` / `ELF32` → 64 o 32 bit.
- **Type**: `DYN` = PIE (indirizzi relativi, caricato a un base casuale; in GDB `0x555555554000`), `EXEC` = non-PIE (indirizzi assoluti fissi, tipicamente da `0x400000`, es. `0x401180`).
- **Entry point**: punta a `_start` (inizio di `.text`), **non** a `main`. `_start` chiama `__libc_start_main`, che a sua volta chiama `main`. Per fermarsi davvero all'inizio: `starti`.

### `.rodata` e stringhe
- Le stringhe letterali (`"usage: %s WIDTH HEIGHT\n"`, `"area = %d\n"`) stanno in `.rodata` (read-only), terminate da `00`.
- Nel codice le trovi tramite **indirizzamento RIP-relative**: `lea rdi, [rip+0xe8c]`. objdump aggiunge un commento con l'indirizzo calcolato (`# 2004`), che poi cerchi con `objdump -s -j .rodata`. Così colleghi una funzione alle stringhe che usa, molto utile quando non hai il sorgente.

### Sezioni (`readelf -S -W`)
- **Flg**: `A` = allocata in memoria, `X` = eseguibile, `W` = scrivibile, `M`/`S` = merge/strings.
- `.text` (`AX`) contiene il codice di **tutte** le funzioni, incluse `_start`, `main` e quelle del programma.
- `.bss` è `NOBITS`: occupa spazio in memoria ma **non nel file** (stesso offset della sezione successiva).
- Le sezioni senza `A` (`.symtab`, `.strtab`, `.comment`) non vengono caricate in memoria.

### Simboli (`readelf -s -W`)

|Tabella|Contenuto|Note|
|---|---|---|
|`.dynsym`|simboli importati/esportati da librerie (`printf@GLIBC_2.2.5`, `atoi`, `stderr`)|necessaria a runtime, resta anche dopo `strip`|
|`.symtab`|tutti i simboli del programma (funzioni locali, file sorgente, variabili)|sparisce con `strip` → binari "stripped" senza nomi di funzione|

- **Value** = indirizzo, **Size** = dimensione in byte, **Type** = `FUNC` / `OBJECT` / `FILE`.
- **Ndx**: `UND` = non definito qui (viene da una libreria), `ABS` = assoluto, un numero = **indice della sezione** in `readelf -S` (es. `15` → `[15] .text`, `25` → `[25] .bss`). Così trovi subito in che sezione sta un simbolo.
- Builds diversi danno indirizzi diversi (nel Task 1 `area` sta a `0x1129`, nella demo a `0x1159`): verifica sempre sul binario che hai davanti.

### W^X (Write XOR Execute)
Nessuna zona di memoria è contemporaneamente scrivibile ed eseguibile: un attaccante non può scrivere codice e poi eseguirlo nello stesso posto.

- Si vede nei **segmenti** (`readelf -l -W`, colonna `Flg`: `R E` vs `RW`) e in GDB con `info proc mappings`. I permessi reali li danno i segmenti, non le sezioni.
- Stack e heap sono `RW` (bit **NX**): shellcode iniettato sullo stack non parte → motivo per cui esistono tecniche come il riuso di codice (ROP), che vedrai più avanti.
- Ci sono comunque zone **scrivibili che contengono puntatori a codice** (`.got`, `.init_array`, `.fini_array`, return address sullo stack): sovrascriverle permette di dirottare il flusso senza scrivere codice eseguibile. Per questo esistono difese come **RELRO** (rende la GOT read-only dopo l'avvio) e gli **stack canary**.