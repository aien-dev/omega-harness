---
name: omega-encoding
description: "Write .om that the seeded Omega compiler actually accepts: syntax, types, traps, bounds, register limits, host I/O, byte encoding, compiler errors, verified examples. Use before writing, reviewing or explaining any Omega program."
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

### Current main pin (checked 2026-10-31)

| Item | Value |
|---|---|
| Repository | `github.com/aien-dev/omega-mv`, branch `main` |
| Commit | `9920aab1413983c2867d02d75b2276341595adb8` (squash of [#994](https://github.com/aien-dev/omega-mv/pull/994), `scrub` keeps live float registers, merged 2026-10-31T10:48:56Z) |
| Compiler source `seed/omc/omc.om` | 16,432 lines, 495 `^fn ` lines, sha256 `d4b4b89d8a58cc90d30d15a525b417e4b31e5bfe5288d21d665f1c7958a330f6` |
| Compiler `seed/out/omc` | sha256 `be8c51301307555254fc59e351d281aae1aa096d91521777e33a41c8b446448a`, 517,856 bytes (equals `seed/out/omc.sha256`; last `seed/omc/size-history.tsv` row `scrub-flive`) |
| Seed loader `seed/rung0/seed0.hex` | sha256 `baaf081fe6055a768485c5d25d7435681ce00b04cfa80b26c48169f78944e14d` |

Merges after `91c83438d` (omc `2aec185d…d8d5`) that change the compiler, oldest first: #967
(`96ea448ad`, IR-mode `TYPE` casts read the operand at its own type, section 4), #948 (`dc0d06e95`,
B/BL imm26 displacements checked by the shared `bdisp`, section 4 error 24 entry), #875
(`236787439`, OM-2 words `dit` 225 / `scrub` 226, section 6), #918 (`2d848d939`, host word
`peername` 224, section 9), #853 (`4722cba58`, floats in V registers, codegen only; see the `scrub`
entry in section 6), #848 (`157bab57c`, `while ... bound N` up to 2^63-1, section 8), #857
(`3a163bc5e`, `for ... bound N`, IR loop ranges, section 8) and #994 (`9920aab14`, `scrub` no longer
zeroes the V16+j twin of a live float slot, section 6; the only merge after `3a163bc5e`). #990, #992
and #989 touch tests and docs only. `seed/rung0`, `seed/rung4/om4.s`, `seed/rung4/om4.bin` (sha256
`d0da40e4…19efab`), `seed/omega-from-seed.sh` and `seed/omc/gate12.sh` are unchanged between
`91c83438d` and `9920aab14`. `omc.om` went from 512 to 495 `^fn ` lines with #853 (#994 adds none),
so the om4 cap (section 8) has 17 free entries at this pin.

Observed 2026-10-31 on a Linux AArch64 host in a clean detached worktree at `9920aab14`:
`sh seed/omega-from-seed.sh` exited 0 (`RC=0`) and left `seed/out/omc` at exactly `be8c5130…448a`
with a clean `git status`. Both guide examples were rerun with that omc and gave the same results
as at the previous pin: `fnv1a32.om` and `trap_narrow.om` compiled with exit 0 to binaries
byte-identical to the ones `24caacf4` emits; `abc`, `a` and empty input gave `0b e9 47 1a`,
`2c 29 0c e4`, `c5 9d 1c 81`, exit 0; 65536 zero bytes gave no output, exit 2; `trap_narrow.om` ran
with exit 105. The same `fnv1a32.om` prefixed with `target x86linux` printed `0b e9 47 1a` for
`abc` with exit 0 under `qemu-x86_64` 8.2.2: EMULATED, not native x86.

The omega MCP tools (`compile`, `run`) use an older pin: omega-mv `e5b32567`, omc `2c824805…029f`.
Their results do not describe main's omc `be8c5130…`; for main, run `git show main:seed/out/omc`
directly.

### Previous pin `3a163bc5` (2026-10-31, history)

