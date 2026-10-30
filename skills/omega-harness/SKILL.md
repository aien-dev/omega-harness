---
name: omega-harness
description: Operate in Omega and the AIEN repositories. Use for any Omega or .om task, compiler or migration work, AIEN architecture, repository routing, or project status claim. Covers source pins, .om-first implementation, compile and run evidence, safe effects, communication and stopping repeated errors.
---

# Omega Harness: operate, encode, communicate

Merge status, 2026-10-29: omega-mv #904 and #923 are merged (x86 main
`bf54b19fa590e8ae925c0e80815516dbf41e7326`); omega-bridge #40, including the
earlier #37 proposal, merged at `ed420fcbffa1ad83db78111deb836dd880ed891e`.
The pending labels in the dated observations below describe pre-merge tests.
Compiler identities and execution limits are unchanged. Native x86 remains NOT_RUN.

These skills are operating guidance for coding agents. They are not a compiler and cannot guarantee correct output. What they do is require the checks that catch the mistakes agents keep repeating. English is how the work gets steered. Explicit programs, checked contracts and replayable execution evidence are what establish that something was done.

Companion skills in this package:

- `omega-encoding`: the language the seeded compiler actually accepts, with verified examples.
- `omega-communication`: status labels, reports, handoffs and corrections.
- `aien-repositories`: what AIEN builds and which repository owns a task.

## Development method: Human-Directed Agentic Engineering

Use Human-Directed Agentic Engineering (HDAE) to describe the development workflow. Humans define the goal, architecture, constraints, acceptance criteria and stopping points. Agents implement, execute checks, diagnose failures and revise inside that boundary. Return architectural decisions, acceptance and release decisions to human review; an agent or advisor cannot enlarge its own authority.

