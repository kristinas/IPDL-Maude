# Session Summary — IPDL/SpeX/Maude setup, test run, and authoring AGENTS.md

Date: 2026-06-18
Repo: `IPDL-Maude` (github.com/kristinas/IPDL-Maude), branch `main`

## What was asked, in order

1. Set up the repo and run the tests; prove the tests were actually run.
2. Check whether any releases/tags should be used instead of `main`.
3. Re-run the original instruction scoped specifically to the `IPDL-Spex/` subfolder.
4. Clarifying Q&A: what does `Ongoing` mean? are these the expected results? how can a file
   "parse-fail" yet return `Ongoing`?
5. Write a thorough `AGENTS.md` documenting how to use the tool, plus an IPDL language manual
   grounded in original toy proofs the agent wrote and made pass.
6. Provide exact commands to run the three toy test cases.
7. Resolve a naming confusion (two `toy2` files).
8. Produce this summary + append the raw session transcript.

---

## 1. Setup & how the tool runs

The repo is a formal-verification tool for cryptographic protocol equivalences, layered as:

- **Maude** — the term-rewriting engine (must be installed).
- **SpeX** — a Maude-hosted framework for building specification languages (`IPDL-Spex/`).
- **IPDL** — *Interactive Probabilistic Dependency Logic*, the language protocols/proofs are
  written in. A proof reduces a "real" protocol to an "ideal" one via equivalence-preserving
  rewrites; `check-proof` reports whether the reduction closed.

There are two implementations: `IPDL-Spex/` (new `.ipdl` concrete syntax, active/mid-port) and
`IPDL-AS/` (old `.maude` low-level syntax, legacy/complete).

### Setup performed

- The repo ships only a Linux Maude binary and Maude is not in Homebrew. Downloaded the official
  **Maude 3.5.1 macOS arm64** build into `.maude-dist/` (verified `Mach-O arm64`, `--version` =
  `3.5.1`).
- Set `MAUDE_LIB=<repo>/.maude-dist` so Maude finds its prelude.
- Launched the documented SpeX REPL: `maude -no-banner -allow-files run-SpeX` from `IPDL-Spex/src/`,
  then `load ../lib/<file>.ipdl`.
- Discovered the README's testing instructions are **stale**: it says `.ipdl` files are in
  `IPDL-Spex/src/`, but they actually live in `IPDL-Spex/lib/`.

### Verdict semantics

Each case-study file ends in `check-proof <Ideal>` (`IPDL-Spex/src/Lang/IPDL/Processor.maude:324`):

- **`Done`** — proof closed (reached protocol == normal form of the ideal).
- **`Ongoing`** — proof did **not** close (a definite "no", not an error, not "still running").
- **no token** — a `quit` earlier in the file ended the interpreter before `check-proof` ran.

---

## 2. Test results (IPDL-Spex case studies)

Ran all 20 `check-proof` case studies. Engine work was real (e.g. `GMW-N`: **1,119,609,061
rewrites / ~134s**; `dhke`: 2.8M rewrites; `csHelloWorld`: 135k rewrites). 20 raw output logs were
saved under `/tmp/ipdl-results/`.

