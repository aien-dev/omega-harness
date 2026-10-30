---
name: omega-encoding
description: Write .om that the seeded Omega compiler actually accepts: syntax, types, traps, bounds, register limits, host I/O, byte encoding, compiler errors, verified examples. Use before writing, reviewing or explaining any Omega program.
---

# Omega encoding: writing `.om` that the seeded compiler actually accepts

Use this before writing, reviewing or explaining any `.om` program, before claiming what Omega can or
cannot express, and before encoding bytes, integers or host I/O in Omega.

## 0. Pin and how to re-check it

Merge status, 2026-10-29: compiler batch #904 merged at `e2662fffbce1c21ef09da71c0d8808133a6af8ab`;
x86 integration #923 merged at `bf54b19fa590e8ae925c0e80815516dbf41e7326`.
Earlier pending labels describe the recorded pre-merge tests. Native x86
execution remains NOT_RUN; merging is not hardware qualification.

The quantity and diagnostic audit is merged through omega-mv #914 at main
`d94a503f7aae57d09697046405da2e9a5895be6d`. The older candidate observations
below remain dated history, not claims about the newer integration.

### Current main pin (checked 2026-10-29)

| Item | Value |
|---|---|
| Repository | `github.com/aien-dev/omega-mv`, branch `main` |
| Commit | `bf54b19fa590e8ae925c0e80815516dbf41e7326` (merge of #923, 2026-10-28) |
| Compiler source `seed/omc/omc.om` | 15,624 lines, sha256 `96d51b63c8f451661ac715395078e04f39195e50c9e8fe37746cda121e4d7a1e` |
| Compiler `seed/out/omc` | sha256 `62775d2a391cfef7cde64e387655caa9809a267093112910ec76bfa568f3712b` (equals `seed/out/omc.sha256`; last `seed/omc/size-history.tsv` row `FR9-acceptance-fixes`) |
| Seed loader `seed/rung0/seed0.hex` | sha256 `baaf081fe6055a768485c5d25d7435681ce00b04cfa80b26c48169f78944e14d` |

Hashes were read with `git show origin/main:<path> | sha256sum`; the compiler was not rebuilt
for this note. Both guide examples were rerun on 2026-10-29 with that `seed/out/omc` on a
Linux AArch64 host: `fnv1a32.om` compiled with exit 0 and empty stderr, and `abc` gave
`0b e9 47 1a`, exit 0; `trap_narrow.om` compiled with exit 0 and ran with exit 105; deleting
only its unmasked store (negative control) changed the run exit to 44. The same `fnv1a32.om`
prefixed with `target x86linux` compiled with exit 0 to an x86-64 static ELF that printed
`0b e9 47 1a` with exit 0 under `qemu-x86_64` 8.2.2: EMULATED, not native x86.

### x86 integration as tested before merge (observed 2026-10-28)

Then-draft [#923](https://github.com/aien-dev/omega-mv/pull/923), source
`7ab5fdd067884fd3044155084b39163da1be5c8a`, integrated main and batch 3
(#904). Its `seed/omc/omc.om` and `seed/out/omc` hashes equal the current main
pin above. The seed gate passed, including GATE5. Both guide examples compiled with empty
stderr: `fnv1a32.om` on `abc` exited 0 with bytes `0b e9 47 1a`;
`trap_narrow.om` exited 105, while removing its unmasked store exited 44.

The exact first line `target x86linux` selects the x86 backend. Quantity
negation and complement refuse with error 7 on both targets. Empty text remains
sized with length zero and emits no literal bytes; bounds checks still apply.
Target-specific diagnostic contracts preserve the extra x86 stack-check records
and exact growth-refusal records. See the source's `docs/X86-LINUX.md` and
`seed/omc/x86-diagnostic-contracts.json`. Historical failed experiments and
frozen compatibility rows remain separate from current acceptance.
At main `bf54b19f`, `docs/X86-LINUX.md` is headed "Status: DRAFT; native x86
qualification NOT_RUN". Its last complete differential (section "Current acceptance
contracts", source `7ab5fdd0`, QEMU, EMULATED) reported 15179 PASS, 4 NOT_RUN,
222 EXCLUDED and no FAIL cases, aggregate verdict FAIL because NOT_RUN is not PASS;
`00ebdee8` then passed the two text-write cases (5 PASS), leaving two invalid-fd ioctl
cases unrun. Merged in source; not native x86 qualification.

### Historical quantity/check/next candidate (observed 2026-10-28)

| Item | Value |
|---|---|
| Repository | `github.com/aien-dev/omega-mv`, PR #911 head |
| Commit | `1fe5976d398646b17acabbf6d9dfae96b017e439` |
| Compiler source `seed/omc/omc.om` | 12,520 lines, sha256 `3f937160c823d3a90a6b4cc296f00f517f36ede7fd27a06505c998985ec62240` |
| Compiler `seed/out/omc` | sha256 `d6290c7663c2b4bafb59eb3fdf812476e977f4c144def71095c6248ef9b96b84` |
| Seed loader `seed/rung0/seed0.hex` | sha256 `baaf081fe6055a768485c5d25d7435681ce00b04cfa80b26c48169f78944e14d` |

Host-only observations below used a fresh clone checked out detached at this commit.
Hash and count commands: `sha256sum seed/omc/omc.om seed/out/omc seed/rung0/seed0.hex`
and `wc -l seed/omc/omc.om`; their observed values are in this table.
The fresh-clone `sh seed/omega-from-seed.sh` run exited 0 and printed:

```text
PASS seed0 408 bytes, rebuilds itself
PASS hex1 776 bytes, born from seed0
PASS asm2 2464 bytes, born from hex1
PASS om4 om4.bin 14224 bytes
PASS omc fixpoint d6290c7663c2b4bafb59eb3fdf812476e977f4c144def71095c6248ef9b96b84
PASS fwd exit 0
PASS fwd runs, exit 7
PASS mut exit 0
PASS mut runs, exit 0
PASS cycle exit 9
PASS assign exit 7
RC=0
```
This reproduces the independently recorded binary digest, not an independent
correctness proof or a full gate5/host qualification.


### Historical general-language pin (2026-10-25)

| Item | Value |
|---|---|
| Repository | `github.com/aien-dev/omega-mv`, branch `main` |
| Commit | `55f9b0af47de70a3d06357985cf99ae3e45c016f` (2026-10-25, merge of PR #766) |
| Compiler source `seed/omc/omc.om` | 7,838 lines, sha256 `2e35db7d6c2066fb4a20d19b8a9579c44caca9c9937b0594a89422010b673637` |
| Committed compiler `seed/out/omc` | sha256 `734f0a39319fbfa930bcc62cb30dbf06d47d78e126372b74f5baf478e6126b61` |
| Seed loader input `seed/rung0/seed0.hex` | sha256 `baaf081fe6055a768485c5d25d7435681ce00b04cfa80b26c48169f78944e14d` |
| Language document `docs/one_layer/OMEGESE.md` | blob `89a27d2e68519eb45c103abb23dfe05a68d8ad69` |

Every citation links to that commit: `https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/<path>#L<n>`.

### Merged IR/wasm backend (2026-10-26)

Source: omega-mv main `3364a1dfd0ca50c1b82bf19ba34932d8bc10a7a9`,
[#787](https://github.com/aien-dev/omega-mv/pull/787), after the separate boolean IR
fix [#786](https://github.com/aien-dev/omega-mv/pull/786), merge
`87489da90a1a7a750fd4a7f9eeb005b1221721f6`. Source merge is not deployment.

| Backend item | Verified value |
|---|---|
| `seed/omc/omc.om` | sha256 `bd64bb6f20952818e155328f429ab6cd36bba213e3e00b60d796336f9d0c9343` |
| `seed/out/omc` | sha256 `befb8199db8c4bca188a494dfaac2a6f0ee5c36fcea11188c75666c55fce8a1d` |
| Prior main compiler at `ec07d25d5506d33567adbd4d9b518177b7933bad` | sha256 `499236398aabc652e2dae27fe4bdc4c5db3d3726314356e0dbef23e3345803ec` |
| `seed/wasm/irwasm.om` | sha256 `8dc7bf602eb882e26a18b9e2dc4cacdd3ad68827215e71874b14c4acb237ccf8` |

The compiler corrects IR-mode boolean AND/OR/XOR typing: all three previously
exited 7 and now compile to bytes identical to the C encoder, accepted by checker2.
Boolean arithmetic and shifts remain refused. The om4-built compiler and two
self-rebuilds are byte-identical; the full compiler referee/wrong-word-mutant gate
ended `GATE5 PASS`. Retained evidence:
`evidence/IR-BOOL-20261026/` from merged #786.

`seed/wasm/irwasm.om` is a seeded Omega backend for **scalar OSC IR v1**, not an
arbitrary foreign-source converter. It writes an import-free WebAssembly MVP module
and exports each function under its ASCII IR name. ABI tags bool/u8/u16/u32/i8/i16/i32
map to i32, u64/i64 to i64, void to no result. Internal canonical i64 locals preserve
width and signedness; signed overflow, invalid shifts/casts/division, explicit traps
and lowered loop guards become wasm runtime traps. Wasm does not expose Omega's
numeric trap code. Supported instructions: CONST/MOV/BIN/UN/CMP/CAST/CALL/TRAP/BR/CBR/RET.
Limits and exact mappings:
[`OMEGESE.md` section 11.1](https://github.com/aien-dev/omega-mv/blob/3364a1dfd0ca50c1b82bf19ba34932d8bc10a7a9/docs/one_layer/OMEGESE.md#L914).
Versions other than 1, memory/reference/arena/pool/slice tags and operations refuse.
The module is buffered so parse/type/capacity refusals have empty stdout.

Host command: `nice -n 19 sh seed/wasm/gate-irwasm.sh`.
Observed on Linux AArch64, Node v24.21.0 used **only as a read-only oracle**:
`PASS parity cases=129 units=13 traps=29; every export exercised`.
The parity checker rejected an add-to-sub IR-byte mutant with exit 1; two translations
of every unit were identical, and named refusal exits were pinned with empty stdout.
Source-bound full logs/artifacts: `evidence/IRWASM-20261026/` from merged #787.
Re-exercised at head `0e81d5a6f990000f1dca50f05f0c6050f7cdf9a5` before merge;
the backend gate did not run browser integration or GPU qualification.

Existing guide examples were rerun with this compiler: FNV `abc` emitted
`0b e9 47 1a`, compile/run exit 0; `trap_narrow.om` compiled with exit 0 and ran with
exit 105. Removing only the unmasked store (negative control) changed the run exit
to 44. The historical examples and their old measured evidence below remain intact.

Downstream browser integration is separately **merged**, not qualified as deployed here:
[aienos.com #109](https://github.com/aien-dev/aienos.com/pull/109), merge
`a6aec5e2628804abd27101ba184d7a319f95774e`; #108 closed as superseded.
Its existing OM-1 compiler pin `734f0a39319fbfa930bcc62cb30dbf06d47d78e126372b74f5baf478e6126b61`
compiled the same upstream `irwasm.om` to **identical** backend bytes
`2d235a70c39ac1461a09c41e10e058d962b767ae83893cdcfc693ef6c47182e6`.
No site compiler pin bump was necessary. `irwasm.provenance` records the source and build.
The site unit `om/hive-page/fresh.om` accepts clock/timestamps, the caller's stale
threshold and fetch status; `check.sh` uses an `ir` header and the pinned backend to
produce `public/wasm/hive-fresh.wasm`, sha256
`af9e1fb0d9e736949b2aa83c0bf978ac2d98f37428a8507176abaa1dd50b777b`.

`freshness-js.om` emits the loader and explicit JS load-failure fallback.
`data-fresh-engine="wasm|js"` discloses the engine; the four state codes are
0 fresh, 1 stale (strictly past the supplied threshold), 2 backwards timestamp,
3 interrupted fetch. Source-clock trust gating remains separate. Browser-supplied
fractional elapsed age is rounded upward only at the integer wasm boundary,
preserving a strict comparison against the validated integer threshold.
`sh om/hive-page/gate-fresh.sh` compared ten boundary cases with actual native
AArch64 execution and read-only Node wasm execution. Real Chromium poll fixtures
exercised all four wasm states, incorrect-MIME arrayBuffer loading, and one-byte
module corruption falling back to JS. Corrupting that byte on disk made the
x86_64-compatible asset hash test fail with exit 101; original bytes were restored.
Historical evidence/fixtures/results remain `om/hive-page/fresh-evidence.log` and sibling
`fresh-*` files on closed #108, not in the #109 main tree. These were synthetic acceptance
feeds, not production telemetry. The helper source and wasm artifact were carried over
byte-identically. Production browser verification, deployment and GPU qualification
remain NOT_RUN by this guide update.

### Current main, OM-2 contract and the 1.0 roadmap (2026-10-26)

omega-mv main `19f014f2178ca9cd3473728fe00a475a2398c409` (merge of
[#801](https://github.com/aien-dev/omega-mv/pull/801)) changed no compiler file after `3364a1dfd`:
`seed/out/omc` and its sidecar are still `befb8199...8a1d`, the backend pin above. Read only; not
rebuilt for this note.

- **OM-2 is merged.** The opt-in profile (first non-`note` line `om2`, `om2 diag` or `om2 fmt`)
  landed through [#750](https://github.com/aien-dev/omega-mv/pull/750),
  [#753](https://github.com/aien-dev/omega-mv/pull/753),
  [#754](https://github.com/aien-dev/omega-mv/pull/754),
  [#764](https://github.com/aien-dev/omega-mv/pull/764), the SHA-256 cutover
  [#776](https://github.com/aien-dev/omega-mv/pull/776) and the qualification harness
  [#752](https://github.com/aien-dev/omega-mv/pull/752). What `omc` implements is
  [OMEGESE section 9.13](https://github.com/aien-dev/omega-mv/blob/19f014f2178ca9cd3473728fe00a475a2398c409/docs/one_layer/OMEGESE.md#L624).
- **The normative OM-2 contract is on main since #801**, byte-identical to branch `om2/contract`
  @ `23fc7f5c9`: `spec/omegese-om2.md` sha256 `245bb74e940c811ab325e6fabb33f0599ad46aaf6b9064fd2750c43980040881`,
  `spec/omegese-om2-errata.md` (E1..E18) `7b8c1ea58738806cfc29a145a2d60ede5e1c399366a1665004859304bdd2d671`,
  `spec/omegese-om2-measure.md` `507702d0e69ccc76cefb3675e2b22f7279497d362b42ee3043f3a1057aa17bad`.
  Before #801 these files existed only on that branch; older notes that call the contract "missing"
  or "draft only" are stale.
- **Omega 1.0 roadmap:** [`docs/ROADMAP-1.0.md`](https://github.com/aien-dev/omega-mv/blob/19f014f2178ca9cd3473728fe00a475a2398c409/docs/ROADMAP-1.0.md),
  milestones RM1..RM11 with acceptance gates, tracked in
  [#790](https://github.com/aien-dev/omega-mv/issues/790) (gate coverage),
  [#791](https://github.com/aien-dev/omega-mv/issues/791) (contracts),
  [#792](https://github.com/aien-dev/omega-mv/issues/792) (efficiency),
  [#793](https://github.com/aien-dev/omega-mv/issues/793) (memory model),
  [#794](https://github.com/aien-dev/omega-mv/issues/794) (strings),
  [#795](https://github.com/aien-dev/omega-mv/issues/795) (modules),
  [#796](https://github.com/aien-dev/omega-mv/issues/796) (bridge FFI),
  [#797](https://github.com/aien-dev/omega-mv/issues/797) (diagnostics),
  [#798](https://github.com/aien-dev/omega-mv/issues/798) (targets),
  [#799](https://github.com/aien-dev/omega-mv/issues/799) (compiler trust and C retirement),
  [#800](https://github.com/aien-dev/omega-mv/issues/800) (stability). It is a plan, not a readiness
  claim; a gap listed there is missing, not impossible.
- **Gate coverage at main:** `seed/wasm/gate-irwasm.sh` and `seed/qualification/om2/gate.om` are
  merged but no `tests/gates/*.gate` runs them (RM1, #790). `tests/gates/run.sh seed` passing says
  nothing about OM-2 fixtures or irwasm; run those yourself and report them separately.
- **OM-2 defect** [#775](https://github.com/aien-dev/omega-mv/issues/775), closed 2026-10-29: an
  empty `text` passed to a `view uchar *` parameter was reported to see the next text's bytes
  (`len` 32, not 0). Fixed by #897 inside the #904 merge `e2662fff`, an ancestor of main
  `bf54b19f`: an empty `text` is now a sized array of length 0. On 2026-10-29, with main
  `seed/out/omc` `62775d2a...712b` (AArch64 host), `seed/omc/om2/s4/v-text-empty.om` compiled
  and ran with exit 0, and `v-bad-text-empty.om` refused with `omc: line 5: error 21`. Compiler
  pins in this file older than the #904 batch predate the fix.

### Merged compiler batch: contracts, capacity and f32 returns

Omega main `91b3c9326919faf1491e29ae9276929e8349b8c7` merges #768, including executable
contracts #807 and f32 return repair #812. Status: MERGED and EXERCISED on arm64 Linux;
not GPU qualification. The batch receipt is `evidence/BATCH-189ada82/README.md` at that commit.
Host qualification with C oracles, gate5, contract checks and mutants, float and capacity
boundaries and IR-trap mutants passed. Frozen C2 retains three expected failures from the
intentional capacity expansion; `manifest-c2-capacity.txt` records those three newly accepted
units separately. Do not claim the frozen manifest itself passes or rewrite it.

| Item | Value |
|---|---|
| `seed/omc/omc.om` | 11,555 lines, sha256 `de7f5c8035608c244fb612ed5bf7691ae22d3b64735a5e7122fe9d0c67853d2b` |
| `seed/out/omc` | sha256 `189ada82e00bb440cb41351bd252cc68c54a1b1050051cc42a1df700081c3b54` |
| Language text | `docs/one_layer/OMEGESE.md` section 12, "Executable native contracts, version 1" |
| Fixtures | `seed/omc/contracts/contracts.cases` (44 rows), `mutant.cases` (2 mutants, both must FAIL) |
| Size ledger | `seed/omc/size-history.tsv`, row `RM2-capacity-f32` with that sha256 |

The capacity is 4096 functions, 8192 names and 524288 call sites per unit. Native `to f32`
returns now work: internal void return code 12 is distinct from f32 value type 10 and f64 11.
`gate8.sh` exercises f32 values and rejects void-as-value (7) and f32 fallthrough (18).
Consumers must explicitly migrate and qualify their own pins; a main merge does not deploy them.

### Compiler batch 3 (merged #904; superseded on main by #923)

[omega-mv#904](https://github.com/aien-dev/omega-mv/pull/904) merged at
`e2662fffbce1c21ef09da71c0d8808133a6af8ab`; that merge carries the same `omc.om` and
`seed/out/omc` hashes as the tested source below. Tested on 2026-10-28 at
`7bce2a4fddc43268ac2af09cddc4a5a67d78312d` over main `d94a503f7`, merges #896
(integer `bitcast` re-extension), #897 (empty `text` sized `nm_len 0`, view count 0; closes the
#775 defect above), #898 (function names that start with a keyword) and #903 (`pread`/`pwrite`
destination writable check) and reseeds `seed/out/omc` to sha256
`1811d4047f118440cc35e995261af6768b606148cb17e18c42158afd15fcad58` (12,577 source lines, 418
functions, size-history row `RM-batch3-current`). Compiler source SHA-256:
`d33ef5cab603f85a5cbcbdfdd55391f2ce0448bfcbc5fecbf0393599b636ef04`.
Status when recorded: HOST_TESTED before merge; merged in source, not deployed. The targeted regression cases, seed/GATE5,
OM-2 s1 through s4 and their mutants, current-language quantity cases, diagnostic experiment,
bindings/parity, JSON/HTTP, WebAssembly, CPU OpenCL, IR-trap checks and local TLS suites pass.
The frozen C2 corpus still fails its six historical assertions; its capacity and batch2
replacement manifests pass. TLS live-internet parity and GPU gates are NOT_RUN.
The earlier `397346de` receipt remains immutable history in `evidence/BATCH-397346de/`.
At this new pin, the guide's FNV example with `abc` outputs `0be9471a` (exit 0), the narrowing
example exits 105, and removing its unmasked store exits 44. All compile with empty stderr.
Main has since moved to the x86 integration compiler `62775d2a...712b` (section 0); cite
`1811d404...ad58` only as the #904 merge pin, never as current main or as x86 qualification.


Re-check recipe (read only, in a fresh clone, never in a shared working tree another agent may be using):

```sh
git clone --depth 1 https://github.com/aien-dev/omega-mv.git omega-src
git -C omega-src rev-parse HEAD
sha256sum omega-src/seed/omc/omc.om omega-src/seed/out/omc
```

If the hashes differ from the table, the rules below are unverified for that tree. Read the changed
parts of `OMEGESE.md` and `omc.om` first.

Status words used here: **implemented** (in `omc.om` at the pin), **gated** (a committed gate script
or fixture checks it), **author-verified** (the repository records a run; not re-run for this note),
**proposed** (described, not on `main`). Nothing here is hardware qualification, and nothing here is
deployed anywhere.
### `qty`: checked quantities, not modular words

At the current candidate pin, `qty` is distinct from `ulong`. Do not silently mix
quantity arithmetic with modular words. Save this as `qty-overflow.om`:

```text
fn main to int
a = qty 18446744073709551615
b = qty 1
c = add a b
ret int 0
end
```

Use the section 1a compile/run recipe with `prog.om` replaced by `qty-overflow.om`.
Observed: compile exit 0, empty stderr; run exit 101, empty stdout and stderr.
The boundary literal is copied from the pinned `tests/quantity/qty_add_overflow.om`.

Negative control: change only `a = qty 18446744073709551615` to `a = qty 0`
and `ret int 0` to `ret int c`. Observed: compile exit 0, run exit 1,
empty stdout/stderr. The trap is not an unconditional program exit.

For the modular contrast, replace both `qty` spellings with `ulong` and return
`int c`. Observed: compile exit 0, run exit 0, empty stdout/stderr: `ulong`
still wraps. Mixed operands are refused, not coerced:

```text
fn main to int
a = qty 1
b = ulong 1
c = add a b
ret int 0
end
```

Observed: compile exit 7, empty stdout, stderr:

```text
omc: line 4: error 7
```

### `check` and `next`: text diagnostics and prefix context

These are leading input lines to the seeded compiler, not command-line flags
and not the legacy C `omegese next` interface. At the current candidate pin:

```sh
printf 'check\nfn main to int\nret true\nend\n' | seed/out/omc
```

Observed stdout (exit 7):

```text
omega-diag v1
om1 TYPE span=19-23 FINAL REPAIRABLE code=7 line=2 expect=value found=true
```

Observed stderr:

```text
omc: line 3: error 7
```

Negative control: replace `ret true` with `ret 0`. Observed exit 0, empty stderr,
stdout exactly `omega-diag v1` followed by a newline. Check mode emits diagnostics,
not an executable. Its record line is relative to the program after the leading
mode line; the ordinary stderr line includes that mode line.

```sh
printf 'next\nfn main to int\na = ulong 3\nwhile lt a 9\n' | seed/out/omc
```

Observed exit 0, empty stderr, stdout:

```text
omega-query v1
scope: function
constructs: ret if while for trap exit put end note break continue
names:
a ulong local
functions:
fn main to int
```

This is prefix context (scope, constructs, visible names and signatures), not a
promise of an exact next-token mask. The unfinished loop above is intentional.
Negative control: compare the observed query with a copy whose name line is
`a ulong param`. Observed comparison: unequal (`REJECTED`), so a local/parameter
kind error is detectable.


## 1. Which language is which

| Thing | Where | Status |
|---|---|---|
| **Omegese**, the `.om` language | `docs/one_layer/OMEGESE.md`, sections 1-11 on main | current |
| Profile **OM-2** (opt-in per unit) | contract `spec/omegese-om2.md` plus errata E1..E18 (normative, on main since #801); implemented subset in OMEGESE section 9.13 | current, merged; host evidence only |
| WebAssembly MVP backend `seed/wasm/irwasm.om` | OMEGESE section 11.1; scalar OSC IR v1 only | merged; gate not in any manifest (#790) |
| Executable contracts (`requires`, `ensures`, `invariant`, `result`) | OMEGESE section 12; section 11a here | merged in #768, including #807; host-exercised |
| The seeded compiler `omc`, written in `.om` | `seed/omc/omc.om`, built `seed0 -> hex1 -> asm2 -> om4 -> omc` by `sh seed/omega-from-seed.sh` | on `main` since PR #696 ([README.md#L27](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/README.md#L27)) |
| Keyword table, the authority for reserved words | `fn kw` in `seed/omc/omc.om` ([#L375](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L375)); same list in `seed/omc/genkw.awk` ([#L16](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/genkw.awk#L16)), parity checked by `seed/omc/gate18.sh` | gated |
| C translator `tools/omegese/omegese.c` and `omegese next` | OMEGESE sections 1-8 | legacy referee and token-mask tool, not the compiler |
| `spec/omega-language-v0.md` (`let x: u8 = ...`, `fn f(x: T) -> T { }`, `//` comments) | Visor console line language, lowered by C in `src/language/` ([#L1-L13](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/spec/omega-language-v0.md#L1-L13)) | history; never write `.om` in that syntax |
| C under `src/`, including `src/compiler/` | `main` | unfinished migration ([README.md#L50](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/README.md#L50)) |

OMEGESE sections 2-6 describe the C translator's rules; sections 9-12 describe `omc` at
`91b3c932`. Where they
disagree, `omc` is what compiles your program:

- Calls: the C path allows calls to earlier functions only. `omc` reads every signature first, so a
  call may name any function in the unit; recursion is allowed in program mode and refused (error 9)
  in function mode ([OMEGESE#L409-L417](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L409-L417)). IR mode
  (a leading `ir` line) again allows earlier functions only ([#L755-L763](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L755-L763)).
- `new` lengths: C path 1..64 ([#L85](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L85)); `omc` 1..16777216
  per array and at most 4 MiB of local arrays per program ([#L460-L463](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L460-L463), gate12).
- Unread names: the C path refuses a name that is written and never read ([#L37-L38](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L37-L38)).
  `omc` fixtures contain unread results (`w = write ...` in `seed/rung4/tests/bytecopy.om`). Do not
  rely on either behaviour; check every result you write.

Language rule: `.om` is the only forward first-party implementation language. C, Rust and shell that
remain are unfinished migration, not exemptions ([README.md#L50](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/README.md#L50)).
- **Tokenization is not a language rule:** Language design optimizes for verified programs produced
  per unit of inference cost (measured on held-out tasks by first-attempt correctness, successful
  compilation, verified behavior, latency, and total compute cost). Fitting words into single tokens
  under particular tokenizers (e.g. Qwen3) is an empirical measurement and cost consideration, not a
  binding rule of Omega or Omegese. The compiler-guided interface remains portable across models and
  tokenizers.

## 1a. Compile and run (Linux AArch64 host only)

```sh
OMC=omega-src/seed/out/omc                 # after checking its sha256 against section 0
"$OMC" < prog.om > prog 2> prog.err
compile_rc=$?
printf 'compile rc=%s\n' "$compile_rc"
cat prog.err
if [ "$compile_rc" -eq 0 ]; then
    chmod +x prog
    ./prog
    run_rc=$?
    printf 'run rc=%s\n' "$run_rc"
fi
```

Only compile status 0 permits running the new output. Check the runtime status
separately: a program may intentionally return nonzero (modulo 256), and a trap
has its own status. Empty compiler stderr alone is not a successful compile.
To rebuild the compiler from the seed instead, run `sh seed/omega-from-seed.sh` from the root of a
disposable clone (prints `PASS ...` lines and `RC=0`). It writes `build/seed/`, rewrites the tracked
`seed/rung4/om4.s` size word and `seed/rung4/om4.bin` (`seed/rung4/mk4.sh`), and rewrites
`seed/out/omc` and `seed/out/omc.sha256` with whatever fixpoint it reached, so check `git status`
and compare the new hash with the pin yourself ([omega-from-seed.sh#L52-L60](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omega-from-seed.sh#L52-L60)).
Full host qualification is `sh seed/host/check.sh` (author-verified on 2026-10-23 per
`seed/host/README.md`); a host pass is never hardware qualification.

## 2. Program shape

```
note any text to the end of the line
const LIMIT ulong 64
global buf uchar 4096
text msg "hello\n"
struct Pair a uint b long end
fn helper x ulong y ulong to ulong
ret add x y
end
fn main to int
ret 0
end
```

- One statement per line; words separated by spaces or tabs; no commas, semicolons, braces or
  indentation. `note` is the only comment form; the rest of its line is not read
  ([omc.om#L4873](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L4873)).
- `fn NAME (PARAM TYPE)* to TYPE|void` ... `end`. At most 6 parameters (error 9). `main` takes none.
- A unit with `fn main` is a static AArch64 Linux ELF (program mode). Without `fn main` the output is
  bare function words (function mode): it compiles with exit 0 but is **not runnable**. Host words
  and `global` are error 9 there, and `text` is program-mode only ([#L400-L407](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L400-L407), [#L454-L459](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L454-L459), [#L629](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L629)).
- Program exit status = `main`'s value mod 256 (0 for void main). `exit v` (v `int`) ends at once.
  A runtime trap exits **100 + code**. Keep your own exit codes below 100, or a result is
  indistinguishable from a trap ([#L404-L407](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L404-L407), [#L484](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L484)).
- A value-returning function used as a statement is error 7. Bind the result: `w = write 1 buf 0 n`.
- A function with a return type whose `end` is reachable is error 18. `ret`, `break`, `continue`,
  `trap` and `exit` end the path; a line after them in the same block is error 17.

## 3. Values: prefix words, fixed arity

Every operator is a prefix word with a fixed number of values, so nesting needs no brackets:
`add mul r r 4` = (r*r)+4; `eq rem n 2 0` = (n mod 2)=0 ([#L73-L75](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L73-L75)).
Evaluation is innermost first, left to right ([#L137-L141](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L137-L141)).

| Words | Takes | Result |
|---|---|---|
| `add sub mul div rem and or xor shl shr` | 2 | same type as operands |
| `eq ne lt le gt ge` | 2 | `bool` |
| `neg not` | 1 | negate / bool not or bitwise not |
| `TYPE v` | 1 | typed number (`uint 5`) or checked conversion (`ulong x`) |
| `bitcast TYPE v` | 2 | same-width reinterpretation (section 5) |
| `get A i`, `get S f`, `get S f i` | 2-3 | load from array / struct field |
| `len A` | 1 | array length, `ulong` |
| `sizeof T`, `offsetof S f` | 1-2 | compile-time `ulong` ([#L591-L599](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L591-L599)) |
| `FUNCTION args` | its parameter count | call |
| `( v )` | 1 | grouping only; adds no code ([#L583-L588](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L583-L588)) |

There are no infix operators. `+ - * /` are not keywords (they are absent from the keyword table);
OMEGESE 9.9 ([#L564](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L564)) writes "`+ - * /`" but its own
float fixtures use `add sub mul div` (`seed/omc/tests/float/arith.om`). Write the words.

`and` / `or` evaluate **both** sides; there is no short circuit ([#L104](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L104)).
`and lt i len a eq get a i 0` still loads `a[i]` when `i` is out of range and traps 3. Test the
bound in an outer `if`, then load inside it.

## 4. Types, literals, typing

- Types: `bool uchar ushort uint ulong byte short int long` = bool, u8, u16, u32, u64, i8, i16, i32,
  i64; aliases `u8 .. u64`, `i8 .. i64`; floats `f32 f64` ([#L109-L111](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L109-L111), [#L549](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L549)).
  `uchar` is an 8-bit number, never a character ([#L442](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L442)).
- **No implicit conversions** (decision 20): both values of an op, a value and the name it is written
  to, an argument and its parameter must have the same type, else error 7
  ([#L440-L442](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L440-L442)). `add get b 0 uint 1` with `b` a
  `uchar` array is error 7; write `add uint get b 0 1`.
- A bare number takes the type of the other value in its op, or the type its place needs
  (`ret 0`, an argument, an index). Two bare numbers in one op, or a bare number with no context
  (`x = 5` for a new `x`), is error 11: write `x = uint 5` ([#L94-L100](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L94-L100), gate15 `e11_ambiguous`).
- Integer literals: decimal only, optional leading `-`, at most 20 digits, and the value must fit the
  type it takes (error 8), e.g. `long` -2^63..2^63-1, `ulong` 0..2^64-1. A non-digit (`12a`,
  `0x10`) is error 6 ([#L443-L446](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L443-L446)). No hex, no octal,
  no character literals: write byte 65, not `'A'`. `true` / `false` are the bools.
- Shift with a bare left operand takes the shift amount's type: `shl 1 sh` with `sh` a `uchar` shifts
  a `uchar`. Write `shl ulong 1 sh` when you mean 64 bits ([#L97-L99](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L97-L99)).
- Float literals: `digits.digits[e[+|-]digits][f32|f64]`, a digit on both sides of the dot; exact
  decimal-to-binary rounding; overflow to infinity is error 8 ([#L555-L563](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L555-L563)).
- Named constants: `const NAME TYPE LITERAL` at top level, visible only after their definition, also
  usable as array lengths, `trap` codes and `while` bounds ([#L602-L622](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L602-L622)).

## 5. Arithmetic, conversions and traps

From OMEGESE 9.3 ([#L429-L447](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L429-L447)), gate fixtures in
`seed/rung4/tests/t_*.om` with expected statuses in the matching `.want` files:

| Behaviour | Result | Fixture |
|---|---|---|
| Unsigned `add sub mul shl neg not` | wrap mod 2^width | gate 3 |
| Signed `add sub mul neg div shl` out of range (incl. MIN div -1) | trap 1, exit 101 | `t_ovf.om` -> 101 |
| `div` / `rem` by zero | trap 2, exit 102 | `t_div0.om` -> 102 |
| Array index or I/O slice out of bounds (also a negative signed index) | trap 3, exit 103 | `t_bounds.om`, `t_rw.om`, `t_write_wrap.om` -> 103 |
| Loop past its bound | trap 4, exit 104 | `t_loop.om` -> 104 |
| Conversion whose value does not fit the new type | trap 5, exit 105 | `t_conv.om`, `t_conv_sign.om` -> 105 |
| Call or local array below the 8 MiB stack | trap 6, exit 106 | `t_stack.om` -> 106 |
| Shift amount outside 0..width-1 (every type) | trap 7, exit 107 | gate 4 s.5 |
| `trap N`, N 1..14 | exit 100+N | `seed/omc/tests/loop/t1.om` -> 105 |

Signed `div` rounds toward zero; signed `rem` has the dividend's sign; signed `shr` keeps the sign.
Comparisons follow the operand type: with `ulong` x = 2^64-1 and y = 1, `lt x y` is false
(`t_cmp_unsigned.om` -> 0).
Floats never trap: IEEE results, NaN compares false except `ne` ([#L564-L567](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L564-L567)).

Conversions are **checked**, not truncating. `uchar x` with x = 300 traps 5. Mask first:
`uchar and x 255` (see `examples/trap_narrow.om`). `long x` with x = 2^63 as `ulong` traps 5.

`bitcast T v` reinterprets bits of the same width; `bool` and different widths are error 7, a bare
number is error 11 ([omc.om#L4640-L4662](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L4640-L4662)). Every fixture
uses it between floats and unsigned integers. The code also accepts integer-to-integer of equal
width (`bitcast int u`), but no fixture covers that; prove it with your own fixture before relying on
it, or decode signed values arithmetically (section 8).

## 6. Names, scope, registers

- Function names: up to 63 characters. Every other name (variable, parameter, global, text, const,
  struct, field): **at most 8 characters**, else error 13 ([#L423](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L423), gate15 `e13_long_variable`).
- A new name may not equal a visible name, a function or a struct (error 10). A name first written
  inside `if` / `else` / `while` / `for` dies at its `end`; assign it before the block to carry it out.
- Reserved words cannot be names. Besides the obvious ones, the table reserves `len read write text
  new free own view edit global note exit const bound` and every host word, including `stat link bind
  send recv random fork cwd chdir umask accept listen connect waitio memfd mapget mapset`. `bits min
  max abs sqrt` are deliberately **not** reserved (gate11). Check the keyword table, not memory.
  A reserved word used as a name is **error 16** (unknown word), not error 10, so the message does
  not say "reserved". Observed on omc `2c824805…` (omega-mv `e5b32567`), host only: `fn f len ulong
  to ulong` gives `omc: line 1: error 16`, and a local `len = ulong 3` gives `omc: line 2: error 16`.
  The same program with the parameter renamed to `n` compiles (exit 0). Use `n`, `cnt` or `size`.
- **16 registers per function** (pool x0-x6, x8-x15, x17; [omc.om#L82-L91](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L82-L91)).
  Parameters, every live name, the hidden counter of each open `while` / value-count `for`, and the
  temporaries of the current statement all draw from it; error 12 when the next one is not available
  ([omc.om#L1479-L1487](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L1479-L1487)). gate15 pins the 17th live name as
  error 12. In practice keep functions to about 10 live names and split deep nested expressions or
  helpers into smaller functions.

## 7. Arrays, globals, structs

From OMEGESE 9.4 and 9.7 ([#L449-L474](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L449-L474), [#L494-L516](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L494-L516)):

- `global NAME T N`: zero-filled pool in the program image, N 1..16777216, at most 1 GiB of pools
  total (error 23), program mode only. Globals are visible in every function.
- **A global is never passed as an argument** (error 19). A function works on a global by naming it
  directly; parameters carry scalars and local arrays only. The arrays of one call must be distinct
  names (error 19).
- `NAME = new T N v`: local stack array, N elements each v, only in a function's outermost block
  (error 9 inside a block), at most 4 MiB of local arrays (error 23). **N must be a literal or an
  integer `const`**; a variable length is error 16 (gate16 `len_varname`), so there are no
  run-time-sized arrays. No heap and no `free` (decision 24; `free` is error 9).
- Array parameters state type and length exactly: `a view uchar 64` / `a edit uchar 64` (error 7 for
  any other length), so there are no length-generic array parameters; use fixed sizes, an integer
  `const`, or a global. A view or a `text` is read-only (error 20 on `put` or `read` into it).
- `get A i`, `put A i v`, `len A`. A constant index outside the array is error 21; any other index is
  checked at run time (trap 3). No word yields an address or adds to one.
- `text NAME "bytes"`: read-only `uchar` array; escapes only `\n \t \\ \"` (error 22 otherwise).
- Structs: fields are 8-byte cells, no padding, no packed layouts (a `uint` field still takes 8
  bytes). `read`/`write`/`len` of a struct are error 7. Never treat a struct as a wire layout.

## 8. Loops and bounds

From OMEGESE section 3 and 4 ([#L58-L60](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L58-L60), [#L131-L135](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L131-L135)) and [omc.om#L5493-L5494](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L5493-L5494):

- `while COND bound N`: at most N body passes, N 1..999999 (error 8 outside). Without `bound` the
  limit is 1000000 passes. One more pass traps 4. The condition is recomputed before every pass.
- `for i T K` runs i = 0..K-1. A literal K needs no hidden guard; a name or expression K gets the
  1000000-pass guard.
- `break` and `continue` (`omc` only, gate7); `continue` in a `while` re-runs the condition and bound.
- Choose N from a proof, not a guess. Example: a read loop where every pass adds at least one byte to
  a 65536-byte buffer needs at most 65536 passes. Write that reasoning in a `note`.
- No loop runs forever and recursion is bounded by the 8 MiB stack. A long-running service needs an
  explicit epoch/restart design, not a bigger bound.

### Compiler limits checked at main `bf54b19fa` (2026-10-29)

Checked with main's `seed/out/omc` (sha256 `62775d2a391cfef7cde64e387655caa9809a267093112910ec76bfa568f3712b`)
and `seed/rung4/om4.bin` (`d0da40e4…19efab`) on the Linux AArch64 host; facts 2, 3, 5 and 6 were also
reproduced with the omega MCP `run` tool (pin `e5b32567`, compiler `2c824805…029f`). Host evidence only.

- **Value slots.** `fn alloc` ([omc.om#L2848-L2855](https://github.com/aien-dev/omega-mv/blob/bf54b19fa590e8ae925c0e80815516dbf41e7326/seed/omc/omc.om#L2848-L2855))
  fails error 12 when `npar + nslot >= 16`. There is no spilling: split the function. Fifteen
  `aN = int N` names summed in one nested `add` chain already give `omc: line 17: error 12`.
- **Loop guard.** `for i ulong k` with a name `k = ulong 1000001` compiles and exits 104 (trap 4,
  LOOP_BOUND); `k = ulong 1000000` exits 0. A literal count is unguarded: `for i ulong 2000000` exits 0.
- **Float literal mantissa.** The digits of a float literal, read without the dot, must fit in
  2^63-1 = 9223372036854775807 (Wolfram `2^63-1`). `922337203685477580.7f64` compiles;
  `92233720368547758.08f64` and `9223372036854775807.0f64` are error 8.
- **Forward calls** are allowed: `main` calling `f` defined after it compiles and exits 7.
- **Hex.** OM-1 rejects `0x10` with error 6; under `om2` the same unit compiles and exits 16.
- **x86 target.** A first line exactly `target x86linux` emits an x86-64 ELF; with a leading space it
  is error 16.
- **om4 function cap (contributors to `omc.om`).** om4, which compiles the first `omc` from
  `seed/omc/omc.om`, accepts at most 511 functions plus `main`. Main's `omc.om` has exactly 512
  `^fn ` lines, so it is full: adding one function makes `om4 < omc.om` print
  `om4: line 15610: error 14` and the seed chain breaks. Fold single-call helpers into their caller,
  or extend om4, before adding any function. Check with `grep -c '^fn ' seed/omc/omc.om`.

## 9. Host I/O: counts, errors, evidence

Base words ([#L476-L486](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L476-L486)): `read fd A off n`,
`write fd A off n`, `pread`/`pwrite fd A off n pos`. Native host words ([#L624-L747](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L624-L747)):
files, directories, clocks, sockets, poll, processes, arguments, environment, random.

- Types: `fd`, flags, modes, pids are `int`; offsets, counts, capacities `ulong`; file positions and
  intervals `long`; buffers are `uchar` arrays. Every result is `long`.
- A result is a **count or minus the Linux errno**. `read` may return fewer bytes than asked, 0 at
  end of input, or a negative error; `write` may write fewer bytes. Loop until done, and handle
  `k < 0` and `k == 0` separately. Do not copy `seed/rung4/tests/bytecopy.om`'s loop as error
  handling: it ends silently on a negative result.
- Slices are checked before the kernel call: `n <= len A` and `off <= len A - n`, else trap 3. `off`
  does not wrap (`t_write_wrap.om`).
- Count conventions differ per word and are part of the contract: `cwd` includes the trailing zero;
  `readlnk` has none and a result equal to capacity may be truncated; `sockname` may return more
  than capacity; `argread`/`envread` return -105 on short capacity without touching the buffer.
- `send` adds `MSG_NOSIGNAL`; `write` does not, so writing to a closed pipe follows ordinary Linux
  SIGPIPE behaviour [INFERENCE from Linux semantics; not gated here].
- These words carry the process's permissions; they are not a sandbox. Linux ARM64 is the only
  qualified host ([#L739-L743](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L739-L743)).
- Evidence for any host claim: compiler sha256, source sha256, output binary sha256, exact command,
  exit status, and the observed bytes. A program that ignores a negative result is not evidence.

## 10. Wire serialization (only what the pin defines)

- There is no byte-layout type. Encode and decode integers with shifts and masks into `uchar`
  arrays. Little-endian `uint` store (as in `examples/fnv1a32.om`):
  `put b 0 uchar and h 255`, `put b 1 uchar and shr h 8 255`, ... `shr h 24`.
- Little-endian `uint` load: `or or or uint get b 0 shl uint get b 1 8 shl uint get b 2 16 shl uint get b 3 24`.
  Split it over lines if it hits error 12.
- Signed decode without relying on integer `bitcast`: with `u` the loaded `uint`, write
  `v = long u`, then `if ge u 2147483648` / `v = sub v 4294967296` / `end`, then use `int v`
  (`v` is assigned before the block so it survives the block's `end`).
- Formats the pin itself defines: packed lists are a little-endian `u16` count (1..64) then records
  of little-endian `u16` length plus bytes, at most 4096 bytes, no padding or terminator
  ([#L724-L730](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L724-L730)); `pipefds` writes two
  little-endian 32-bit descriptors; `fstat`/`stat` write the 128-byte Linux ARM64 stat structure;
  `waitpid` writes a 4-byte status. Any other protocol layout needs its own spec citation and test
  vectors.

## 11. Compiler errors (exit code = error number, stderr `omc: line N: error K`)

From `seed/omc/gate15.sh` ([#L29-L75](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/gate15.sh#L29-L75)), one minimal program
per code; the message format is `fn fail` ([omc.om#L122-L148](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L122-L148)).

| Code | Meaning | Code | Meaning |
|---|---|---|---|
| 1, 2 | compiler I/O | 13 | non-function name over 8 characters |
| 3 | unknown name or field | 14 | capacity at `91b3c932`: 8192 names, 4096 functions, 512 nested blocks, 1024 nesting depth, 8 MiB input/output |
| 4 | line ends too early | 15 | missing `end` |
| 5 | extra words on the line | 16 | syntax / unknown word |
| 6 | malformed number | 17 | line after `ret`/`break`/`continue`/`trap`/`exit` |
| 7 | type mismatch | 18 | missing `ret` |
| 8 | number out of range | 19 | global passed, or the same array twice in one call |
| 9 | word in a disallowed place | 20 | write to a view or text |
| 10 | duplicate name | 21 | constant index out of range |
| 11 | ambiguous bare number | 22 | bad text literal |
| 12 | register pool exhausted | 23 | size capacity |

A refused compile may still leave bytes on stdout. Never run output from a non-zero compile.
Capacity boundaries are gated in `seed/omc/gate12.sh` (8192 globals accepted, 8193 refused;
4095 helper functions plus `main` accepted, 4096 plus `main` refused).

OM-2 units add or extend codes ([OMEGESE 9.13](https://github.com/aien-dev/omega-mv/blob/19f014f2178ca9cd3473728fe00a475a2398c409/docs/one_layer/OMEGESE.md#L624),
[`spec/omegese-om2.md` section 8](https://github.com/aien-dev/omega-mv/blob/19f014f2178ca9cd3473728fe00a475a2398c409/spec/omegese-om2.md#L225)):
27 initialization (`fresh`), 28 constructor discipline (`seal`, `holds`), 29 external `layout`;
14 also covers expansion fuel (`obligation=growth`) and a 33rd `export`; 24 keeps its OM-1 meaning
(branch displacement). gate15 pins only codes 3..23 (roadmap RM8, #797). Merged contracts
(#807 through #768) add error **30**, diagnostic `obligation=contract` (section 11a).

## 11a. Executable contracts (merged #807 through #768)

Source: OMEGESE section 12 and `seed/omc/contracts/` at main
`91b3c9326919faf1491e29ae9276929e8349b8c7`, compiler `189ada82...3b54` (section 0).
The committed fixtures below were exercised by the batch host qualification:
`evidence/BATCH-189ada82/qualification.log` records all 44 contract cases and the two mutants.
Older compiler `befb8199` does not implement these contextual contract words.

- **Opt-in and policy.** Only OM-2 units. The first non-`note` line selects the build policy:
  `om2` or `om2 debug` checks contracts at run time; `om2 release` erases their evaluation after the
  same syntax, scope, type and purity checks; `om2 diag debug` / `om2 diag release` add diagnostic
  records; `om2 fmt` keeps its canonical output and still checks contracts. No environment variable
  selects the policy. The words are contextual: OM-1 units and the keyword table are unchanged.
- **Where clauses go.** `requires BOOL` and `ensures BOOL` form a function's prefix, before its first
  executable statement (blank and `note` lines may separate them; at most 64). `invariant BOOL` forms
  the prefix of a `while` or `for` body (at most 64 per loop). Anywhere else is error 30.
- **When they run.** Preconditions in order on entry, inline calls included. Postconditions in order
  on every normal return (`ret`, void fallthrough, inline return), never on `exit` or a trap; the
  return value is computed once and `result` (value functions only) names it. A parameter in a
  postcondition is its value at return: there is no `old`. Invariants run at the loop head before
  the condition: on entry even for zero trips, on every backedge, after `continue` (a `for` increments
  first) and before the final false test; `break` skips the next check.
- **What a predicate may use.** Type `bool`; integer and bool literals and names, integer and bool
  operations, checked conversions, grouping, `len`, ordinary checked array, struct and layout reads.
  User calls (inline too), floats, abstract values, host and device words, allocation, writes and
  ownership words are error 30, recursively, in release as well. `and`/`or` stay eager, so they do
  not guard a bounds check.
- **Failure.** False `requires` is trap 9 (exit 109); false `ensures` or `invariant` is trap 10 (exit
  110); function-mode callers get the same codes through `OscRt.trap`. A trap raised while evaluating
  a predicate keeps its own code (the `arithmetictrap` fixture expects 102). Note that 10 is also the
  OM-2 `holds` check trap, so the trap site, not the code, identifies the failing clause.
- **Erasure.** Only a predicate that folds to true without trapping is proved and erased in debug.
  Release erases evaluation by policy, never uses a predicate as an optimizer assumption, and never
  removes the body's own arithmetic, bounds, initialization, stack or loop-fuel checks. The fixtures
  require byte-identical output for `release.om` against `erased.om` and for `proved.om` against
  `plain.om`, not just equal exit statuses.
- **Diagnostics.** With `om2 diag` (debug) or `om2 diag release`, every contract site gets a
  `omc-check 1 kept|erased line=N rule=RULE kind=KIND` record on stderr: rule `runtime` for a kept
  check, `constant-true` for a proved one, `release` for policy erasure; kind `requires`, `ensures` or
  `invariant`. Postconditions are reported at their return sites. Error 30 carries
  `obligation=contract` in the `omc-diag` record. Fixture expectations: `diagproof.om`
  (`requires eq add uint 1 2 3`) gives `omc-check 1 erased line=3 rule=constant-true kind=requires`;
  `diagrelease.om` (`om2 diag release`, `requires false`) gives
  `omc-check 1 erased line=3 rule=release kind=requires`.

`seed/omc/contracts/ok.om` (expected exit 0):

```
om2
fn step x uint to uint
requires lt x 10
ensures eq result add x 1
ret add x 1
end
fn main to int
i = uint 0
while lt i 3
invariant le i 3
i = step i
end
ret int sub i 3
end
```

`seed/omc/contracts/requires.om` (expected exit 109; the precondition fails on entry):

```
om2 debug
fn f x int to int
requires gt x 0
ret x
end
fn main to int
ret f 0
end
```

`seed/omc/contracts/release.om` (expected exit 0, and byte-identical to `erased.om`, which is the
same program without the three contract lines and with `om2` in place of `om2 release`):

```
om2 release
fn f x int to int
requires gt x 0
ensures eq result add x 1
ret x
end
fn main to int
while false
invariant false
end
ret f 0
end
```

Refusals pinned by `contracts.cases`: a user call, host word, float (also through a float constant
or float array), misplaced clause, late `invariant`, `result` in a `requires` or in a void function
are error 30; a non-bool predicate is 7; a name declared later in the body is 3; reading an
unwritten `fresh` element is 27. The cases run through the existing OM-2 conductor
`seed/qualification/om2/gate.om`. `mutant.cases` builds two broken compilers: one with enforcement
disabled (`requires.om` must still exit 109) and one whose prover treats any constant predicate as
proved (`false.om`, `requires false`, must exit 109). Both run to exit 0, so the suite must end
`OM2GATE FAIL cases=2 pass=0 fail=2 invalid=0 not_run=0` (conductor exit 1), which
`seed/host/check.sh` requires with the exact `VERDICT` lines. A passing mutant suite would mean the
check proves nothing.

## 12. Before any claim

1. Inspect the compiler, not memory. Find the rule in `omc.om` or a gate fixture at the pinned commit
   and cite file and line. Skills, older notes and this file can be stale.
2. Compile with the pinned `omc` and run. Report what was observed, labelled host-only.
3. Add a negative control: change an expected value or remove a guard and confirm the exit status
   changes.
4. No silent fallback. If `omc` cannot express something, do not write it in C, Rust, shell or
   Python and call the result Omega. Say exactly which word or capacity is missing.
5. Missing is not impossible. Omega is meant to be extended. State the gap as an extension proposal:
   the hardware or OS need, the proposed word and types, refusal codes, and the gate that would test
   it. Known decision-blocked gaps at `55f9b0af` ([OM_COMPLETION.md#L62-L72](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OM_COMPLETION.md#L62-L72)),
   unchanged at main `bf54b19f` ([#L65-L73](https://github.com/aien-dev/omega-mv/blob/bf54b19fa590e8ae925c0e80815516dbf41e7326/docs/one_layer/OM_COMPLETION.md#L65-L73)):
   packed structs (FR-2), imports (FR-3, units concatenate), threads/atomics (FR-6), C ABI/shared
   objects (FR-7), heap (decision 24). x86-64 (FR-9) is no longer missing: since main `bf54b19f`
   (#923, 2026-10-28) `target x86linux` selects an x86-64 Linux static-ELF backend in
   `seed/omc/omc.om`. Its status there is DRAFT, x86 execution is EMULATED (QEMU) only and native
   x86 qualification is NOT_RUN (`docs/X86-LINUX.md` lines 3 and 286).
6. Documents disagree with the code at this pin; trust the code and name the conflict:
   OM_COMPLETION FR-5 still says mmap/ioctl are absent, and `seed/host/MAPPING.md`
   ([#L66-L68](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/host/MAPPING.md#L66-L68)) still calls the device words a proposal not
   on `main`, while `omc.om` reserves `devctl mapnew ... mapbind` (ids 166-177) and README
   ([#L27](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/README.md#L27)) says they are on `main`. GPU launch and model execution through
   them are not qualified. (README line 5's old "candidate branches" wording was corrected on main by
   #801.) At main `bf54b19f`, `OM_COMPLETION.md` lines 72 and 80 still call FR-9 a draft with its
   full fixture differential NOT_RUN, while `docs/X86-LINUX.md` lines 431-438 record the complete
   EMULATED differential at `7ab5fdd0` (15179 PASS, 4 NOT_RUN, 222 EXCLUDED, no FAIL cases,
   aggregate FAIL). Cite X86-LINUX.md and name the conflict.

## 13. Examples

- `examples/fnv1a32.om`: reads all of stdin with a partial-read loop, hashes it (FNV-1a 32, unsigned
  wrap), writes 4 little-endian bytes with a partial-write loop, distinct exit codes for each failure.
- `examples/trap_narrow.om`: a masked narrowing succeeds, an unmasked one traps 5 (exit 105).

Observed on 2026-10-25, Linux AArch64 host only, compiler sha256 `734f0a39...6b61`:

| Check | Observed |
|---|---|
| `fnv1a32.om` compile | exit 0, empty stderr |
| input `abc`, `a`, empty, 65535 zero bytes | `0be9471a`, `2c290ce4`, `c59d1c81`, `47dac371`; each exit 0 and equal to an independent Python FNV-1a computation |
| 65536 zero bytes | exit 2, no output |
| prime changed to `16777617` (negative control) | `abc` gives `55bedcf5`, differing from the expected hash |
| `trap_narrow.om` | compile exit 0, run exit 105 |
| unmasked line removed | run exit 44 |
| refusal below | compile exit 7, stderr `omc: line 3: error 7` |

Refusal to try inline (expected exit 7, the same class as gate15 `e7_type_mismatch`):

```
global b uchar 1
fn main to int
x = add get b 0 uint 1
ret int x
end
```

- Numeric work in or behind an `.om` unit (limits, sizes, masks, checksums, expected values, tolerances) is computed with Wolfram and quoted with the query, never by hand: see `skills/wolfram-math/SKILL.md`.