| Item | Value |
|---|---|
| Commit | `3a163bc5eafa5be4b9ca97b7f17046c33f0d9032` (squash of [#857](https://github.com/aien-dev/omega-mv/pull/857), `for ... bound N` and the IR loop ranges, 2026-10-31) |
| Compiler source `seed/omc/omc.om` | 16,422 lines, 495 `^fn ` lines, sha256 `0ed35c70437d498cbce3deb661bb24ba4753ec16a8890450a482aaea147bca2a` |
| Compiler `seed/out/omc` | sha256 `24caacf463cc29c5413d5b2565ef8f88c76e60a585bbad44d2388cb1ac29abb7`, 517,768 bytes (equals `seed/out/omc.sha256`; last `seed/omc/size-history.tsv` row `forbound-857-main`) |

This pin had #967, #948, #875, #918, #853, #848 and #857 but not #994, so its `scrub` zeroed live
float locals (section 6). #990, #992 and #989 touched tests and docs only, and the seed files listed
above were unchanged between `91c83438d` and `3a163bc5e`.

Observed 2026-10-31 on a Linux AArch64 host in a clean detached worktree at `3a163bc5e`:
`sh seed/omega-from-seed.sh` exited 0 (`RC=0`) and left `seed/out/omc` at exactly `24caacf4…abb7`
with a clean `git status`. Both guide examples were rerun with that omc: `fnv1a32.om` compiled with
exit 0 and empty stderr to a binary byte-identical to the one `2aec185d` emits; `abc`, `a` and empty
input gave `0b e9 47 1a`, `2c 29 0c e4`, `c5 9d 1c 81`, exit 0; 65536 zero bytes gave no output,
exit 2; the wrong-prime control (`16777617`) gave `55 be dc f5` for `abc`. `trap_narrow.om`
compiled with exit 0 and empty stderr and ran with exit 105; deleting only its unmasked store
(negative control) changed the run exit to 44. The same `fnv1a32.om` prefixed with
`target x86linux` printed `0b e9 47 1a` with exit 0 under `qemu-x86_64` 8.2.2: EMULATED, not
native x86.

### Previous pin `91c83438` (2026-10-31, history)

| Item | Value |
|---|---|
| Commit | `91c83438daec0bdb88bf5c077cae10f3d15d365d` (squash of [#965](https://github.com/aien-dev/omega-mv/pull/965), error-code registry and IR error 30, 2026-10-31) |
| Compiler source `seed/omc/omc.om` | 15,952 lines, 512 `^fn ` lines, sha256 `92eb49f35715147ccb6c6b00a96b9cb38d7f7383de14d4f6521d2e07eaad465c` |
| Compiler `seed/out/omc` | sha256 `2aec185d4ed6d7a6a5779ed656b4e56904742bde5551b02893fb5f73916ad8d5`, 495,668 bytes (last `seed/omc/size-history.tsv` row `error-registry-965`) |

Three merges after `4376c2112` (omc `62775d2a…712b`) changed the compiler: #946 (`d94617177`, host
words `cpufence` 199 / `cpurelax` 200), #970 (`1db941627`, `execpure` word 198 and `close_range`
before `exec`; omc `7a8b84fcc07808a52e6cbb65aa7450c754b3ad62eeddf9ec64cc0edd0938b2de`) and #965
(`91c83438d`: error-code registry and IR error 30, section 11; float literal scans and the untyped
long float literal miscompile, section 4). The `omc.om` line links in this file are permalinks to the
commit they name; on main the line numbers have moved.

Observed 2026-10-31 on a Linux AArch64 host in a fresh clone at `91c83438d`: the committed
`seed/out/omc` hashed to `2aec185d…d8d5`; `seed/rung4/om4.bin` alone compiled that `omc.om` with exit 0
to a 495,668-byte binary with the same sha256; `sh seed/omega-from-seed.sh` exited 0 (`RC=0`) and
left `seed/out/omc` at exactly `2aec185d…d8d5`. Both guide examples were rerun with that omc, with
the same results as with the previous main omc `7a8b84fc…` (both compilers emit byte-identical
`fnv1a32.om` binaries): `fnv1a32.om` compiled with exit 0 and empty stderr; `abc`, `a` and empty
input gave `0b e9 47 1a`, `2c 29 0c e4`, `c5 9d 1c 81`, exit 0; 65536 zero bytes gave no output,
exit 2; the wrong-prime control (`16777617`) gave `55 be dc f5` for `abc`. `trap_narrow.om`
compiled with exit 0 and ran with exit 105; deleting only its unmasked store (negative control)
changed the run exit to 44. The same `fnv1a32.om` prefixed with `target x86linux` compiled with
exit 0 to an x86-64 ELF that printed `0b e9 47 1a` with exit 0 under `qemu-x86_64` 8.2.2:
EMULATED, not native x86.

### x86 integration as tested before merge (observed 2026-10-28)

Then-draft [#923](https://github.com/aien-dev/omega-mv/pull/923), source
`7ab5fdd067884fd3044155084b39163da1be5c8a`, integrated main and batch 3
(#904). Its `seed/omc/omc.om` and `seed/out/omc` hashes equal the main pin `bf54b19fa`
(omc `62775d2a…712b`, main until #946). The seed gate passed, including GATE5. Both guide examples compiled with empty
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

**Error 25 on x86 at main `62d1d0d66` (observed 2026-10-30, compile only; rechecked 2026-10-31 at `4376c2112`).** `docs/X86-LINUX.md`
([#L201-L202](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/docs/X86-LINUX.md#L201-L202))
defines error 25 as an unsupported target/emission path. In `seed/omc/omc.om` at that commit it is
raised by the target-line parser for any `target` word other than `x86linux`
([#L15549-L15550](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/omc/omc.om#L15549-L15550);
`target x86efi` gives `omc: line 1: error 25`), by `emit` when a nondefault target reaches the
AArch64 emitter ([#L571-L574](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/omc/omc.om#L571-L574)),
by `hostval`, `xhost` and `xio` on a target mismatch
([#L6807-L6809](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/omc/omc.om#L6807-L6809),
[#L14663-L14665](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/omc/omc.om#L14663-L14665),
[#L15398-L15400](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/omc/omc.om#L15398-L15400)),
and by the `xcc` condition-code fallthrough ([#L14841](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/omc/omc.om#L14841)).
The device and mapping words 166..177 are **not** refused on x86: `xhost` dispatches them to x86
helpers (`xhdev`, `xhmnew`, `xhmbind`, `xhmdrop`, `xhmcopy`, `xhmmio`;
[#L14679-L14707](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/omc/omc.om#L14679-L14707)).
`seed/host/tests/device.om`, `mapping.om` and `devcall.om`, each prefixed with `target x86linux`,
compiled with exit 0 to x86-64 ELF files with main's omc, at `62d1d0d66` and again at `4376c2112`
(where `target x86efi` still gave `omc: line 1: error 25`, exit 25). They were not run, so no x86
execution of device words is claimed. The VFIO words `vfioctl`/`vfiomap` are not on main (pending
draft [omega-mv#878](https://github.com/aien-dev/omega-mv/pull/878), open on 2026-10-31).
At main `91c83438d` (omc `2aec185d…`, 2026-10-31), at main `3a163bc5e` (omc `24caacf4…`,
2026-10-31) and again at main `9920aab14` (omc `be8c5130…`, 2026-10-31) `target x86efi` still gives
`omc: line 1: error 25`, exit 25 (observed); `vfioctl` is still absent from `omc.om` at `9920aab14`
and #878 is still an open draft. Since #970, `docs/X86-LINUX.md`
also lists `execpure` (word 198) among the `xhost` error-25 refusals on x86 (source text, not run
here).

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

### Rerun at omega-mv `9920aab1` (2026-10-31)

The example gate pins main `9920aab1413983c2867d02d75b2276341595adb8` (#994), `seed/out/omc`
sha256 `be8c51301307555254fc59e351d281aae1aa096d91521777e33a41c8b446448a`. #994 changed only the
`scrub` emitter, and no example uses `scrub`; with only the pin moved and no expectation changed,
`bash examples/gate.sh` against that binary printed
`GATE PASS examples=101 pass=101 fail=0 omc=be8c51301307555254fc59e351d281aae1aa096d91521777e33a41c8b446448a`
(exit 0) on 2026-10-31, Linux AArch64. Negative controls: the gate pointed at the previous omc
`24caacf4…abb7` was refused with exit 2 (`GATE FAIL omc sha256 24caacf4…, pinned be8c5130…
(omega-mv 9920aab1…)`), and on a scratch copy `const N ulong 16` changed to 15 in
`examples/consts.om` ended `GATE FAIL examples=101 pass=100 fail=1` with exit 1 (`stdout differs
from consts.stdout`). The section below is the previous pin.

### Rerun at omega-mv `3a163bc5` (2026-10-31, previous pin)

The example gate then pinned main `3a163bc5eafa5be4b9ca97b7f17046c33f0d9032` (#857), `seed/out/omc`
sha256 `24caacf463cc29c5413d5b2565ef8f88c76e60a585bbad44d2388cb1ac29abb7`. With the previous tree
and only the pin moved, the gate against that binary ended `GATE FAIL examples=99 pass=97 fail=2`:
`mistake_bound_range.om` (`while ... bound 1000000`, valid since #848) and `mistake_for_bound.om`
(`for ... bound 10`, valid since #857) both compiled with exit 0 where error 8 and error 5 were
expected. `mistake_bound_range.om` now uses `bound 9223372036854775808` (error 8, line 4);
`mistake_for_bound.om` became `for_bound.om` (exit 10); `trap4_for_bound.om` (a value count 11 over
`bound 10`, exit 104) and `mistake_ir_bound.om` (`ir`, `while ... bound 1000001`, error 8, line 5)
are new. On 2026-10-31, Linux AArch64, `examples/gate.sh` against `24caacf4…abb7` printed
`GATE PASS examples=101 pass=101 fail=0` (exit 0). Negative controls on a scratch copy each ended
`GATE FAIL examples=101 pass=100 fail=1` with exit 1: `bound 9223372036854775807` in
`mistake_bound_range.om` (`compile exit 0, expected 8`) and `bound 4` in `for_bound.om`
(`exit 104, expected 10`). The gate pointed at the previous omc `2aec185d…d8d5` was refused with
exit 2. The section below is the previous pin.

### Rerun at omega-mv `91c83438` (2026-10-31)

The example gate then pinned main `91c83438daec0bdb88bf5c077cae10f3d15d365d` (#965), `seed/out/omc`
sha256 `2aec185d4ed6d7a6a5779ed656b4e56904742bde5551b02893fb5f73916ad8d5`. On 2026-10-31, Linux
AArch64, `examples/gate.sh` against that binary printed `GATE PASS examples=98 pass=98 fail=0`
(exit 0) with no expectation changed. The rerun also showed that code 25 is emitted (it has been
since the x86 integration `bf54b19f`): `target foo` as the first line is `omc: line 1: error 25`
under both `2aec185d` and the pre-#965 `7a8b84fc`. `examples/err25_target.om` now records it, and
the gate printed `GATE PASS examples=99 pass=99 fail=0` (exit 0); with its first line changed to
`target x86linux` (a known target) that example fails (`compile exit 0, expected 25`). Negative
controls on a scratch copy of the 99-program tree each ended `GATE FAIL examples=99 pass=98 fail=1`
with exit 1: `const N ulong 16` changed to 15 in `examples/consts.om` (stdout `15` instead of
`16`), and one byte appended to `consts.stdout`. The gate pointed at the pre-#965 omc `7a8b84fc…`
was refused with exit 2. The 2026-10-29 section below is the previous pin.

### Rerun at omega-mv `bf54b19f` (2026-10-29)

The example gate now pins this compiler. On 2026-10-29, Linux AArch64, `examples/gate.sh` against
`seed/out/omc` sha256 `62775d2a391cfef7cde64e387655caa9809a267093112910ec76bfa568f3712b` printed
`GATE PASS examples=98 pass=98 fail=0` after the keyword-prefix expectation was updated. The same
gate pinned to `2c824805…029f` had passed 98/98 with the old expectation. A copy of the new gate
pointed at `2c824805` was refused (exit 2).

| Item | Value |
|---|---|
| Commit | `bf54b19fa590e8ae925c0e80815516dbf41e7326` (2026-10-28, merge of PR #923) |
| `seed/omc/omc.om` | 15,624 lines, sha256 `96d51b63c8f451661ac715395078e04f39195e50c9e8fe37746cda121e4d7a1e` |
| `seed/out/omc` | sha256 `62775d2a391cfef7cde64e387655caa9809a267093112910ec76bfa568f3712b` |
| `seed/rung0/seed0.hex` | sha256 `baaf081fe6055a768485c5d25d7435681ce00b04cfa80b26c48169f78944e14d` (unchanged) |
| `docs/one_layer/OMEGESE.md` | blob `90fa5d0c32757da6e7c271e96406b75105ba8008` |

Two defects recorded at `e5b32567` are fixed in this compiler (the #904 integration, `e2662fff`):
`fn continue_x` compiles and the program exits 7 (`examples/defect_keyword_prefix.om`); `bitcast int`
of `uint 4294967295` compares equal to -1 and `add` 2 returns 1, and `bitcast uint` of `int -1`
compares equal to 4294967295. At `2c824805` those three programs were error 16, exit 2 and exit 2,
and `add` 2 trapped 1. `fn continue` is still error 16. The #775 empty-text report was not
reproduced: a two-text unit passing `text e ""` to `view uchar *` returned len 0 on both compilers.
Keep the `part e 0 0` workaround until that report has an exact failing program. Other
"observed at e5b32567" lines below are dated history; the example suite still matches them.

### Historical rerun at omega-mv `e5b32567` (2026-10-27)

Every example and rule in this file that a program can show was rerun on 2026-10-27 with the
compiler below, on a Linux AArch64 host only. The 2026-10-29 section above is current where it
disagrees with this list.

| Item | Value |
|---|---|
| Commit | `e5b32567a7aff3ce594e3254a622a846a6cd190b` (2026-10-26, merge of PR #868) |
| `seed/omc/omc.om` | 12,040 lines, sha256 `7703da00055fc0c7a91fcae43071df76a3f277dbe867cb284a3bdff3bedb7e1b` |
| `seed/out/omc` | sha256 `2c82480596bbdc50cda9626866209ef662d5834f278f839ff4ce0250c495029f` |
| `seed/rung0/seed0.hex` | sha256 `baaf081fe6055a768485c5d25d7435681ce00b04cfa80b26c48169f78944e14d` (unchanged) |
| `docs/one_layer/OMEGESE.md` | blob `119c559557e4f227dee348c0e0e1240c71a42df3` |

The executable record is [`examples/gate.sh`](../../examples/gate.sh): it refuses any other compiler
hash, then compiles and runs 98 programs (`examples/*.om` and this skill's `examples/*.om`) against
exact expected stdout bytes, exit statuses and compiler stderr lines. Observed:
`GATE PASS examples=98 pass=98 fail=0`. Negative controls on a copy each ended `GATE FAIL` with
exit 1: one changed digit in an expected stdout, an expected exit 105 changed to 104, an expected
stderr line number changed, a removed `.expect` file, and a changed program body; a compiler with one
appended byte was refused with exit 2. Arithmetic expectations were checked with Wolfram
(`examples/WOLFRAM.md`).

Changes and confirmations against the rules below, each observed with a program:

- **Hex literals are OM-2 only.** In an OM-1 unit `uint 0x10` is error 6. After an `om2` line it is
  16; `0X10` (uppercase prefix) is error 6, and the value must fit the type: `int 0xffffffff` is
  error 8 (`examples/om2_hex_rotr.om`, `mistake_om2_upper_hex.om`, `mistake_om2_int_hex.om`).
- **`+` is refused.** `y = x + 1` is error 5 and a value starting with `+` is error 3
  (`mistake_infix.om`). OMEGESE 9.9's "`+ - * /`" is still wrong at this pin.
- **Float to integer truncates toward zero and saturates; it never traps.** `long 1.0e30f64` is
  9223372036854775807, `long -1.0e30f64` is -9223372036854775808, `long -5.75f64` is -5,
  `uint -5.5f64` is 0, `uchar 300.0f64` is 255, and a NaN converts to 0 (`float_to_int.om`). Integer
  to integer conversion stays checked (trap 5).
- **The `=` of an assignment is optional** (OMEGESE line 24), so `return x` compiles as an assignment
  to a new name `return`. In a value function the missing `ret` then shows as error 18
  (`mistake_return.om`).
- **`for ... bound N` and the bound range** (since #848 and #857, in the pin since `3a163bc5`): `for i ulong
  n bound 10` compiles (`for_bound.om`, exit 10) and a count past the bound traps 4
  (`trap4_for_bound.om`); a bound is 1..9223372036854775807, so `bound 0` and `bound
  9223372036854775808` are error 8 (`mistake_bound_range.om`), and in IR mode it is 1..1000000
  (`mistake_ir_bound.om`). `trap` takes 1..14 (`trap 15` and `trap 0` are error 8; `trap 14` exits
  114).
- **Names:** an 8-character variable compiles and 9 is error 13; a 63-character function name
  compiles and 64 is error 13. `len` or `stat` as a variable is error 16, `free` is error 9, and
  `bits` is an ordinary name.
- **Placement:** a `struct` line after the first `fn` is error 9; `new` inside a block is error 9; a
  variable `new` length is error 16; `main` with a parameter is error 9. A `const` in function mode
  compiles.
- **Parameter slots, not parameter count.** Parameters fill x0..x5: an OM-2 unsized `view T *` /
  `edit T *` takes two slots (pointer, count), every other parameter one, a sized array included
  (OMEGESE 9.13 "Parameter slots"). Observed: 3 sized views plus 3 scalars compile and run; 2 unsized
  views plus 2 scalars compile; 2 unsized views plus 3 scalars, or 7 scalars, are error 9 on the `fn`
  line (`examples/om2_param_slots.om`, `mistake_om2_param_slots.om`, `mistake_seven_params.om`).
- **Function names of 9..63 bytes are not keywords by their first 8 bytes** (fixed in the #904
  integration, measured 2026-10-29 at omc `62775d2a`). `continue_x` compiles and returns 7
  (`examples/defect_keyword_prefix.om`). At omc `2c824805` the same file was error 16. An exact
  keyword name (`fn continue`) is still error 16.
- **OM-2 slices:** `part A off n` is only an argument to an unsized `view T *` parameter. As a value
  of its own it is error 9, to a sized parameter error 7, of a global error 19; `off + n` past the end
  traps 3 at run time (`om2_views.om`, `om2_part_trap.om`).
- **Errors 1 and 2 reproduce:** a directory as the compiler's stdin gives exit 1,
  `omc: line 0: error 1`; stdout on `/dev/full` gives exit 2 with the line after the last input line
  (`omc: line 5: error 2` for a 4-line unit). A 33rd `export fn` in an OM-2 unit is error 14; exports
  are function mode, so an `export` beside `fn main` is error 9.
- **Error 24 needs more than 1 MiB of code inside one loop.** 40 functions of 4,000 checked `add`
  lines each compiled and ran, because each function's branches stay short. One `while` loop with
  270000 copies of `x = add x 1` was refused with `omc: line 270006: error 24` (the loop's `end`);
  with 140000 copies it compiled. The check is `disp19` (omc.om line 2465 at `e5b32567`, line 2956
  at `91c83438`): a conditional branch reaches at most 2^18 - 1 words forward. The recipe is in
  `writing-omega` section 11. Rerun 2026-10-31 with omc `2c824805` and with the then pin omc
  `2aec185d`: both give exit 24 and that line for 270000 copies, and exit 0 for 140000 copies (the
  program then exits 0). At `e5b32567`, codes 25 and 26 are not emitted by any `fail` call. Since
  #948 (`dc0d06e95`, in the pin since `3a163bc5`) every B.cond/CBZ/CBNZ imm19 and every B/BL imm26 site goes
  through one checked `bdisp` ([omc.om#L2982-L2992](https://github.com/aien-dev/omega-mv/blob/3a163bc5eafa5be4b9ca97b7f17046c33f0d9032/seed/omc/omc.om#L2982-L2992));
  the 8 MiB output ceiling (error 14) keeps every B/BL inside its 128 MiB reach, so imm26 error 24
  is a defence no legal program reaches (`seed/qualification/security673/README.md`). Observed
  2026-10-31 at `3a163bc5e`: that directory's `gate669.om`, built with omc `24caacf4…`, ended
  `GATE669 PASS bounded Linux AArch64 execution and independent decoding only` (exit 0, host only);
  rerun 2026-10-31 at `9920aab14` with omc `be8c5130…`: compile exit 0, the same PASS line, exit 0
  (host only). `bdisp` is unchanged by #994.
- **Narrow `bitcast`:** at omc `2c824805`, `bitcast int` of `uint 4294967295` was not -1 and `add` 2
  trapped 1, and `bitcast uint` of `int -1` was not 4294967295. At omc `62775d2a` those two cases
  are correct (exit 1 and exit 1; `add` 2 returns 1). `ushort`/`short`, `uchar`/`byte` and `f32` to
  `int` were not retested on 2026-10-29. Do not assume they are fixed. Decode signed values
  arithmetically when you have not measured the pair (`examples/wire_le.om`).
- **`byte` is signed 8-bit** (`i8`, OMEGESE line 109); `char` is not a type (error 3).
- **Empty text through `view uchar *` (omega-mv #775).** A 2026-10-27 report said `len` was 60 with
  one other text and 28 with none. A two-text unit on 2026-10-29 returned 0 on both `2c824805` and
  `62775d2a`, so that report was not reproduced. Keep passing `part e 0 0`
  (`examples/empty_text_part.om`) until an exact failing program exists.
- **Skill examples rerun:** `fnv1a32.om` compile exit 0, empty stderr; `abc`, `a`, empty and 65535
  zero bytes gave `0be9471a`, `2c290ce4`, `c59d1c81`, `47dac371`, each exit 0; 65536 zero bytes gave
  exit 2 and no output; the prime changed to `16777617` gave `55bedcf5` for `abc`.
  `trap_narrow.om` compiled with exit 0 and ran with exit 105, and 44 without the unmasked store. The
  section 13 inline refusal gave exit 7, `omc: line 3: error 7`.
- **Section 11a contract fixtures rerun** from `seed/omc/contracts/`: `ok.om` exit 0, `requires.om`
  exit 109, `release.om` exit 0 and byte-identical to `erased.om`; `diagproof.om` stderr
  `omc-check 1 erased line=3 rule=constant-true kind=requires`, `diagrelease.om` stderr
  `omc-check 1 erased line=3 rule=release kind=requires`, each compile exit 0.

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
Main has since moved to the x86 integration compiler `62775d2a...712b` and then to `2aec185d…d8d5` (section 0); cite
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
| Keyword table, the authority for reserved words | `fn kw` in `seed/omc/omc.om` ([#L375](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L375) at `55f9b0af`, [#L765](https://github.com/aien-dev/omega-mv/blob/e5b32567a7aff3ce594e3254a622a846a6cd190b/seed/omc/omc.om#L765) at `e5b32567`, [#L818](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/seed/omc/omc.om#L818) and `fn kw2` [#L1328](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/seed/omc/omc.om#L1328) at `91c83438`); same list in `seed/omc/genkw.awk` ([#L16](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/genkw.awk#L16) at `55f9b0af`; [#L16-L18](https://github.com/aien-dev/omega-mv/blob/e5b32567a7aff3ce594e3254a622a846a6cd190b/seed/omc/genkw.awk#L16-L18) at `e5b32567`, which adds `flock` 197 and a second list for `peercred`, `pollfds`, `sendmsg`; [#L17-L20](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/seed/omc/genkw.awk#L17-L20) at `91c83438`, whose second list adds `qty`, `cpufence`, `cpurelax`), parity checked by `seed/omc/gate18.sh` | gated |
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
- **IR-mode `TYPE value` casts** (merged on omega-mv main as [omega-mv#967](https://github.com/aien-dev/omega-mv/pull/967),
  squash commit `96ea448add639f8448a4787ef36a2ade06a1a244`, 2026-10-31; harness issue #141). In IR mode a
  cast reads its operand at the operand's own type, so it converts an operand of any scalar type; a bare
  number is range-checked against the cast's type (error 8); bool to or from a number is refused (error 9),
  a bool comes only from a comparison. Pinned by `seed/omc/gate-ir.sh`. Observed (host only) with omc
  sha256 `e4d84bc3ff5c91b83ce9a156d262eb28c9563e43374537293129898da093b7a6` built from seed at omega-mv
  main `d39450a23b5936a0aafd35cc23ca70e5468fd4c0`: `ret ulong sub c 48` and `ret long sub c -128`
  compile; `ret uchar 256` and `ret ulong sub c 300` (c uchar) are error 8; `ret bool c` and
  `ret ulong eq a b` are error 9. An earlier pin of this skill (omega-mv `91c83438`, omc `2aec185d…`)
  predates #967: it gives error 7 for the two widening casts and for `ret ulong eq a b`, and compiles
  `ret bool c`. The current pin `9920aab1` includes #967 (since `3a163bc5`): observed 2026-10-31, the
  IR unit `fn dig c uchar to ulong` / `ret ulong sub c 48` compiles with omc `24caacf4…` (124 bytes of
  IR) and is `omc: line 3: error 7` with `2aec185d…`; with omc `be8c5130…` the `ir` unit
  `ir` / `fn dig c uchar to ulong` / `ret ulong sub c 48` / `end` compiles with exit 0 to output
  byte-identical to `24caacf4…`'s (sha256 `40b76639…6cba`).
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
  In an OM-2 unit the limit is 6 parameter *slots*: an unsized array (`view T *`, `edit T *`) takes 2,
  so a 7th slot is error 9 on the `fn` line (observed with omc `befb8199`, 2026-10-26, building
  omega-mv `om/tls`). Pack small fixed inputs into one array. A scalar or a sized array
  (`view uchar 8`) takes one slot: `fn f a view uchar * b view uchar * c ulong d ulong to ulong`
  (6 slots) compiles, adding `e ulong` is `line 2: error 9`, and two `view uchar 8` plus three `ulong`
  (5 slots) compiles in both OM-2 and OM-1. `part B OFF LEN` (OM-2) is an argument form only:
  binding it to a local (`r = part b 0 4`) is error 9; pass `part` directly in the call (exit 4 for a
  4-byte part). In an OM-1 unit the same call is `error 3` (unknown name). First probed on omc `2c82480596bb…`
  (omega-mv `audit/tls-integration` 9472e9615); all rechecked 2026-10-31 with main omc
  `62775d2a…712b` at omega-mv `4376c2112`.
- A unit with `fn main` is a static AArch64 Linux ELF (program mode). Without `fn main` the output is
  bare function words (function mode): it compiles with exit 0 but is **not runnable**. Host words
  and `global` are error 9 there, and `text` is program-mode only ([#L400-L407](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L400-L407), [#L454-L459](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L454-L459), [#L629](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L629)).
- Program exit status = `main`'s value mod 256 (0 for void main). `exit v` (v `int`) ends at once.
  A runtime trap exits **100 + code**. Keep your own exit codes below 100, or a result is
  indistinguishable from a trap ([#L404-L407](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L404-L407), [#L484](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L484)).
- A value-returning function used as a statement is error 7; a bare host word used as a statement
  (`write 1 m 0 3` on its own line) is error 16 (observed 2026-10-26, omc `befb8199...a1d` at omega-mv
  `19f014f2`; rechecked 2026-10-31 with main omc `62775d2a…712b` at omega-mv `4376c2112`: bare
  `write` gives `omc: line 3: error 16`, a bare call of a value-returning `fn` gives error 7, and the
  bound form prints and exits 0). Bind the result either way: `w = write 1 buf 0 n`.
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
float fixtures use `add sub mul div` (`seed/omc/tests/float/arith.om`). Write the words. Observed at
`e5b32567`: `y = x + 1` is error 5 (`examples/mistake_infix.om`).

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
  `0x10`) is error 6 ([#L443-L446](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L443-L446)). In an OM-1 unit: no hex, no octal,
  no character literals: write byte 65, not `'A'` (error 3). OM-2 units (first non-`note` line `om2`)
  also take lowercase hex, `0xdeadbeef`, which must still fit its type (`int 0xffffffff` is error 8,
  `0X10` is error 6; observed at `e5b32567`; full rule in the next item). An `om2` unit with
  `x = uint 0x10` compiled and the program exited 16; the same line without `om2` is error 6 (omc
  `befb8199...a1d`, 2026-10-26; same results 2026-10-31 with main omc `62775d2a…712b` at omega-mv
  `4376c2112`; see also **Hex** in section 8).
  `true` / `false` are the bools.
- Hex literals, OM-2 units only (`om2` first non-note line; `spec/omegese-om2.md` section 2): `0x`
  then 1..16 lowercase digits `0-9a-f`, value unsigned, typed by the place under the decimal range rule
  (`int 0xffffffff` is error 8: no two's-complement reinterpretation). Error 6: `-0x..`, `0X`, no
  digits, 17+ digits, uppercase, `_`, any other character. `0x2a` and `42` compile to identical bytes;
  signed minima need decimal. An OM-1 unit, IR mode included, has no hex literal (error 6). Fixtures:
  `seed/omc/om2/hex/` (gate PR #951, merged by train r3 #979 as `4376c2112`). Rerun 2026-10-31 at
  main `91c83438d` with omc `2aec185d…`: `hex.cases` gives `OM2GATE PASS cases=51 pass=51`, and
  `mutants.cases` gives the expected `OM2GATE FAIL cases=8 pass=3 fail=5` (each `tok_hex` mutant killed).
- Shift with a bare left operand takes the shift amount's type: `shl 1 sh` with `sh` a `uchar` shifts
  a `uchar`. Write `shl ulong 1 sh` when you mean 64 bits ([#L97-L99](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L97-L99)).
- Float literals: `digits.digits[e[+|-]digits][f32|f64]`, a digit on both sides of the dot; exact
  decimal-to-binary rounding; overflow to infinity is error 8 ([#L555-L563](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L555-L563)).
  Two compiler defects in float literals are fixed at main `91c83438d` (#965, omc `2aec185d…`;
  source [`fparse`/`fexp`/`fbits`](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/seed/omc/omc.om#L1908-L1913)
  and [`fspan`](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/seed/omc/omc.om#L2252-L2255)).
  Observed 2026-10-31 on the AArch64 host, with omc `7a8b84fc…` (main before #965) as the control:
  - **Miscompile: float literal of 64 or more characters without an `f32`/`f64` suffix.** Such a
    literal is converted late, once its type is known, from a stored span `pos*64+len`; a length of
    64 or more spilled into the position, so the wrong input text was read (the `fspan` note above).
    `x = f64 2.0` / `y = add <61 zeros>1.5 x` / `ret int y` (a 64-character literal) exits 2 with
    `7a8b84fc` and 3 with `2aec185d`; 70 zeros: 2 and 3; 60 zeros (63 characters) and plain `1.5`:
    3 with both. The type word does not avoid it: `add f64 <70 zeros>1.5 x` and
    `x = f64 <70 zeros>1.5` / `add x f64 2.0` also gave 2 with `7a8b84fc` and 3 with `2aec185d`.
    The suffixed `add <70 zeros>1.5f64 x` gave 3 with both. A binary built by an older omc from
    such a literal computed with the wrong value; recompile it.
  - **Literals longer than about 1000000 characters.** The digit scans ran under the 1000000-pass
    loop guard, so the compiler itself trapped 4 (exit 104, no diagnostic) on such a literal. Now
    they compile: `x = f64 <1000001 zeros>3.5` / `ret int x` exits 3, `1.5e<1000001 zeros>1` gives
    15, `add f64 0.<1000001 zeros>1 f64 7.0` gives 7 (each compile 104 with `7a8b84fc`). The
    mantissa limit below still applies: `7.<1000001 zeros>1` is error 8 with both compilers.
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
width. At omc `2c824805` results across signedness below 64 bits were wrong (section 0). At omc
`62775d2a` the two measured 32-bit pairs are correct; narrower pairs and `f32` to `int` were not
retested. Do not bitcast an unmeasured pair; decode signed values arithmetically (section 10).

Float to integer conversion is the exception to "checked": it truncates toward zero and saturates at
the target's limits, and a NaN gives 0, with no trap (observed at `e5b32567`,
`examples/float_to_int.om`).

## 6. Names, scope, registers

- Function names: up to 63 characters. Every other name (variable, parameter, global, text, const,
  struct, field): **at most 8 characters** on the pinned omc `2c824805`, else error 13 ([#L423](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L423), gate15 `e13_long_variable`).
- PENDING, not merged or pinned: [omega-mv #837](https://github.com/aien-dev/omega-mv/pull/837) (batch 3
  candidate `21d34e12`) raises every such name to 63 bytes, compared in full (`counter_a` and
  `counter_b` are distinct). Error 13 then means over 63. IR-mode locals and parameters, `abstract`/`layout`
  names and layout fields stay at 8, and so does om4. Until the batch-3 pin lands, keep names at 8.
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
- **`dit` and `scrub`: merged in [omega-mv#875](https://github.com/aien-dev/omega-mv/pull/875) as
  `2367874399001efdc7f1ff345fb327469bc1a135` (2026-10-31), committed omc sha256
  `3862655902e3bb9f502ebca2e1e5e5edb6175bf512be731a63c700a6dce8d355`; included in the current pin
  `9920aab1` (omc `be8c5130…`) and in the previous pin `3a163bc5` (omc `24caacf4…`).** Keyword ids: `dit` 225, `scrub` 226 in `fn kw2`, the OM-2 table
  ([omc.om#L1329](https://github.com/aien-dev/omega-mv/blob/2367874399001efdc7f1ff345fb327469bc1a135/seed/omc/omc.om#L1329),
  [#L1384-L1388](https://github.com/aien-dev/omega-mv/blob/2367874399001efdc7f1ff345fb327469bc1a135/seed/omc/omc.om#L1384-L1388)).
  Both are emitted inline, not as helper functions, because om4 holds 512 functions and `omc.om` is at
  that limit ([#L7525-L7526](https://github.com/aien-dev/omega-mv/blob/2367874399001efdc7f1ff345fb327469bc1a135/seed/omc/omc.om#L7525-L7526));
  the merge commit has 512 `^fn ` lines. Two OM-2-only, program-mode-only, AArch64-only value
  words, reserved only in `om2` units. `dit v` (v int 0/1, else trap 3) writes PSTATE.DIT and gives the
  previous bit as a long. `scrub n` (n ulong, a multiple of 16, at most 1048576, else trap 3; trap 6 when
  SP - n is below the stack floor) zeroes `[SP - n, SP)`, the dead general registers (free pool
  registers, X16, X7, X18-X26, X29), V0-V15 and every dead float twin V16+j (since #994; up to
  `3a163bc5` all of V0-V31), and gives 0. Refusals: function mode error 9, a wrong
  operand type error 7, use as a name or as a bare statement (`scrub 16` on its own line) in an `om2`
  unit error 16, `target x86linux` error 25; `ir` with `om2` is already error 16. Observed with omc
  `38626559…` on 2026-10-31 (host only), each fixture of `seed/omc/om2/s5/` at `23678743` run directly:
  `dit-ok` exit 0, `dit-two`/`dit-neg`/`scrub-odd`/`scrub-big` trap 3 (exit 103), `scrub-floor` exit
  106, `scrub-ctl` exit 10 (its negative control), `scrub-mem`/`scrub-live`/`scrub-emit`/`om1-names`
  exit 0, the refusal fixtures 9/7/16 as above, `scrub-emit` output sha256 `5ba83eaf…` as in
  `s5.cases`, and `dit-ok` behind `target x86linux` error 25. The previous pin omc `2aec185d…` does not know
  the words: every fixture that uses them is error 3, while `dit-name`/`scrub-name` (the words as
  names) compile as in OM-1, and `scrub-ctl`/`om1-names` behave as with `38626559…`.
  Rerun 2026-10-31 with the then pinned omc `24caacf4…` (host only): every `s5` fixture gives the same exit
  as with `38626559…` above, `scrub-emit` output sha256 `5ba83eaf…` again, and `dit-ok` behind
  `target x86linux` is `omc: line 6: error 25`; the pin before it, `2aec185d…`, still gives error 3 for
  the fixtures that use the words.
  **Fixed by [omega-mv#994](https://github.com/aien-dev/omega-mv/pull/994) (`9920aab1`): `scrub`
  keeps live floats.** Up to `3a163bc5` (omc `24caacf4…`) the emitter zeroed all of V0-V31 although
  #853 keeps f32/f64 locals and held float values in the twin V16+j of their pool slot. Since #994 it
  emits `MOVI #0` only for V0-V15 and the dead twins, sparing the twin of each live slot whose value
  lives there; nothing is saved or restored
  ([omc.om#L7830-L7842](https://github.com/aien-dev/omega-mv/blob/9920aab1413983c2867d02d75b2276341595adb8/seed/omc/omc.om#L7830-L7842),
  [#L7873-L7884](https://github.com/aien-dev/omega-mv/blob/9920aab1413983c2867d02d75b2276341595adb8/seed/omc/omc.om#L7873-L7884);
  OMEGESE 9.13.1). Observed 2026-10-31, Linux AArch64 host: the unit `om2` / `fn main to int` /
  `f = f64 1.5` / `g = f64 2.5` / `k = scrub 16` / `h = add f g` / `if ne h f64 4.0` / `ret 50` /
  `end` / `ret int k` / `end` compiles with exit 0 under both compilers and exits 50 with omc
  `24caacf4…` (the defect) and 0 with omc `be8c5130…`; with `f64 4.5` in the comparison (negative
  control) the `be8c5130` build exits 50. At `9920aab1`, with omc `be8c5130…` run directly on each
  `seed/omc/om2/s5/` fixture: `dit-ok`/`om1-names`/`scrub-mem`/`scrub-live`/`scrub-emit`/
  `scrub-flive`/`scrub-femit` exit 0, `dit-two`/`dit-neg`/`scrub-odd`/`scrub-big` exit 103,
  `scrub-floor` exit 106, `scrub-ctl` exit 10, `dit-fm`/`scrub-fm` error 9, `dit-type`/`scrub-type`
  error 7, `dit-name`/`scrub-name` error 16, and `dit-ok` behind `target x86linux` is
  `omc: line 6: error 25`. The `scrub-emit` output sha256 is
  `5ba83eaff95e73bdca1584d1d8bcfd083b18af549c7da2777f7a5c03005a04cc` (unchanged: no live float
  there) and the new `scrub-femit` output is
  `3d33c8ebfeff422d2ad746952f32fedab01cc46238097bb64b4b9a0a068c20c8` (with `24caacf4` it is
  `c01cd0c5…7b76`, and `scrub-flive`/`scrub-femit` exit 1). `OM2GATE` on `s5.cases` printed
  `OM2GATE PASS cases=21 pass=21 fail=0 invalid=0 not_run=0`; on `s5-mutants.cases` all 7 `ctl-*`
  cases passed and all 9 `m-*` mutants (including `m-twinall` and `m-twinnone`) were detected
  (`OM2GATE FAIL cases=16 pass=7 fail=9`, the expected verdict).
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
  directly; parameters carry scalars and local arrays only. Two `part`s of one array, or an array and
  a part of it, in one call are also error 19 (omc `befb8199`). The arrays of one call must be distinct
  names (error 19).
- `NAME = new T N v`: local stack array, N elements each v, only in a function's outermost block
  (error 9 inside a block), at most 4 MiB of local arrays (error 23). **N must be a literal or an
  integer `const`**; a variable length is error 16 (gate16 `len_varname`), so there are no
  run-time-sized arrays. No heap and no `free` (decision 24; `free` is error 9).
- Array parameters state type and length exactly: `a view uchar 64` / `a edit uchar 64` (error 7 for
  any other length), so OM-1 has no length-generic array parameters; use fixed sizes, an integer
  `const`, or a global. OM-2 (`om2` first line) adds unsized views, `a view T *`: one function took
  arrays of 5 and 9 bytes and `len a` returned each length (exit 14; omc `befb8199...a1d`, 2026-10-26;
  also `examples/om2-tour.om` at omega-mv `19f014f2`; rechecked 2026-10-31 with main omc
  `62775d2a…712b` at omega-mv `4376c2112`: exit 14, and the OM-1 form is `line 1: error 16`). A view
  or a `text` is read-only (error 20 on `put`, `read` or `pread` into it; see section 9).
- `get A i`, `put A i v`, `len A`. A constant index outside the array is error 21; any other index is
  checked at run time (trap 3). No word yields an address or adds to one.
- `text NAME "bytes"`: read-only `uchar` array; escapes only `\n \t \\ \"` (error 22 otherwise).
  There is no `\r`: protocols that need CR LF (HTTP, SMTP) write the CR at run time
  (omega-mv `om/tls/test/tls/client.om` `cl_crlf`).
- Structs: fields are 8-byte cells, no padding, no packed layouts (a `uint` field still takes 8
  bytes). `read`/`write`/`len` of a struct are error 7. Never treat a struct as a wire layout.

## 8. Loops and bounds

From OMEGESE section 3 and 4 ([#L58-L60](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L58-L60), [#L131-L135](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L131-L135)) and [omc.om#L5493-L5494](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L5493-L5494);
the bound ranges below are those of the pin `3a163bc5`, and the cited lines are unchanged at the current pin `9920aab1` (the `omc.om` ones moved down 10 lines) (OMEGESE [#L63-L69](https://github.com/aien-dev/omega-mv/blob/3a163bc5eafa5be4b9ca97b7f17046c33f0d9032/docs/one_layer/OMEGESE.md#L63-L69),
[#L140-L149](https://github.com/aien-dev/omega-mv/blob/3a163bc5eafa5be4b9ca97b7f17046c33f0d9032/docs/one_layer/OMEGESE.md#L140-L149);
`s_while` [omc.om#L8853-L8854](https://github.com/aien-dev/omega-mv/blob/3a163bc5eafa5be4b9ca97b7f17046c33f0d9032/seed/omc/omc.om#L8853-L8854),
`sf_num` [#L8912-L8915](https://github.com/aien-dev/omega-mv/blob/3a163bc5eafa5be4b9ca97b7f17046c33f0d9032/seed/omc/omc.om#L8912-L8915),
`ir_bound` [#L13162-L13163](https://github.com/aien-dev/omega-mv/blob/3a163bc5eafa5be4b9ca97b7f17046c33f0d9032/seed/omc/omc.om#L13162-L13163)):

- `while COND bound N`: at most N body passes, N 1..9223372036854775807 (2^63-1), a number or an
  integer const; 0 and 9223372036854775808 are error 8 on the loop's line. Without `bound` the limit
  is 1000000 passes. One more pass traps 4. The condition is recomputed before every pass. Merged as
  omega-mv #848 (`157bab57`); before it (pins up to `91c83438`, omc `2aec185d`) the range was
  1..999999 and `bound 1000000` was error 8.
- `for i T K` runs i = 0..K-1. A literal K needs no hidden guard; a name or expression K gets the
  1000000-pass guard. `for i T K bound N` (omega-mv #857, `3a163bc5`) takes the same N as `while`;
  with a bound every count, literal or not, traps 4 after N passes. Before #857 `for` took no `bound`
  word (error 5).
- IR mode (`ir` first line) keeps the C oracle's ranges so that checker2 proves every guard: `bound N`
  on `while` or `for` is 1..1000000, and a number or integer-const `for` count is 0..1000000; outside
  them (a minus sign included) is error 8 on the loop's line. An IR `for` with a value count (a name)
  now carries the hidden 1000000-pass guard; before #857 it was emitted unguarded and checker2
  rejected it (C2-LOOP-UNBOUNDED).
- Observed 2026-10-31, Linux AArch64 host, omc `24caacf4…abb7` (negative control: the previous pin
  omc `2aec185d…d8d5`): `while lt i 1500000 bound 2000000` exits 83 = 1500000 mod 89 (old: error 8);
  `bound 9223372036854775807` and `bound 1000000` compile and run (old: error 8); `bound 0` and
  `bound 9223372036854775808` are `omc: line 3: error 8` on both; `const BIG ulong 5000000` as the
  bound runs 1100000 passes (exit 49 = 1100000 mod 89; old: error 8); `bound 5` with 10 passes wanted
  and 1000001 passes with no bound both exit 104 (trap 4) on both. `for i ulong 1234567 bound 1300000`
  exits 48 = 1234567 mod 89 (old: error 5); `for i ulong 11 bound 10` and the same with `k = ulong 11`
  as the count exit 104 (old: error 5); `for ... bound 0` and `bound 9223372036854775808` are error 8
  (old: error 5); `for i ulong 2000000` without a bound still runs unguarded (exit 81 = 2000000 mod
  89). IR mode: `while ... bound 1000000` compiles (old: error 8), `bound 1000001` is error 8 on both;
  `for i ulong 1000000` compiles, `for i ulong 1000001` is error 8 (old: compiled); `for i ulong 3
  bound 10` compiles and `bound 1000001` is error 8 (old: error 5 for both); `for i ulong k` with
  `k = ulong 3` grows from 162 to 225 bytes of IR (the guard). Wolfram:
  `{Mod[1500000,89], Mod[1100000,89], Mod[1234567,89], Mod[2000000,89]}` -> `{83, 49, 48, 81}`.
- `break` and `continue` (`omc` only, gate7); `continue` in a `while` re-runs the condition and bound.
- Choose N from a proof, not a guess. Example: a read loop where every pass adds at least one byte to
  a 65536-byte buffer needs at most 65536 passes. Write that reasoning in a `note`.
- No loop runs forever and recursion is bounded by the 8 MiB stack. A long-running service needs an
  explicit epoch/restart design, not a bigger bound.

### Compiler limits checked at main `bf54b19fa` (2026-10-29; rechecked at `91c83438d` on 2026-10-31)

Checked with main's `seed/out/omc` (sha256 `62775d2a391cfef7cde64e387655caa9809a267093112910ec76bfa568f3712b`)
and `seed/rung4/om4.bin` (`d0da40e4…19efab`) on the Linux AArch64 host; facts 2, 3, 5 and 6 were also
reproduced with the omega MCP `run` tool (pin `e5b32567`, compiler `2c824805…029f`). Host evidence only.
The first six facts (value slots through x86 target) were rerun on 2026-10-31 with main
`91c83438d`'s omc `2aec185d…d8d5` and with the previous main omc `7a8b84fc…` (#970): the same exit
codes and diagnostics with both. The om4 cap was rechecked as stated in its entry; the 4096 cap on
user programs was not rerun. Float literals longer than about 1000000 characters are a separate,
fixed limit (section 4).

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
- **om4 function cap: 512 functions in `omc.om` itself (contributors to the compiler).** Measured at
  main `57a4135d6` on 2026-10-31: `omc.om` has 495 `^fn ` lines (16,432 lines, sha256 `d4b4b89d…30f6`,
  the same file as at `9920aab14`) and `om4.bin` is still `d0da40e4…19efab`. `omc.om` plus 17 appended
  `fn zqN to ulong` / `ret 0` / `end` functions compiled with exit 0; plus 18 gave
  `om4: line 16484: error 14`, exit 14. So 17 entries are free. **Decision (Drake, 2026-10-31): stay
  within 512 for now (option 3).** A prototype that moves om4's table to 4096 entries is draft
  [omega-mv#1013](https://github.com/aien-dev/omega-mv/pull/1013) (`646ed1ec`, om4 `bfc1cfd1…`, omc
  bytes unchanged), **parked until after alpha**: do not merge it unless `omc.om` needs more than 512
  functions or an alpha-critical fix needs more than the free entries. If it merges, the published
  challenge pin moves in the same release as a new version (v1 = om4 `d0da40e4` at `369dbc96` stays
  checkable), `seed/challenge/run.sh` builds the new pin, and never while an outside recheck
  (DRA-58) is running. Its cubic cycle check is omega-mv#1014. Until it merges, the 512 cap below holds. Earlier
  checks: main `62d1d0d66` (2026-10-30), `4376c2112` and `91c83438d` (2026-10-31), when `omc.om` had
  512 functions and one more gave `om4: line 15953: error 14`. om4 is the seed rung that compiles `seed/omc/omc.om` into the first
  `omc` (`seed/omega-from-seed.sh`
  [#L43-L46](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/omega-from-seed.sh#L43-L46)).
  Its function table is 128-byte entries at `0x1840000`
  ([om4.s#L45](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/rung4/om4.s#L45)),
  and a new function is refused with `E_FULL` (error 14) once 512 are recorded
  ([om4.s#L768-L769](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/rung4/om4.s#L768-L769),
  `cmp.x.i x10 512` / `b.hs E_FULL`; [#L3073-L3075](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/rung4/om4.s#L3073-L3075)).
  512 entries x 128 bytes fill exactly `0x1840000..0x1850000`, where the state slots begin
  (Wolfram `IntegerString[16^^1840000 + 512*128, 16]` -> `1850000`). At `62d1d0d66` main's `omc.om` had exactly 512
  `^fn ` lines (`main` included), so it was full. Observed there (dated): om4 compiled that `omc.om` with exit 0;
  the same file plus one three-line function gave `om4: line 15625: error 14`, exit 14, and the seed
  chain stops there. Any omc change that adds a function must fold or remove another one (#875 inlines
  `dit`/`scrub` for this reason; its `omc.om` note is at L7525-L7526 in the merge commit `23678743`,
  which still has exactly 512 `^fn ` lines). On 2026-10-30
  a clean `git merge-tree` of main with draft
  [#817](https://github.com/aien-dev/omega-mv/pull/817) (head `d0d794d0`) has 532 functions and with
  draft [#832](https://github.com/aien-dev/omega-mv/pull/832) (head `707faca6`) 526, so both are
  blocked until 20 and 14 functions are folded (Wolfram `{532-512, 526-512}` -> `{20, 14}`) or the cap
  is raised. Raising the om4 cap changes a seed rung and is Drake's decision, not a contributor's.
  Check before every compiler PR: `grep -c '^fn ' seed/omc/omc.om` must print at most 512. To
  measure the headroom, append k three-line functions (`fn zqN to ulong`, `ret 0`, `end`, N unique;
  never `fN`, because `f32`/`f64` are type words) and run `seed/rung4/om4.bin < omc.om > /dev/null`.
- **Do not confuse it with omc's cap on user programs.** The compiled `omc` holds 4096 functions per
  program ([omc.om#L116-L118](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/omc/omc.om#L116-L118),
  refusal [#L7809-L7810](https://github.com/aien-dev/omega-mv/blob/62d1d0d668f897386a0e311374f8dbcc5c32dde8/seed/omc/omc.om#L7809-L7810);
  gated by `seed/omc/gate12.sh`, 4095 helpers plus `main` accepted, 4096 plus `main` error 14). The
  "1024 functions" in omega-harness issue #9 described omc on branch `feat/allen-imac-competition`
  (`7eca2ac3f`, 2026-10-25), not om4 and not current main.

## 9. Host I/O: counts, errors, evidence

Base words ([#L476-L486](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L476-L486)): `read fd A off n`,
`write fd A off n`, `pread`/`pwrite fd A off n pos`. Native host words ([#L624-L747](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md#L624-L747)):
files, directories, clocks, sockets, poll, processes, arguments, environment, random.

**cpufence / cpurelax: merged in [omega-mv#946](https://github.com/aien-dev/omega-mv/pull/946) as `d94617177`; in the current pin `9920aab1` / omc `be8c5130` (since `91c83438`).**
Rechecked 2026-10-31 at omega-mv main `91c83438daec0bdb88bf5c077cae10f3d15d365d` (omc sha256
`2aec185d4ed6d7a6a5779ed656b4e56904742bde5551b02893fb5f73916ad8d5`).
These are operand-free statements. `cpufence` (keyword 199) is a full host CPU barrier, AArch64 `DSB SY`
and x86-64 `MFENCE`. `cpurelax` (200) is a spin-wait hint with no ordering, `YIELD` and `PAUSE`.
Unlike every other host word they compile in function mode too. An operand gives error 5, use as a
value or at top level gives error 16, use as a name gives error 5, after `ret` gives error 17, and IR
mode gives error 16. `seed/host/cpufence.sh` proves this: a fixture that exits 42 natively and under
qemu-x86_64, objdump instruction deltas, and a nop-fence mutant that is caught. These are host results
only. An omc older than `d94617177` (its parent's omc is `62775d2a…`) does not know either word: the
fixture fails with error 4 (a bare `cpufence` line read as a name waiting for `=`) and `cpurelax 1` is
error 11, so check the pin before writing them.
**Keyword ids:** #946 landed with ids 199/200. Draft [omega-mv#878](https://github.com/aien-dev/omega-mv/pull/878)
(`vfioctl`/`vfiomap`, with #890 stacked on it) is still open, PENDING, and must take ids that do not
collide; recheck `fn kw` in `seed/omc/omc.om` at the pin before quoting any id.

**pread/pwrite writability (landed via [omega-mv#904](https://github.com/aien-dev/omega-mv/pull/904), `e2662fff`).**
At omega-mv main `bf54b19f` (omc sha256 `62775d2a391cfef7cde64e387655caa9809a267093112910ec76bfa568f3712b`),
`pwrite` only reads its array, so it accepts a `text` or a `view`; `pread` writes its array, so
`pread` into a `text` or an unsized `view` is error 20, as for `read`. Observed 2026-10-30 on the
AArch64 host with main `seed/out/omc` (`omc < file`): `seed/omc/om2/s4/v-bad-pread-text.om` gives
`omc: line 5: error 20` (rc 20); `seed/omc/om2/s4/v-pwrite-text.om` compiles (rc 0) and runs with
exit 0, writing `ABC\n` to stdout. Host evidence only.

Historical note: up to omc `2c824805…` (omega-mv `e5b32567`, the omega MCP pin) the check sat on
the wrong word: `pwrite` from a `text` was error 20 and `pread` into a `text` compiled and returned
`-EFAULT` at run time. Code written for that pin copied a text into a `new` array before `pwrite`.

**`peername`: bounded peer address of an existing descriptor** (merged in
[omega-mv#918](https://github.com/aien-dev/omega-mv/pull/918) as `2d848d939`, committed omc
`38748442…`; in the current pin `9920aab1` / omc `be8c5130`, since `3a163bc5` / omc `24caacf4`). `peername fd A off n` is host word 224,
program mode only. It writes the connected peer's address into `A[off, off+n)` through AArch64
`getpeername` 205 (not `getsockname` 204) or x86-64 `getpeername` 52 (not 51), with the same slice and
private 32-bit length convention as `sockname`. Zero capacity is allowed, at the array end too, and
returns the full length. The result is the full address length (above `n` means truncated) or a
negative errno. An invalid slice traps 3; a read-only buffer (a view or text) is error 20. It creates
and connects no descriptor; the caller owns the descriptor's lifetime. The peer is the transport
endpoint, not an authenticated user; forwarding headers gain no authority from it. Observed
2026-10-31 (host only) with `seed/host/tests/peername.om` at `3a163bc5e`: omc `24caacf4…` compiles it
and the run prints `PASS native peername: accepted peer, truncation, empty slices, errno and bounds`,
exit 0; its mutant arguments 1, 2 and 3 each exit 103; prefixed with `target x86linux` it gives the
same PASS line and exits under `qemu-x86_64` 8.2.2 (EMULATED, not native x86). The earlier pin
`2aec185d…` gives `omc: line 24: error 3` for that file, and compiles `peername = int 1` as a name
(at `24caacf4` that is `omc: line 2: error 16`). Rerun 2026-10-31 at `9920aab14` with omc
`be8c5130…` (host only): the same PASS line with exit 0, mutant arguments 1, 2 and 3 each exit 103,
the `target x86linux` build gives the same PASS line with exit 0 under `qemu-x86_64` 8.2.2 (EMULATED),
and `peername = int 1` as a one-line unit is `omc: line 1: error 16`. This supersedes the pending
candidate notes of harness draft #108 (candidate omc `4ef67f18`/`6a2916cd`).

**`execpure` and descriptor closing in `execprog`** (merged in
[omega-mv#970](https://github.com/aien-dev/omega-mv/pull/970) as `1db941627`; in the current pin
`9920aab1` / omc `be8c5130`; observed below with the then pin `91c83438` / omc `2aec185d`). `execpure A off n` is host word 198
(AArch64 only; `target x86linux` is error 25): it decodes the packet as `execprog` does, pins the
executable as descriptor 3, applies a fixed strict stdio profile (Landlock execute/read beneath that file,
no-new-privs, seven `prlimit64` ceilings, a seccomp filter allowing only `read` on 0, `write` on 1 and 2,
`exit`, `exit_group` and `execveat` on 3) and calls `execveat(AT_EMPTY_PATH)`. It returns only on failure,
with the first failing step's negative errno, and has no weaker fallback. `execprog` also changes: it now
closes every descriptor at or above 3 with `close_range(3, UINT_MAX, CLOSE_RANGE_UNSHARE)` before exec,
so a child no longer inherits them. Observed 2026-10-31 with omc `2aec185d` (host only): the
`seed/qualification/security673/pure-launch.om` fixture launching a static omc program that exits 7 gave
exit 7; launching `/nonexistent/x` printed `PURE_LAUNCH_REFUSED` and exited 2 (ENOENT); a child that reads
descriptor 3 got EPERM (exit 1) under `execpure`, and EBADF (exit 9) under `execprog` with the parent's
descriptor 3 open. The omc of `1db941627^` (`0243e4aff`, sha256 `1578b357…`) refuses `execpure` with
error 3, and its `execprog` child read the inherited descriptor 3 (exit 50). Older pins, including the
omega MCP pin `2c824805…`, lack both.

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
- Measured 2026-10-25 while building a disk image with the seed compiler `seed/out/omc` sha256
  `c99eda2465036c0637a444689a65dd7fc736e4fa8a8b4cf3456b0ce1b57f8996` (omega branch
  `feat/allen-imac-competition` `c3fb79697`, off main `55f9b0af`). Except where noted, these are NOT
  re-measured at the current pin (omc `2c824805…`). The failures that cost the most wasted compiles:
  - `fopen` returns `long`, but `read`, `write`, `fclose`, `fsync`, `fseek`, `ftrunc` and `fchmod`
    take `fd` as `int`: `read int fi A 0 n`. `fseek`'s offset and `ftrunc`'s length are `long`, so a
    `ulong` constant must be converted explicitly (`ftrunc int fo long IMGLN`); a plain `ulong`
    value there is error 7, not a quiet narrowing.
  - There are no implicit conversions in any argument position: mixing a `ulong` value with a
    `uchar` array element (`xor c get A i`) is error 7 until you write `xor c ulong get A i`
    (reproduced at omc `2c824805…`: `d = xor c get A 0` is error 7, `xor c ulong get A 0` compiles).
  - Host words and `global` need program mode: without `fn main` the unit compiles in function mode
    and every host word is error 9 with an empty output file, which reads like a broken pin.

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

The single source of the codes on main `91c83438d` (#965) is the registry
[`seed/omc/error-codes.reg`](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/seed/omc/error-codes.reg)
(issue #797). `seed/omc/gate15.sh` ([#L1-L9](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/seed/omc/gate15.sh#L1-L9),
cases [#L45-L72](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/seed/omc/gate15.sh#L45-L72))
holds one minimal program per code for codes 1..25 and 27..30 (26 has no emitting site), each
checked for its exact exit code and `omc: line L: error C` line, and runs `seed/omc/codereg.om`,
which fails when an emitting site in `omc.om` has no registry row, an emitted row has no site, the
table in OMEGESE section 9.14 ([#L811-L855](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/docs/one_layer/OMEGESE.md#L811-L855))
differs from the registry, or a case did not refuse as listed. irwasm exit codes 20..28 are a
separate namespace in the same registry (OMEGESE section 11.1). The message format is `fn fail`
([omc.om#L122-L148](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L122-L148) at `55f9b0af`;
[omc.om#L292-L322](https://github.com/aien-dev/omega-mv/blob/e5b32567a7aff3ce594e3254a622a846a6cd190b/seed/omc/omc.om#L292-L322) at `e5b32567`;
[omc.om#L324-L356](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/seed/omc/omc.om#L324-L356) at `91c83438`,
`fn fail` on the first line and `exit int code`, the only non-zero `exit` in the compiler, on the last).

| Code | Meaning | Code | Meaning |
|---|---|---|---|
| 1, 2 | compiler input / output (stdin a directory: 1; stdout `/dev/full`: 2; both observed) | 13 | non-function name over 8 characters |
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
| 24 | AArch64 branch displacement out of range | 28 | abstract type: `seal` outside its constructor, or an unsupported `holds` |
| 25 | unknown target, or an AArch64-only path on another target | 29 | external `layout` out of bounds or misplaced |
| 26 | unused (no emitting site) | 30 | contract syntax misplaced or not supported here (IR mode included) |
| 27 | `fresh` element read before it is definitely written, or a fresh array passed on | | |

A refused compile may still leave bytes on stdout. Never run output from a non-zero compile.
Capacity boundaries are gated in `seed/omc/gate12.sh` (8192 globals accepted, 8193 refused;
4095 helper functions plus `main` accepted, 4096 plus `main` refused).

OM-2 units add or extend codes ([OMEGESE 9.13](https://github.com/aien-dev/omega-mv/blob/19f014f2178ca9cd3473728fe00a475a2398c409/docs/one_layer/OMEGESE.md#L624),
[`spec/omegese-om2.md` section 8](https://github.com/aien-dev/omega-mv/blob/19f014f2178ca9cd3473728fe00a475a2398c409/spec/omegese-om2.md#L225)):
27 initialization (`fresh`), 28 constructor discipline (`seal`, `holds`), 29 external `layout`;
14 also covers expansion fuel (`obligation=growth`) and a 33rd `export`; 24 keeps its OM-1 meaning
(branch displacement). Merged contracts (#807 through #768) add error **30**, diagnostic
`obligation=contract` (section 11a).

**IR mode refuses contract clause lines with error 30** (main `91c83438d`, #965, #791; OMEGESE
section 11 [#L1097-L1102](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/docs/one_layer/OMEGESE.md#L1097-L1102)).
Contracts are native only; an IR unit that writes one is refused rather than misread. Only the clause
form is refused: a contract word followed by `=` (`ensures = ulong 1`) still assigns a local of that
name. Observed 2026-10-31 with omc `2aec185d…`, control omc `7a8b84fc…` (main before #965):
`ir` / `fn f x uint to uint` / `requires lt x 10` / `ret x` / `end` is `omc: line 3: error 30`,
exit 30 (control: exit 0, 59 bytes of IR, the clause read as a local named `requires`); the same
with `ensures lt x 10`: error 30 (control: exit 0, 59 bytes); with `invariant lt x 10`: error 30
(control: `omc: line 3: error 13`, a 9-byte name). `ensures = ulong 1` / `ret ensures` compiles with
exit 0 to the same 55 bytes of IR under both compilers. Native units are unaffected: with
`2aec185d`, as with `7a8b84fc`, an OM-1 unit (no `om2` first line) with `requires ge x 0`, called
with -5, still compiles and exits 6 (the contract is not checked), and the same unit with `om2`
first exits 109 (trap 9).

Limits met while porting CUDApp's encoder and cubin writer (aien-dev/CUDApp#143, merged
`2fab72fde0`), rechecked with omega-mv main `bf54b19f` (`seed/out/omc` sha256 `62775d2a...`):
an array parameter of length 16777216 compiles and 16777217 is error 23, so one `.om` buffer holds at
most 16 MiB of `uchar`; `text` inside a function-mode unit is error 9 (both OBSERVED, identical on
omc `2c824805`). AUTHOR-CLAIMED, not rechecked: `global` inside a function-mode unit is error 9, and
omega-bridge `431fff0a` `native c` exits 110 when an entry's array length is written as a `const`
name (write the literal in the entry signature).

One minimal refusal per code is in `examples/errNN_*.om` of this repository, with its exact stderr
line in `errNN_*.expect`, and `examples/gate.sh` runs them: codes 1..23 and 27..30, observed at
`e5b32567`, plus code 25 (`err25_target`, added 2026-10-31 at `91c83438`). Code 24 needs a
generated program of more than 1 MiB inside one loop (recipe in `writing-omega` section 11;
observed at `e5b32567` and rerun at `91c83438`, section 0). 25 and 26 were unused at `e5b32567`;
since `bf54b19f` the target checks emit 25 (section 0, rerun at `91c83438`).

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
  Warning: an OM-1 unit that contains a contract line still compiles, and the contract never runs.
  `fn step x long to long` / `requires ge x 0` / `ret add x 1` called with -5 compiles with no
  diagnostic and returns -4; adding `om2` as the first line makes the same call trap 9 (exit 109),
  and with `om2` a call with 5 passes the contract and returns 6. Observed with omc 734f0a39,
  2c824805, the guide pin 2aec185d (omega-mv `91c83438`) and omc e4d84bc3 (omega-mv main
  `d39450a2`). Every unit that relies on a contract must start with `om2`.
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
5. Missing is not impossible, and it is not a stopping point. Drake's standing rule (2026-10-26):
   "If .om doesn't have a feature, you need to make that feature." Build it in Omega as an omega-mv
   pull request, coordinated with whoever owns the compiler lane at the time. The PR names the
   hardware or OS need, the word and its types, refusal codes and a new error code if one is needed,
   gates with a negative control, parity against an independent oracle, and the matching update to
   this skill. Host code is a last resort only. When it is used, record it with its Omega successor
   and the omega-mv PR that will replace it. Known decision-blocked gaps still need Drake's decision
   before they are built (roadmap RM4 heap, RM6 imports;
   [omega-mv#793](https://github.com/aien-dev/omega-mv/issues/793), [#795](https://github.com/aien-dev/omega-mv/issues/795)).
   They are listed at `55f9b0af` ([OM_COMPLETION.md#L62-L72](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OM_COMPLETION.md#L62-L72)),
   unchanged at main `bf54b19f` ([#L65-L73](https://github.com/aien-dev/omega-mv/blob/bf54b19fa590e8ae925c0e80815516dbf41e7326/docs/one_layer/OM_COMPLETION.md#L65-L73)):
   packed structs (FR-2), imports (FR-3, units concatenate), threads/atomics (FR-6), C ABI/shared
   objects (FR-7), heap (decision 24). x86-64 (FR-9) is no longer missing: since main `bf54b19f`
   (#923, 2026-10-28) `target x86linux` selects an x86-64 Linux static-ELF backend in
   `seed/omc/omc.om`. Its status there is DRAFT, x86 execution is EMULATED (QEMU) only and native
   x86 qualification is NOT_RUN (`docs/X86-LINUX.md` lines 3 and 286).
6. Documents can disagree with the code; trust the code and name the conflict. At main `91c83438d`
   (rechecked 2026-10-31) the earlier conflicts are resolved: OM_COMPLETION FR-5
   ([#L68](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/docs/one_layer/OM_COMPLETION.md#L68))
   lists `devctl devcall uvmctl mapnew ... mapbind` (ids 166..177 in `seed/omc/genkw.awk`) as
   implemented for AArch64 program mode (omega-mv #944, merged by train r5 #985 as `613765d2b`; it
   said "absent" through `4376c2112`), and `seed/host/MAPPING.md`
   ([#L68-L70](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/seed/host/MAPPING.md#L68-L70))
   says the device words are on `main` (train r2 #972, `f5c9bd89a`). GPU launch and model execution
   through them are not qualified. The FR-9 row
   ([#L72](https://github.com/aien-dev/omega-mv/blob/91c83438daec0bdb88bf5c077cae10f3d15d365d/docs/one_layer/OM_COMPLETION.md#L72))
   and `docs/X86-LINUX.md` line 450 now cite the same EMULATED differential at `62d1d0d6` (15182 PASS,
   0 FAIL, 2 NOT_RUN, 222 EXCLUDED, aggregate FAIL); native x86 stays NOT_RUN.

## 13. Examples

- `examples/fnv1a32.om`: reads all of stdin with a partial-read loop, hashes it (FNV-1a 32, unsigned
  wrap), writes 4 little-endian bytes with a partial-write loop, distinct exit codes for each failure.
- `examples/trap_narrow.om`: a masked narrowing succeeds, an unmasked one traps 5 (exit 105).

Both carry `.expect` (and for `fnv1a32`, `.stdin` `abc` and the 4 expected bytes in `.stdout`) and
run in [`examples/gate.sh`](../../examples/gate.sh) with the 97 programs in the repository's
`examples/` directory; section 0 records the `e5b32567` rerun of every row below. The portable guide
for writing a first program is [`skills/writing-omega/SKILL.md`](../writing-omega/SKILL.md).

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
