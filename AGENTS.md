# AGENTS.md — IPDL / SpeX / Maude

This repository is a formal-verification tool for **cryptographic protocol equivalences**.
It has three layers:

- **Maude** — the term-rewriting engine everything runs on (you must install it; see Setup).
- **SpeX** — a Maude-hosted framework for building specification languages (`IPDL-Spex/`).
- **IPDL** — *Interactive Probabilistic Dependency Logic*, the language you actually write
  protocols and proofs in. A proof reduces a "real" protocol to an "ideal" one via
  equivalence-preserving rewrites; the tool checks the reduction closed.

There are **two implementations** in the repo:

| Folder | Notation | Status | How you run it |
|---|---|---|---|
| `IPDL-Spex/` | new, user-friendly concrete syntax (`.ipdl`) | active, mid-port | SpeX REPL (this doc) |
| `IPDL-AS/`   | old low-level Maude syntax (`.maude`)        | legacy, complete | `maude lib/FILE.maude` |

**This document is about `IPDL-Spex/` (the `.ipdl` language).** Ignore `IPDL-AS/` unless explicitly asked.

---

## 1. Setup (install Maude)

Maude is **not** in Homebrew and the repo ships only a Linux binary (`Maude-linux.zip`).
Install the official build matching the host arch into a local dir:

```bash
cd <repo-root>
# pick the asset for your platform: macos-arm64 | macos-x86_64 | linux-x86_64
ARCH_ASSET=Maude-3.5.1-macos-arm64.zip      # `uname -m` -> arm64 on Apple Silicon
mkdir -p .maude-dist && cd .maude-dist
curl -sL -o maude.zip \
  "https://github.com/maude-lang/Maude/releases/download/Maude3.5.1/$ARCH_ASSET"
unzip -o maude.zip
xattr -d com.apple.quarantine maude 2>/dev/null   # macOS Gatekeeper
chmod +x maude
./maude --version    # -> 3.5.1
```

Maude needs `MAUDE_LIB` pointed at its prelude (the unzipped dir). Always export it:

```bash
export MAUDE_LIB="<repo-root>/.maude-dist"
```

(Asset list for any version: `curl -s https://api.github.com/repos/maude-lang/Maude/releases/latest`.)

---

## 2. Running proofs / the test suite

`.ipdl` proof files live in **`IPDL-Spex/lib/`** (the README says `src/` — that is **stale**, the
files are in `lib/`). You run them through the SpeX REPL, launched from `IPDL-Spex/src/`:

```bash
cd <repo-root>/IPDL-Spex/src
MAUDE_LIB=<repo-root>/.maude-dist \
  printf 'load ../lib/dhke-otp.ipdl\n' \
  | <repo-root>/.maude-dist/maude -no-banner -allow-files run-SpeX
```

`run-SpeX` resolves to `run-SpeX.maude`. Feeding `load <file>` on stdin loads and executes the
whole file (the file ends in its own `check-proof … / quit`).

### Reading the result

Each case-study file ends with `check-proof <Ideal>`. Its verdict is one token
(see `IPDL-Spex/src/Lang/IPDL/Processor.maude:324`):

- **`Done`**  — the proof closed: the protocol you reached equals the normal form of `<Ideal>`. ✅
- **`Ongoing`** — the proof did **not** close; the current protocol ≠ the ideal. ❌ (Not an error — a definite "no".)
- **no `Done`/`Ongoing` printed** — a `quit` earlier in the file ended the interpreter before
  `check-proof` ran (the verdict line is dead code). Check command order.

Useful trailing commands inside a file: `current-protocol` (print the protocol you ended on —
invaluable for debugging an `Ongoing`), `show-bounds` (complexity bounds).

### Benign noise

- `Warning: <file>, line N: Stream error: Bad string.` near EOF is harmless — it's stdin hitting
  EOF after `quit`. Ignore it.
- The REPL prints ANSI color codes; strip with `sed 's/\x1b\[[0-9;]*m//g'`.
- Big GMW proofs run **>10⁹ rewrites / 2+ minutes** — don't assume a hang. Run them backgrounded.