| Case study | Verdict |
|---|---|
| csHelloWorld, auth-to-secure, dhke-otp, coin-toss, coinToss | **Done** |
| GMW-N, 1/2/3/4/5gmwn, modularGMWN, splitGWM-N, singleFile | **Done** |
| cpaSecurity | Ongoing (parse-fails on old `approx assumption` syntax) |
| secure, dhke, el-gamal | Ongoing (parse clean, proof doesn't close) |
| gmwN, noThen4gmwn | no verdict (`quit` precedes their `check-proof`) |

**14 close, 4 `Ongoing`, 2 no-verdict.** This mixed result matches the README's statement that the
repo is mid-port to the new notation. There is **no committed test manifest/CI/expected-output** —
the `check-proof` verdict *is* the ground truth.

---

## 3. Clarifications established

- **`Ongoing`** = the boolean result of `check-proof`'s equality test rendered as a word:
  `new2NF(computeNF(<Ideal>)) == <currentProtocol>` → `Done`, else `Ongoing`. It compares the
  protocol reached by the proof script against the normal form of the named ideal.
- **Expected?** The setup is sound (14 proofs close, incl. the >10⁹-rewrite GMW ones — a broken
  setup couldn't do that). The pass/fail split looks intentional (every cleanly-ported protocol
  closes; the four `Ongoing` are the not-yet-ported holdovers). But no committed ground truth lets
  one certify file-by-file, and the README is itself stale (wrong directory), so it isn't
  authoritative about current pass/fail.
- **parse-fails AND `Ongoing` together** (cpaSecurity): SpeX `load` is command-by-command, not
  all-or-nothing. A command that fails to parse emits a warning and is **skipped**; the rest of the
  file still loads and runs. So the `approx assumption` blocks were dropped, but the protocol
  defs + proof script still executed, and `check-proof` returned `Ongoing` *because* the dropped
  assumptions were exactly what the proof needed.
- **no-verdict** (gmwN, noThen4gmwn): a `quit` appears in the file *before* the `check-proof` line,
  so the verdict line is unreachable dead code on a full-file `load`.

---

## 4. Releases / branches

The upstream repo has **0 releases, 0 tags, and a single branch (`main`)** — nothing newer or more
stable to prefer. The only "alternative" is in-repo: the legacy `IPDL-AS/` implementation has the
complete pre-reorganization case studies (loaded via plain `maude lib/FILE.maude`). The user chose
to stay with `IPDL-Spex/`.

---

## 5. AGENTS.md + IPDL language manual (grounded in original proofs)

Wrote `AGENTS.md` at the repo root covering: setup, running/interpreting proofs, the IPDL language
(declarations, channel contexts, assumption forms, protocols/reactions, the proof-tactic language,
trailing commands), worked examples, hard-won gotchas, a recipe for new proofs, and a source map.

To ground the manual, three **original** toy proofs were written and verified to close (`Done`),
plus one documented limitation:

- `IPDL-Spex/scratch/toy1-otp.ipdl` — minimal one-time pad (`fold`, `use assumption`, `restructure`).
- `IPDL-Spex/scratch/toy2-junkcoin.ipdl` — OTP that samples an unused "junk" coin; proof discards it
  with `drop read` + `absorb`.
- `IPDL-Spex/scratch/toy3-symfrom.ipdl` — ideal hides a simulator channel; proof uses `sym from`.
- `IPDL-Spex/scratch/limitation-doublemask.ipdl` — **intentionally `Ongoing`**; documents that a
  doubly-masked output cannot reduce to a clean single-coin ideal with these tactics.

### Hard-won gotchas discovered by running the tool

1. Assumptions only match a **single bound variable**, not a compound expression (e.g. the OTP
   assumption fires on `xor((m,k))` but not on `xor((xor((m,k1)),k2))`).
2. The normal form does **not** garbage-collect dead samples.
3. `drop read A from B` only fires when `A ::= samp Dist` (a pure sample) and B's read is unused
   (rule `DROP-nf`, `syntax.maude:5249`). You cannot drop a read of a *computed* channel — which is
   why double-masking doesn't collapse to a single-coin ideal.
4. `drop read` leaves the channel orphaned; follow with `absorb`.
5. Fold order matters: the masking sample must end up as the last binding before `return`.
6. `quit` before `check-proof` = no verdict.
7. Debug `Ongoing` by printing `current-protocol` and diffing against the ideal.

---

## 6. Exact commands to run the three toy test cases

```bash
cd <...>/IPDL-Maude/IPDL-Spex/src
export MAUDE_LIB=<...>/IPDL-Maude/.maude-dist
for f in toy1-otp toy2-junkcoin toy3-symfrom; do
  v=$(printf 'load ../scratch/%s.ipdl\n' "$f" \
      | "$MAUDE_LIB/maude" -no-banner -allow-files run-SpeX 2>&1 \
      | sed 's/\x1b\[[0-9;]*m//g' | grep -aoE '^(Done|Ongoing)$' | head -1)
  printf '%-16s %s\n' "$f" "$v"
done
```

Expected:
```
toy1-otp         Done
toy2-junkcoin    Done
toy3-symfrom     Done
```

---

## 7. Naming cleanup

There were briefly two `toy2-*` files: the double-mask (my first original attempt, which would not
close) and the junk-coin (the passing replacement). This was leftover iteration naming. The
double-mask file was renamed to `limitation-doublemask.ipdl` so each numbered toy is a passing case
and the limitation demo is named for what it is.

---

## Artifacts produced

- `<...>/IPDL-Maude/AGENTS.md` — tool usage + IPDL language manual.
- `<...>/IPDL-Maude/.maude-dist/` — Maude 3.5.1 (arm64) install.
- `<...>/IPDL-Maude/IPDL-Spex/scratch/` — `toy1-otp.ipdl`, `toy2-junkcoin.ipdl`,
  `toy3-symfrom.ipdl` (all `Done`), `limitation-doublemask.ipdl` (intentionally `Ongoing`).
- `/tmp/ipdl-results/*.out` — 20 raw case-study run logs.
- This file: `<...>/IPDL-Maude/SESSION-SUMMARY.md`.