This wording follows the method and systems-software case study documented by Gregory M. Kurtzer, *Ginux: Building an Independent Kernel That Runs Linux Userspace through Human-Directed Agentic Engineering* (2026), [doi:10.5281/zenodo.23682548](https://doi.org/10.5281/zenodo.23682548). It does not rename Omega, change compiler pins, rewrite earlier evidence or establish that an unrun gate passed.

## 1. Orient before acting

1. Identify the owning repository with `aien-repositories`. Record its actual remote, default branch and exact source commit, then read its `AGENTS.md` (or `CLAUDE.md`) and the target directory's `.crumb`. Read `crumb-protocol` before editing, creating or moving a directory, or when a gate reports a stale crumb: it covers reading a crumb, claiming and releasing locks, whispers and regenerating committed crumbs. An open PR, a local branch and a deployed service are three different states.
2. Read `omega-encoding` before writing `.om`. Treat skills, memory and model recall as navigation aids, not language authorities. Check their claims against the chosen revision's `docs/one_layer/OMEGESE.md`, `seed/omc/omc.om`, `seed/host/README.md` and real fixtures. Find a path before opening it; never guess one.
3. Work in an isolated checkout pinned to the canonical source. Do not use another agent's dirty working tree, and never checkout, reset or stash in a shared one. Read lock files and the contracts of callers and the build before choosing dependencies.
4. Write a short plan: intended behavior, invariants, affected consumers, allowed effects, the acceptance command and a negative control. For device work, start from the hardware mechanism and its vendor manual. Having a tool installed does not authorize using it.

## Mathematics goes through Wolfram (mandatory)

Every computation, derivation, identity check, unit conversion and numeric constant goes through the
Wolfram tools, not mental arithmetic and not a throwaway script. Read `skills/wolfram-math/SKILL.md`:
it carries the rule, the MCP setup that any agent can perform without a human, the two verification
calls, and the failure handling. A session whose Wolfram tools are not mounted is not math-capable:
say so and stop rather than estimating.

## 2. Implement in the actual language

- First-party implementation belongs in `.om`. Existing C, Rust and shell are unfinished migration, not permission to add more foreign code. Never pass off a foreign implementation as generated code, vendor code, glue or an `.om` file.
- When the language lacks something, write a concrete extension proposal and the smallest experiment that would test it. Do not invent a word, and do not fall back silently to another language.
- Seeded Omegese, compiled by `seed/out/omc`, is the forward language. The historical console syntax in `spec/omega-language-v0.md` and the C compiler checks are different things. As of omega-mv main `55f9b0af47de70a3d06357985cf99ae3e45c016f` (2026-10-25), the seeded compiler and its host and device words are on main. Any source saying they exist only on candidate branches is stale.
- Do not promote design ambitions into supported syntax. Check the keyword table, typing and lowering, and the fixtures of the compiler you will actually run. Compiler limits are implementation limits at a pin, not permanent properties of Omega.
- Optimize for verified programs produced per unit of inference cost on held-out tasks (first-attempt correctness, successful compilation, verified behavior, completion time, and total compute cost). Tokenization is not a language rule: fitting vocabulary to single tokens under a specific tokenizer is an empirical cost measurement, not a binding language constraint.
  The token benchmark entry point is
  `sh bench/inference_cost/driver.sh --run-model --endpoint "$AUTHORIZED_ENDPOINT" --model "$MODEL_ID" --langs om,c,py,rs --arms plain,feedback --attempts 3`.
  It never starts a server. Use only an explicitly authorized endpoint; this guide
  update did not contact a model (real token measurements: NOT_RUN).
  Observed negative control at the section 3 candidate pin:
  `sh bench/inference_cost/driver.sh --run-model --model example --langs om,c,py,rs --arms plain,feedback --attempts 3`
  exited 3 and printed exactly:
  ```text
  NOT_RUN: no --endpoint given; no model was contacted
  ```
- Reuse the existing unit composition, error handling, buffer layouts and build patterns. Make bounds, types, ownership, effects and wire encodings explicit. Keep authorization and refusal paths working, not just the happy-path bytes.
- Migrate the actual consumers before deleting a replaced implementation. Preserve history and immutable evidence. Matching a recorded hash does not prove parity with the current foreign reference; where the contract calls for it, run an independently built oracle.
- Conversion between Omega and other languages is not this skill's job. Route it to `aien-dev/omega-bridge`, which implements checked conversion for a declared subset and refuses universal conversion. An MCP wrapper is optional tooling and does not widen what the bridge can convert. A native binding counts as working for a language only when its caller was built and executed by the bridge's native-call runner (section 3); a binding that was generated but never called is IMPLEMENTED, not EXERCISED.
  Bridge ABI2 rollup [PR #15](https://github.com/aien-dev/omega-bridge/pull/15) merged at
  `826cb8aee4a09b65cc1b2e295b1dc9e50e155253` (2026-10-28), preserving its later main
  native-caller gates. Private-linkage [PR #32](https://github.com/aien-dev/omega-bridge/pull/32)
  merged at `31ce3886d7966c27f8208922e27fc79ed76b39c6`: all private ABI1/ABI2 assembly
  runtime, code and entry labels are local, matching static C helpers and hosted per-TU/TLS
  state. Independently generated translation units link without helper collisions; public
  `omega_`/`omega2_` exports and compiler bytes are unchanged. Its private-symbol regression
  checks independent units, trap recovery, concurrent callers, local `nm` bindings and a
  `.globl ombr_enter` link-failure mutant.
  Freestanding [PR #33](https://github.com/aien-dev/omega-bridge/pull/33) merged at
  `37bf93d158e2e4fd7a3c6a1b9a47b569195a2a66`. It adds generator
  output `freest` and `tools/native --freestanding OUTDIR unit.om...`: generated C/header/object,
  same scalar status ABI, caller-owned stack and no pthread, mmap, heap, TLS or mutable runtime
  state. Whole-source validation includes unused helpers and conservatively refuses arrays,
  globals, text, host effects, OM2 and types outside its declared integer/bool scalar subset.
  Traps return status without writing the result; nested and concurrent calls remain isolated.
  The undefined-import gate permits only the GPU_FS_LINK libc-string allowlist; failed symbol
  extraction/filtering refuses installation. One private input snapshot supplies profile,
  compilation, framing and identity. Tests exercise the pthread-import mutant, failed/missing
  inspectors and a deterministic source swap whose original-reread mutant admits mismatched code.
  Framed source/code must still match the pinned compiler: this is not an arbitrary-bytecode sandbox.
  Current bridge main source pin, including bounded-capacity
  [PR #35](https://github.com/aien-dev/omega-bridge/pull/35) and ABI-v2 verdict
  [PR #34](https://github.com/aien-dev/omega-bridge/pull/34), is
  `0c4cb9ff980af278890b231b994eaede4dc7621c` (2026-10-28; native.om SHA256 unchanged:
  `d3cfe0561696aad92164ddf11340522645cf6daf7af238835c3d3c61ecdef656`).
  Fixed pools hold 4194304 input bytes (accepted input strictly smaller) and 16777216 output
  bytes; exhaustion exits 106 before output. Compiler/per-unit limits are unchanged.
  Historical host evidence with configured language-toolchain PATH: all seven native callers,
  build-failure controls and ABI2 suite passed; private/freestanding controls and a compiler-matched
  16-unit frame beyond both old pool sizes
  passed, while old-bound mutants and both current-overflow probes refused. The independent
  31-root consumer frame emitted and C-compiled with both pinned compilers; those 31 consumer
  functions were not executed by this qualification. Evidence is under
  `evidence/native-{private-symbols,freestanding,capacity}/`. Capacity's
  `coverage-correction.log` preserves and corrects an earlier overclaim; use
  `native-boundaries.log` and `qualification-boundaries.json` for current boundary coverage.
  Remote CodeQL checks are analysis, not functional execution. This is MERGED and HOST EXERCISED,
  not deployed or GPU-qualified.
  A separate ordinary-process-PATH replay at `0c4cb9ff` built and executed C, BTI-C, C++, Python,
  Rust2021 and Rust2024 successfully, but neither Zig nor Go was discoverable on that PATH.
  Both full runners returned exit 1: `NATIVE-CALL NOT_RUN (not run: zig go)` and
  `NATIVE-V2 NOT_RUN (not run: zig go)`. This does not retract the historical configured-PATH
  Zig/Go executions: their logs have individual successful caller lines, not merely aggregate PASS.
  Java, Swift and Node/JavaScript are not callers in these suites. See the
  [coverage clarification and replay receipt hashes](https://github.com/aien-dev/omega-bridge/pull/34#issuecomment-6473079219).
- The scalar IR-to-WebAssembly backend is owned by omega-mv, not a foreign-source import:
  [#787](https://github.com/aien-dev/omega-mv/pull/787) merged as
  `3364a1dfd0ca50c1b82bf19ba34932d8bc10a7a9`, after boolean IR fix
  [#786](https://github.com/aien-dev/omega-mv/pull/786) merged as
  `87489da90a1a7a750fd4a7f9eeb005b1221721f6`. It is written in seeded Omega.
  See `omega-encoding` section 0 for tested hashes, exact limits and refusal boundary.
  Source merge status does not establish browser deployment.

## 3. Compile, execute, challenge

Use the target repository's documented commands. A host-only seed workflow on AArch64 Linux, in a disposable clone pinned to an exact revision, looks like this:

Current main source pin (checked 2026-10-29): omega-mv
`bf54b19fa590e8ae925c0e80815516dbf41e7326` (merge of
[#923](https://github.com/aien-dev/omega-mv/pull/923), after #904 `e2662fff` and #911).
`seed/omc/omc.om`: 15,624 lines, sha256
`96d51b63c8f451661ac715395078e04f39195e50c9e8fe37746cda121e4d7a1e`;
`seed/out/omc`: sha256
`62775d2a391cfef7cde64e387655caa9809a267093112910ec76bfa568f3712b`, equal to
`seed/out/omc.sha256`. Example reruns at this compiler are in `omega-encoding` section 0.
The earlier candidate pin `1fe5976d` (#911, compiler `d6290c76...6b84`) and older pins below
remain dated history.


```sh
git clone https://github.com/aien-dev/omega-mv.git omega-src
cd omega-src || exit 1
git rev-parse HEAD
sha256sum seed/out/omc                        # compare with the pin you expect, recorded beforehand
if seed/out/omc < prog.om > prog 2> prog.err; then
    printf 'compile rc=0\n'
    chmod +x prog || exit 1
    if ./prog; then rc=0; else rc=$?; fi
    printf 'run rc=%s\n' "$rc"
else
    rc=$?
    printf 'compile rc=%s\n' "$rc"
    cat prog.err
    exit "$rc"                               # never execute failed or stale output
fi
```

- The program's return value becomes the process status, modulo 256. A trap exits with 100 plus its code. A nonzero status is not automatically a failure. Tell compiler refusal, runtime trap and intended result apart, and never run output left behind by a failed compile.
- Do not lose the status: no pipelines that hide it, and do not let `set -e` skip collecting an expected nonzero exit.
- Rebuilding with `sh seed/omega-from-seed.sh` rewrites tracked files, including `seed/out/omc.sha256`. Record the expected digest independently beforehand; the rewritten sidecar is not a trust anchor. A self-rebuild fixpoint proves consistency, not correctness and not diverse compilation (diverse double compilation is not on main; omega-mv issue #650 is open). At source `ec07d25d5506d33567adbd4d9b518177b7933bad` on 2026-10-26, the seed gate reproduced `499236398aabc652e2dae27fe4bdc4c5db3d3726314356e0dbef23e3345803ec`. Main later used `befb8199db8c4bca188a494dfaac2a6f0ee5c36fcea11188c75666c55fce8a1d`. Main `91b3c9326919faf1491e29ae9276929e8349b8c7` carried batch compiler `189ada82e00bb440cb41351bd252cc68c54a1b1050051cc42a1df700081c3b54` (contracts, capacity and f32 returns); its host qualification is `evidence/BATCH-189ada82/README.md`.
- Gate5 bookkeeping for any compiler change: since main `91b3c932`, `seed/omc/gate5.sh` writes current outputs to `build/seed/r5/evidence/`, never to immutable `evidence/SEED-0/omc/`. The living ledger `seed/omc/size-history.tsv` keeps the old history byte for byte, then one tab-separated row per compiler change (`rewrite change source_lines functions words sha256`). At `91b3c932` the last row is `RM2-capacity-f32` (`189ada82…`). Gate5 checks that the last row's sha256 equals the freshly built `omc`, and it never appends a row itself. So a compiler change adds exactly one row by hand, with values measured from the new build, in the same commit as the change. Never edit an older row to make the check pass, and never point the check back at `evidence/SEED-0`.
- Merged (2026-10-29): omega-mv#904 merged at `e2662fffbce1c21ef09da71c0d8808133a6af8ab`, carrying the source tested on 2026-10-28 at `7bce2a4fddc43268ac2af09cddc4a5a67d78312d`, compiler `1811d4047f118440cc35e995261af6768b606148cb17e18c42158afd15fcad58` (row `RM-batch3-current`, #896 #897 #898 #903). Main then moved to `62775d2a...712b` (row `FR9-acceptance-fixes`) with #923; at main `bf54b19f` the gate scripts below and `om/units.sh` pin `62775d2a`. The original `397346de` evidence stays historical. A reseed must also move compiler pins in `seed/json/tests/gate-json.sh`, `seed/wasm/gate-irwasm.sh` and `seed/spirv/gate-irspirv.sh`; TLS suites read `OMC_PIN` from `om/units.sh`. The seed/GATE5, current-language and diagnostic checks passed at the tested source. The integration also repairs `s2/mut/cyc.from` and `.to` to preserve the newer `check`/`next` prelude: before repair exact-once substitution matched zero times; after repair the same prohibited-recursion case catches the faulty compiler. An INVALID mutation setup is never a caught fault. Frozen C2 still has its six historical failures, while the separate capacity and batch2 manifests pass; do not call that frozen corpus PASS.
- At main `bf54b19f` (and the earlier candidate pin), `bash tests/gates/run.sh seed` also invokes
  `sh seed/omc/gate5.sh` after the seed build and host checks. Keep one size-history
  row for each distinct compiler change: the qty compiler and the check/next
  compiler have separate rows, even when they arrive in the same PR. A final
  matching row does not excuse omitting an intermediate compiler change.
- Run the changed path and compare exact expected bytes, status and state; a successful compile proves nothing about behavior. Also exercise the failure or refusal boundary most likely to regress. If that is impractical, say exactly what was not run.
- Run a negative control: change an expected value or feed invalid input, and confirm the check fails. A verifier that also accepts its negative control has not established anything.
- Qualification case files (`.cases` in omega-mv) are executed by a conductor, not run directly. More than one conductor exists under `seed/qualification/`, and picking the wrong one looks like a failing gate. Read the conductor's `main` for its argument count first. Observed 2026-10-25 with compiler `734f0a39319fbfa930bcc62cb30dbf06d47d78e126372b74f5baf478e6126b61`: `seed/qualification/om2/gate.om` takes four arguments (`omc pinhash casesfile scratchdir`) and summarises as `OM2GATE PASS cases=N pass=N fail=N invalid=N not_run=N`; `seed/qualification/security673/gate669.om` accepts at most one argument and exits 90 under a four-argument call. A usage status such as 90 from a conductor is INVALID setup, never a qualified refusal.
- Executable contracts reuse that conductor. `seed/omc/contracts/contracts.cases` (44 rows) must end `OM2GATE PASS`. `seed/omc/contracts/mutant.cases` builds two broken compilers, one with enforcement disabled and one whose prover accepts any constant predicate, and must exit 1 with `OM2GATE FAIL cases=2 pass=0 fail=2 invalid=0 not_run=0` plus a `VERDICT … FAIL run exit=0 expected=109` line for each. For that suite, exit 1 with the exact verdict lines is the expected outcome; a pass means the negative control failed. `seed/host/check.sh` runs both.
- Capacity expansion intentionally changes three historical C2 refusals at main `91b3c932`: c2-00304 and c2-00675 (1025 names) and c2-00306 (513 functions). Report frozen `manifest-c2.txt` as 5873/5876 and run the additive `manifest-c2-capacity.txt` (3/3) separately; the pre-capacity compiler must fail that delta. Never edit the frozen manifest or call it a clean pass.
- `seed/wasm/gate-irwasm.sh` and the OM-2 `gate.om` suites are merged but not in any `tests/gates/*.gate` manifest at main `19f014f21`, so `tests/gates/run.sh seed` does not cover them (omega-mv #790). Run and report them separately.
- A benchmark can need a seekable stdin. A checker replay that seeks back over its input 101 times exits 90 when stdin is a pipe, and the same command exits 0 when stdin is an opened regular file. Supply a file, and do not count the setup-status exit as a control result.
- Native callers of compiled Omega units are qualified per language, and only by execution. Since omega-bridge main `adb26cc654656926e76ab1839adae63e92909e88` (PR #18, merged 2026-10-26), `tests/native/run.sh` prints `NATIVE-CALL PASS` only after all seven required callers (`c`, `c-bti`, `cpp`, `python`, `rust`, `zig`, `go`) built and ran. A present toolchain whose build fails prints `NATIVE-CALL FAIL` and a missing toolchain prints `NATIVE-CALL NOT_RUN`; both exit nonzero. Check the refusal paths with `OMEGA_NATIVE_OMC=<pinned-omc> sh tests/native/test_build_failures.sh`: it injects failing Zig and Go toolchains, hides both for the missing-toolchain control, then requires the real Zig and Go callers to pass. It needs real Zig 0.13.0 and Go binaries (`ZIG`, `GO`) and prints `NOT_RUN` without them. The historical `evidence/native-call-v1/run.log` predates this runner and stays unchanged; do not cite it as a current all-caller PASS.
- **x86 native bridge target and compiler refresh (merged 2026-10-29; native x86 NOT_RUN):** omega-bridge [PR #37](https://github.com/aien-dev/omega-bridge/pull/37) (closed unmerged; #40, stacked on it, squash-merged its changes at `ed420fcbffa1ad83db78111deb836dd880ed891e`), commit `1109b46a19c98c650488d36b465d969de65e3d61`, adds explicit `tools/native --target x86linux OUT unit.om...` for hosted SysV AMD64 C/C++/Python/Rust/Zig/Go callers. AArch64 remains the default; all 150 baseline/output comparisons with bridge `0c4cb9ff` passed byte-for-byte. The generator/bootstrap build host is still AArch64; x86 freestanding explicitly refuses. Do not infer the target from the build host.
  Both compiler baselines remain mandatory: `OMEGA_NATIVE_OMC` sha256 `734f0a39319fbfa930bcc62cb30dbf06d47d78e126372b74f5baf478e6126b61` builds the generator (still required at bridge main `ed420fcb`, `docs/NATIVE-ABI.md` lines 15-16). At bridge `1109b46a19c98c650488d36b465d969de65e3d61`, separate `OMEGA_NATIVE_X86_OMC` was pinned to candidate compiler SHA256 `badedbb0c6287887641fcd7e97301b43b5f82f2d7a2838bbe0d17547873108be`, omega-mv source `3e66b6744590cc72c623d8e09413b9072bcab36e`, source SHA256 `4b5392fffd25a3a37844ad8bb5138fbaada690be7dae95132614524bdb78a1e0`, then-pending [compiler draft #923](https://github.com/aien-dev/omega-mv/pull/923) (merged at `bf54b19f`). `OMEGA_NATIVE_OMC2` is unused for x86; do not substitute the candidate for the baseline.
  The tool prefixes only compiler input with `target x86linux`; source snapshots and `src=` identities retain original bytes. X86 export slots are six bytes, not AArch64's four-byte branches. The pinned private ABI uses negative frame slots, R12 for the caller frame anchor, R14 for the runtime table, R15 for the actual stack floor, and ESI/table slot 2 for nonlocal traps. Advanced host helpers refuse bare units; legacy read/write remain permitted. The adapter does not support CET shadow-stack enforcement.
  Run `python3 tests/native/x86/test_build_failures.py [EMPTY_OUTPUT]` with the cross-toolchain, guest Python and `QEMU_X86_64` environment documented in that bridge revision's `docs/NATIVE-ABI.md`. Its strict positive suite executes all six actual language callers (Rust 2021/2024), pointer statuses 32, signal reentry 33, stale identities, private symbols/TLS, traps, fixed-stack-floor and boundary/API mutants. A mismatched candidate must refuse before creating a fresh output directory and preserve existing output bytes. Failed Zig/Go and missing-toolchain controls must refuse aggregate PASS (#34). `tests/native/x86/prepare.py` records exact guest-Python package custody.
  **Historical EXERCISED, EMULATED only:** at bridge `81c7ec3d95469b8ae60b3cf6797068decc1900de`, with its old candidate pin `1f0fdef06574ffec782c164941114edcf51f36f9f9cd1e79bc0452dbcebb3322`, the aggregate command exited 0 under unpatched QEMU 10.0.14, source archive sha256 `c7230968906fdb5540d803e2af1f515e77590a414f2a4dbd3ec9ada3c47083d0`. Those command/runtime/tool/source receipts remain immutable in `evidence/native-x86/`; they do not qualify the refreshed pin. QEMU 8.2.2 produced signal and `/proc/self/maps` failures; the same callers passed with the newer oracle, without weakening them. Physical native x86 and GPU execution are **NOT_RUN**. These were unmerged drafts when recorded; the later merges are not deployment, Alpha or hardware qualification.
  **Historical EXERCISED, EMULATED only (2026-10-28):** at bridge `29087763accea55666ac7a4c14ecb37cef1c2be8` with candidate `f7752bc5c800e85c2393a08153786d0a6bc4516c9cea955fe27624228d6d1409`, the aggregate command exited 0, including both mismatched-pin controls; receipts remain in `evidence/native-x86-f7752bc5/`. The compiler's own full differential at `efd15c234afc584e1e67492e736cbb42806c2a7d` is FAIL (14,930 PASS, 60 FAIL, 10 NOT_RUN, 216 EXCLUDED). Never transplant those totals onto the repaired source.
  **Historical EXERCISED, EMULATED only (2026-10-28):** at bridge `1109b46a19c98c650488d36b465d969de65e3d61` with candidate `badedbb0…08be`, the aggregate command exited 0 (294 PASS lines, 0 FAIL), including both mismatched-pin controls; the previous candidate `f7752bc5` refused with exit 1 before creating OUTDIR. Receipts: `evidence/native-x86-badedbb0/`. The compiler's full differential at `3e66b674` was FAIL (14,936 PASS, 54 FAIL, 10 NOT_RUN, 216 EXCLUDED; omega-mv#923 `d5b445b7`). Those totals do not describe the newer candidate.
  **Merged refresh, EMULATED only (exercised 2026-10-28, merged 2026-10-29):** bridge [#40](https://github.com/aien-dev/omega-bridge/pull/40), merged at `ed420fcbffa1ad83db78111deb836dd880ed891e`, exercised source `8a9618fa2f4e623f332973fd12ab5c466e8356f7`, pins `OMEGA_NATIVE_X86_OMC` to `62775d2a391cfef7cde64e387655caa9809a267093112910ec76bfa568f3712b` (the omega-mv main compiler at `bf54b19f`), omega-mv source `7ab5fdd067884fd3044155084b39163da1be5c8a`. The generator baseline remains `734f0a39…`. Both the full six-language suite and `tests/native/x86/test_build_failures.py` exited 0, including failed/missing-toolchain controls and the real-toolchain positive case. Receipt: `evidence/native-x86-62775d2a.json`. The compiler's complete differential is separate; bridge success does not qualify it. Physical native x86 and GPU execution remain **NOT_RUN**.
  For current compiler acceptance, preserve frozen manifests and replay their pinned compilers. Use only the explicit accepted capacity/batch2 deltas for current C2 rows. Target diagnostic expectations are exact, source-hash-pinned records in `seed/omc/x86-diagnostic-contracts.json`, including required stack checks. Historical failed S2 mutation experiments remain recorded as FAIL; successfully replaying one does not turn it into a successful kill. Never transplant old failure totals or erase their transcripts.
  Full run at `7ab5fdd0`: 15179 PASS, 4 NOT_RUN, 222 EXCLUDED, no FAIL cases; aggregate FAIL because of NOT_RUN. All three original large-input SHA cases passed. The isolated text-write driver addition at `00ebdee8` passed both formerly unrun text-write cases plus exit/encoding controls (5 PASS). The remaining two cases are invalid-fd ioctl checks affected by QEMU's ENOTTY versus native EBADF. The compiler binary is identical across these runs. Receipt: omega-mv `evidence/x86_current_acceptance.1db7902db393b0e4e56e93af8f136133b05663cac58e712c219d84c43fc120b1.json`. This is not native x86 qualification.
- ABI-v2's `tests/native/v2/run.sh` has the corresponding terminal refusal since bridge
  `0c4cb9ff980af278890b231b994eaede4dc7621c` (#34, 2026-10-28): actual failures take precedence,
  otherwise any missing required toolchain emits `NATIVE-V2 NOT_RUN` and exits 1.
  `python3 tests/native/v2/test_verdict.py` passed all five compiler-free tests on that pin;
  these execute the real terminal shell suffix, not upstream detection or native callers.
  The separate full native-v2 run above exercised detection and the new missing-toolchain verdict.
- Compiler changes need the repository's referee and wrong-word-mutant checks, not just a self-fixpoint. Before running any gate, read its prerequisites, device requirements and whether it writes receipts.
- For the merged IR/wasm backend above, run `nice -n 19 sh seed/wasm/gate-irwasm.sh`.
  Node v24.21.0 is a read-only wasm validation/execution oracle only; never use Node to generate
  IR or wasm. The production path is `.om -> omc ir -> irwasm.om -> wasm`, with no Node runtime
  dependency. Observed at that source: `PASS parity cases=129 units=13 traps=29; every export exercised`,
  byte-identical repeat translations, add-to-sub mutant rejected by the parity checker (exit 1),
  and named parse/type refusals with empty stdout. Retained logs/artifacts live in
  `evidence/IRWASM-20261026/`. That backend gate did not exercise a browser or GPU/Alpha qualification.
  Compiler sha256 is `befb8199db8c4bca188a494dfaac2a6f0ee5c36fcea11188c75666c55fce8a1d`;
  its om4-built and self-built fixpoint agrees. Re-exercised at tested head
  `0e81d5a6f990000f1dca50f05f0c6050f7cdf9a5` before #787 merged: seed and om-units
  gates passed and the independently pre-recorded compiler digest matched the rebuild.
  The older pin/fixpoint above remains dated history.
- The site integration merged through [aienos.com #109](https://github.com/aien-dev/aienos.com/pull/109),
  merge `a6aec5e2628804abd27101ba184d7a319f95774e`; #108 closed as superseded.
  `sh om/hive-page/gate-fresh.sh` exercises the native/wasm boundary table; the site check regenerates
  wasm without Node and the existing x86_64 CI test only hashes the artifact. Earlier #108 Chromium
  synthetic-feed fixtures exercised four wasm states and corrupt-module fallback to `js`; those
  dated receipts remain on #108, not in the #109 main tree. The helper source and wasm artifact
  were carried over byte-identically. Merge status does not establish deployment or trusted clocks.
- Record the source revision, the compiler, source and binary hashes, the exact command, the platform, expected versus observed result, and the limits. A local smoke run is not repository qualification, not a merge and not a deployment.
- Pinned `aien-test` verifier sources must be built from a trusted directory outside the candidate tree (`RUNNER_TEMP` in GitHub Actions), with an explicit separate target directory. Cargo loads configuration from its current directory and ancestors even with `--manifest-path`; source pins alone do not exclude a candidate `.cargo/config.toml`. Cache dependency registries/git only, never the compiled verifier target from a candidate run. Challenge both PR and Alpha commands with a candidate rustc-wrapper and a trusted-source negative control. This workflow correction is merged in [sovereign-core #771](https://github.com/aien-dev/aien-sovereign-core-mv/pull/771), merge `2369d7c8`; that source status does not assert deployment of local verifier services.

### SPIR-V OpenCL Kernel backend

`seed/spirv/irspirv.om` in [omega-mv #804](https://github.com/aien-dev/omega-mv/pull/804)
is a handwritten Omega backend for the same OSC IR v1 scalar subset as irwasm.
Source exercised on 2026-10-26:
`4ad014ed3342e202e4fdb1d31b38b8e6f2e0a117`, compiler
`befb8199db8c4bca188a494dfaac2a6f0ee5c36fcea11188c75666c55fce8a1d`.
Merged as `3a10bd29fb7debdcf2c8f5cb58680cc74cb5b1f3`; compiler pin unchanged.

- Read `docs/SPIRV.md` before using the ABI. The output is SPIR-V 1.0, Kernel
  execution model, Physical64/OpenCL memory model, not Vulkan Shader SPIR-V.
  Each original function is a kernel with two global ulong pointers: input
  scalar slots and output `[canonical_result_bits, native_status]`.
  Launch one work item; success status is zero, traps retain `100 + trap`.
- Run `nice -n 19 ionice -c3 sh seed/spirv/gate-irspirv.sh` with
  `SPIRV_VAL` and `POCL_LIBRARY` (absolute trusted PoCL shared-library path).
  The library selection is compiled into the C host, which overwrites inherited
  vendor settings and disables layer discovery before any OpenCL call. It does
  not inspect and reopen an environment-selected ICD manifest.
  The C host is the explicitly requested verification/demo interface to OpenCL,
  never a translator dependency. LLVM is inside the PoCL oracle, not the Omega
  emission path. Validate before executing: the external reader can abort on
  malformed IL, and a nonzero host exit must remain an oracle failure.
- Source-bound CPU evidence: 129 original native/PoCL exact matches, including
  29 traps; 8 supplemental nested-call cases; 3 demo cases; reordered CFG
  regression; 16 byte-identical repeat translations; 17 spirv-val acceptances
  including the valid arithmetic mutant; 16 named refusals with empty stdout.
  The add-to-sub mutant completes successfully but the parity checker exits 1
  for exactly one mismatch. API/setup faults use exit 2 and cannot satisfy it.
  Full logs, per-case tables, raw artifacts and toolchain ledger are retained in
  `evidence/IRSPIRV/20261026-4ad014ed3/`. Host controls also prove inherited
  discovery is ignored and missing/unconfigured libraries fail closed. A separate
  file access trace of the demo host shows no GPU device or system OpenCL
  vendor/layer directory accessed; its public copy redacts home paths.
- Exercised oracle versions: PoCL 7.2 at
  `888c9774b94590fff1afef003154ca2c0ea66605`, LLVM/clang 18.1.3,
  SPIRV-LLVM-Translator 18.1.8, SPIRV-Tools v2025.1, ocl-icd 2.3.2.
  The gate uses `spirv-val --target-env opencl2.1`. Ubuntu PoCL 5.0 lacked IL;
  a user-space SPIR-V-enabled PoCL build was required.
- Non-v1 IR, floats, arrays/memory, vectors, atomics, barriers and host/device
  words are explicit refusals. Contract strings remain metadata, not runtime
  contract checks. No DPC++, Level Zero, GPU or Alpha qualification is implied.
  The standalone acceptance gate is not wired into protected gate manifests.

### CUDApp portable backend (PENDING: draft CUDApp#138, not merged)

- Target status lives in one place: `backend/targets.om` (`targets doctor`, printed by
  `make doctor OMC=/abs/omc`). Only the GB10 (sm_121) is SUPPORTED. Every other GPU row (other NVIDIA
  generations, AMD gfx9-12, Intel Xe, Apple, Qualcomm, Arm, Imagination, Broadcom, Vortex) is PLANNED,
  hardware NOT_RUN. PoCL and lavapipe rows are HOST-ONLY CPUs, never GPU evidence.
- `targets load FORMAT FILE [ENTRY|gfx]` is the `ob_code_load` admission check (SPIRV_CL, SPIRV_VK,
  AMDGPU_HSACO, SM121_RAW; MSL_SOURCE/AARCH64 refused as PLANNED, CUBIN reserved).
- FP32 SPIR-V comes from `backend/spirv/emit.om` (hand-built words; omc IR mode has no floats until E3,
  omega-mv#883), unlike the scalar-ulong irspirv above. Its CPU launcher is migration-pending C until
  E2 (checked shared-library calls, omega-mv#888).
- The gfx1151 HSACO in that branch is an LLVM ORACLE fixture for the loader, not an AMD encoder.
- Gate: `make test-multitarget OMC=…` with negative controls; receipts
  `evidence/MULTITARGET-20261027T0638Z/` (host EXERCISED, 63 checks, CUDApp `0ec44766`). Never say a
  vendor is "supported" or "runs" from these host runs.

## 4. Effects and authority

- A proposed program or effect description does not authorize running it. Confirm capability and privilege at the execution boundary. Model confidence and semantic identity grant no authority.
- Do not weaken a gate, add a new trust path, publish secrets or fabricate hardware receipts. Never edit a historical receipt to make a new claim pass.
- Keep private machine targets, addresses, usernames and home paths in ignored local configuration. Keep intentional public attribution intact.
- Hardware and live services need the operator's current authorization procedure. Host and emulator checks never qualify hardware. Machine-specific operator rules belong in that machine's local instructions, not in public guidance.
- Escalate to an independent reviewer for hard-to-reverse changes to authority, protocols or gates, for contradictory evidence, or when two distinct fixes have failed. A model review is not an execution receipt.

## 5. Communicate and learn

Follow `omega-communication`. Keep proposed, implemented, exercised, qualified, merged and deployed separate. A receiver needs the minimum sufficient definitions, inputs, constraints, evidence and open uncertainty, not the whole conversation. Never drop a prerequisite just to make a handoff shorter.

When an error repeats:

1. Stop repeating the failed pattern.
2. Reduce the problem to a runnable counterexample.
3. Work out whether the syntax, a compiler limit, a protocol, an authority or the expectation itself was wrong.
4. Check the authoritative source.
5. Make one targeted correction.
6. Rerun the counterexample and the negative control.

Record only the verified causal lesson, with its pin and the condition that would invalidate it. Never turn an untested workaround into an unconditional rule.

## 6. Keep the guides current (part of every task)

Whichever agent makes a change also updates the guides. Do not leave it for someone else. A task that changes something these skills describe is not done until the matching skill matches the new reality.

Before you finish, check whether your change touched any of these, and update the skill that covers it:

| You changed | Update |
|---|---|
| Omega syntax, keywords, types, traps, limits, error codes, host or device words, the compiler or its pin | `omega-encoding`, plus the pins in `omega-harness` |
| Build, compile, gate or verification commands, or the seed procedure | `omega-harness` sections 3 and 4 |
| Conversion, encoder or bridge behavior, or what it supports | the conversion note in `omega-harness` section 2, and the README |
| Status vocabulary, receipts, report or handoff conventions, wire formats | `omega-communication` |
| Repository purpose, ownership, names, archive state, plan authority, merge or Alpha policy, a resolved or new contradiction | `aien-repositories` |
| Any aien-dev repository's purpose, status or links, or a new repository | that repository's entry in [aien-dev/sovereign-intelligence](https://github.com/aien-dev/sovereign-intelligence) `index/curated.yml` (layer, evidence, one-line purpose), in the same piece of work |
| These skills, their install method or their layout | the README and the affected skill |

How to do it:

1. Edit the guide in the same piece of work as the change. Give the new source commit and date, and remove any rule the change made untrue; do not just add a contradicting one.
2. If an example or expected output changed, rerun it against the new pin and record what you observed. Never copy a predicted result into the guide.
3. Open a pull request on `aien-dev/omega-harness` with the evidence, and link it to the change that caused it. If that change is not merged yet, open the guide PR as a draft and say it is pending until the change lands.
4. If you are not allowed to change the guide, or you are unsure, open an issue on `aien-dev/omega-harness` describing exactly what is now stale, and say so in your report. Silence is not an option.
5. Your final report must say one of two things: which guides you updated, or that no guide was affected and why.

## Sources

- Shared public rules: https://www.aienos.com/agents/RULES.md
- Repository contract: https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/AGENTS.md
- Seed construction: https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omega-from-seed.sh
- Language: https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OMEGESE.md
- OM-2 contract (normative, on main since #801): https://github.com/aien-dev/omega-mv/blob/19f014f2178ca9cd3473728fe00a475a2398c409/spec/omegese-om2.md
- Omega 1.0 roadmap and its acceptance gates (a plan, not a status claim; tracking issues #790..#800): https://github.com/aien-dev/omega-mv/blob/19f014f2178ca9cd3473728fe00a475a2398c409/docs/ROADMAP-1.0.md

When the compiler changes, refresh the pins and the examples. Older measured evidence stays as dated history: correct the guidance rather than rewriting receipts.