### One-liner to just get the verdict

```bash
cd <repo-root>/IPDL-Spex/src
MAUDE_LIB=<repo-root>/.maude-dist printf 'load ../lib/coin-toss.ipdl\n' \
 | <repo-root>/.maude-dist/maude -no-banner -allow-files run-SpeX \
 | sed 's/\x1b\[[0-9;]*m//g' | grep -aoE '^(Done|Ongoing)$'
```

### Expected state of the suite (main branch, as of this writing)

The repo is mid-port to the new notation, so a **mixed** result is expected:

- **Close (`Done`)**: `csHelloWorld`, `auth-to-secure`, `dhke-otp`, `coin-toss`, `coinToss`,
  `GMW-N`, `1–5gmwn`, `modularGMWN`, `splitGWM-N`, `singleFile`.
- **`Ongoing`**: `cpaSecurity` (uses old `approx assumption` syntax the parser rejects),
  `secure`, `dhke`, `el-gamal` (parse fine but their scripts don't close yet).
- **No verdict**: `gmwN`, `noThen4gmwn` (a `quit` precedes their `check-proof`).

There is **no committed test manifest / CI / expected-output fixture** — `Done`/`Ongoing` from
`check-proof` *is* the ground truth.

---

## 3. The IPDL language

A `.ipdl` file has four sections in order: **declarations → protocol definitions → proof script
→ trailing commands**. Comments start with `` `` `` (two backticks) and run to end of line.

### 3.1 Declarations

```
lang IPDL                                     `` required header, first line

type msg .                                    `` abstract type
function xor : bool * bool -> bool .          `` deterministic function  (product types via *)
distribution flip : unit -> bool .            `` probabilistic sampler (used with `samp`)
parameter q : nat .                           `` universally-quantified bound (for families)
hypothesis 0 < q .                            `` boolean side-condition
```

`bool`, `nat`, `unit` are built in. Function/distribution domains and codomains use `*` for
products: `function enc : msg * key -> ctxt .`

**Channel context** (the observable interface — the adversary boundary; call it `delta`):

```
channel context delta =
 input In  : bool ;
 input X   : bool ;
 output Out : bool
.
```

**Assumptions** state facts the proof may invoke. The forms (left of `|=` is the typing context;
`emptyCCtxt`/`emptyTCtxt`/`nil`/`no assumptions` are the empty contexts):

```
`` two expressions are equal (closed, %automatic = auto-checked; %manual = trusted)
expression-assumption %automatic xorCancel using x y :
  (x : bool) (y : bool) |= xor(( xor((x, y)), y )) = x .

`` two reactions are equal under a channel ctxt ; type ctxt ; inputs
reaction-assumption otp using x y :
   emptyCCtxt ; x : bool ; nil |=
    y : bool <- samp flip (()) ; return xor( (x, y) )
    = samp flip(()) .

`` two protocols are computationally indistinguishable (the crypto hardness step)
approx-assumption ddh using ka kb :
 (chn ddh :: msg) ; no inputs ; no assumptions |=  ddh ::= ...  =  ddh ::= ... .
```

### 3.2 Protocols

A protocol is a parallel composition (`||`) of **channel definitions** `Name ::= <reaction>`,
optionally under `new`/`newfamily` binders that hide internal channels.

```
protocol real =
    new Key : bool in                         `` hidden internal channel
    (
      (Key ::= samp flip (()) )
      ||
      (Out ::= m : bool <- read In ;          `` reaction: bind <- ; sequence with ;
               k : bool <- read Key ;
               return xor((m, k)) )            `` function application: f(( a, b ))
    )
    .
```

**Reaction grammar** (the body of `::=`):

- `return <expr>` — yield a value.
- `samp <dist> ( <expr> )` — sample. `samp flip(())` samples with the unit argument `()`.
- `read <Channel>` / `read <Family>[i]` — read another channel.
- `v : T <- <reaction> ; <rest>` — bind the result of a reaction to `v`, then continue.
- `if <expr> then <reaction> else <reaction>`.
- Expressions: variables, `()`, `True`/`False`, `f((a,b))`, pairs `(a, b)`.

**Families** (indexed channels, for N-party / array protocols):

```
newfamily Msg[bound q] indices: i bounds: bound q : msg in
  ( family Msg[bound q] indices: i bounds: bound q ::= samp unif_msg(()) )
```

**Normal-form constructors** you will SEE in `current-protocol` output (and sometimes write):
`nf(<binds>, <reaction>)`, `preNF(<binds>, <reaction>)` (partially normalized),
`newNF(<typed-channels>, <protocol>)`. `nil`/`empty` are empty binding lists.

### 3.3 Proof script

Begin with `start with <RealProtocol> over <context>`, then chain tactics with `then`
(`then-approx` for the indistinguishability/approximate steps). End with `restructure <Ideal>`.

Tactics I have **verified hands-on** (see §4):

| Tactic | Effect |
|---|---|
| `start with P over delta` | begin; current protocol := `P`, interface := `delta` |
| `fold chn A into chn B` | inline channel `A`'s definition into `B`'s read of it (consumes `A`) |
| `use assumption <name> on chn C` | rewrite `C` with a reaction-assumption |
| `drop read chn A from chn B` | delete `B`'s unused read of `A` — **only if `A ::= samp Dist`** (rule `DROP-nf`) |
| `absorb chn A` | remove a channel (e.g. one left orphaned after `drop read`) |
| `sym from <Ideal> ( <subproof> )` | transform the **ideal** side by `<subproof>` so both sides meet at one normal form |
| `restructure <Ideal>` | renormalize current protocol into the ideal's shape (do this last) |
| `then` / `then-approx` | sequence steps (use `then-approx` around indistinguishability steps) |

Other tactics present in the grammar (use the working `lib/` files as templates;
**verify behavior by running** — the grammar lists them but I did not exercise all):
`subst chn A into chn B`, `add internal channel/family … typed: … assigned: …`,
`gather … from … hiding …` + `bring declarations of hidden channels in front`,
`call-approx <subproof>` + `approx subproof <name> = … .`,
`use approx assumption <name>` (after `then-approx`),
induction: `… rewrite <target> to <cases> by induction on i bound B ( … )`,
`case distinction on <target> ( case: (…) case: (…) ) then merge cases for <target>`,
`compose … in group` / `decompose … with …`, `split … on first index` / `unsplit …`,
`idle` (no-op), `todo` (placeholder).

### 3.4 Trailing commands

```
current-protocol          `` print where you ended (debug Ongoing with this)
check-proof <Ideal>       `` Done / Ongoing
show-bounds               `` complexity bounds
quit                      `` MUST be last — ends the interpreter
```

⚠️ Put `quit` **only at the very end**. Anything after the first `quit` never runs.

---

## 4. Worked examples (all verified `Done`)

Runnable copies live in `IPDL-Spex/scratch/`. Run any with the §2 one-liner against
`../scratch/<file>`.

### 4.1 Minimal one-time pad — `scratch/toy1-otp.ipdl`

Real outputs `In ⊕ Key` (fresh key); ideal outputs a fresh coin. The OTP assumption says a fixed
value masked by a fresh uniform bit *is* a fresh uniform bit.

```
start with real over delta then
fold chn Key into chn Out then          `` Out := m<-In; k<-samp flip; return xor(m,k)
use assumption otp on chn Out then      `` otp fires: x=m (a single var!), y=k -> Out := m<-In; samp flip
restructure ideal
```

No `sym from` needed because the ideal has no hidden channels.

### 4.2 Discard unused randomness — `scratch/toy2-junkcoin.ipdl`

Real samples a `Junk` coin it reads but never uses. The proof must remove it.

```
start with real over delta then
drop read chn Junk from chn Out then    `` Junk ::= samp flip, j unused -> droppable (DROP-nf)
absorb chn Junk then                    `` removes the now-orphaned sample channel
fold chn Key into chn Out then
use assumption otp on chn Out then
restructure ideal
```

### 4.3 Ideal with a hidden simulator — `scratch/toy3-symfrom.ipdl`

The ideal routes `Out` through a hidden channel `H`. `sym from` rewrites the **ideal** side to meet
the reduced real side.

```
start with real over delta then
fold chn Key into chn Out then
use assumption otp on chn Out then       `` real side reduced to: Out := m<-In; samp flip
sym from ideal (
   fold chn H into chn Out               `` fold the ideal's hidden H away -> same normal form
) then
restructure ideal
```

---

## 5. Hard-won gotchas (these will bite you)

1. **Assumptions only match a SINGLE bound variable, not a compound expression.**
   `reaction-assumption otp` with `return xor((x,y))` fires on `return xor((m, k))` (m is a
   variable) but **not** on `return xor(( xor((m,k1)), k2 ))` — the masked value `xor((m,k1))` is
   compound. Route the inner result through its own channel so it becomes a variable.

2. **The normal form does NOT garbage-collect dead samples.** After an assumption collapses a
   reaction, a leftover `c <- samp flip(())` whose result is unused **stays** and blocks `Done`.
   Avoid creating it.

3. **`drop read A from B` only works when `A ::= samp Dist`** (a pure independent sample) and B's
   read is unused (rule `DROP-nf`, `syntax.maude:5249`). You **cannot** drop B's read of a
   *computed* channel this way. Consequence: a doubly-masked output `(In⊕K1)⊕K2` does **not**
   reduce to a clean single-coin ideal, because the dead dependency bottoms out at a computed
   channel, not a sample. (I confirmed this by trying — it's a real limit, not a skill issue.)

4. **`drop read` leaves the channel orphaned** — follow it with `absorb chn A` to delete the
   now-unread channel, or `Done` won't fire.

5. **Fold order matters.** `fold` inlines a channel into its reader in place. To peel a mask with
   an assumption, the masking sample must end up as the **last** binding before the `return`.

6. **`quit` before `check-proof` = no verdict.** Some lib files have multiple sections; only the
   first (up to the first `quit`) runs on a plain `load`.

7. **Debug `Ongoing` with `current-protocol`.** Compare the printed normal form against the ideal;
   the diff tells you exactly which dead read / extra sample / unfolded channel remains.

---

## 6. Recipe for writing a new proof

1. Declare `type`/`function`/`distribution`, then the `channel context delta` (the I/O boundary).
2. Declare the `*-assumption`s for each cryptographic/algebraic step. Keep masked values as single
   variables (gotcha #1).
3. Write `protocol real = …` and `protocol ideal = …`. Put simulator/internal state under `new`.
4. Write the script: `start with real over delta then` … finish with `restructure ideal`.
   - Inline internal channels with `fold`; rewrite with `use assumption … on …`.
   - If the ideal has hidden channels, massage them with `sym from ideal ( … )`.
   - Remove unused fresh coins with `drop read` + `absorb`.
   - Use `then-approx` + `use approx assumption` for indistinguishability steps.
5. End with `current-protocol`, `check-proof ideal`, `quit`.
6. Run it (§2). If `Ongoing`, read `current-protocol` and fix the residual difference.

---

## 7. Key source-file map (for deeper questions)

| File | What's in it |
|---|---|
| `IPDL-Spex/src/Lang/IPDL/Language.maude` | concrete-syntax grammar + parser (`read-cmd`, `read-proof`) |
| `IPDL-Spex/src/Lang/IPDL/Data.maude` | abstract syntax (command/proof constructors) |
| `IPDL-Spex/src/Lang/IPDL/syntax.maude` | the rewrite rules (e.g. `DROP-nf` at line 5249) |
| `IPDL-Spex/src/Lang/IPDL/strategies.maude` | tactic → strategy mappings |
| `IPDL-Spex/src/Lang/IPDL/Processor.maude` | REPL command handlers (`check-proof` at line 324) |
| `IPDL-Spex/lib/*.ipdl` | the case studies — best source of real idioms |
| `IPDL-AS/doc/POPL2023.pdf`, `case_studies.pdf` | the theory and intended proofs |
