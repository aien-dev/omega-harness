---
name: omega-harness
description: Operate in Omega and the AIEN repositories. Use for any Omega or .om task, compiler or migration work, AIEN architecture, repository routing, or project status claim. Covers source pins, .om-first implementation, compile and run evidence, safe effects, communication and stopping repeated errors.
---

# Omega Harness: operate, encode, communicate

These skills are operating guidance for coding agents. They are not a compiler and cannot guarantee correct output. What they do is require the checks that catch the mistakes agents keep repeating. English is how the work gets steered. Explicit programs, checked contracts and replayable execution evidence are what establish that something was done.

Companion skills in this package:

- `omega-encoding`: the language the seeded compiler actually accepts, with verified examples.
- `omega-communication`: status labels, reports, handoffs and corrections.
- `aien-repositories`: what AIEN builds and which repository owns a task.

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
- The scalar IR-to-WebAssembly backend is owned by omega-mv, not a foreign-source import:
  [#787](https://github.com/aien-dev/omega-mv/pull/787) merged as
  `3364a1dfd0ca50c1b82bf19ba34932d8bc10a7a9`, after boolean IR fix
  [#786](https://github.com/aien-dev/omega-mv/pull/786) merged as
  `87489da90a1a7a750fd4a7f9eeb005b1221721f6`. It is written in seeded Omega.
  See `omega-encoding` section 0 for tested hashes, exact limits and refusal boundary.
  Source merge status does not establish browser deployment.

### Neuralese supplement: pending, not a second execution language

Keep **Omega**. Neuralese is a **PROPOSED** learned-representation supplement,
not a rename, new compiler mode or latent-reasoning claim. The bounded native
reference in [omega-mv draft #919](https://github.com/aien-dev/omega-mv/pull/919)
is **IMPLEMENTED / EXERCISED (host)**, not QUALIFIED, MERGED or DEPLOYED.
Candidate documentation/evidence pin:
`e79566c191ea8ef11888e69ce50d8fadf375d4fd`; clean implementation actually
exercised: `39e89fb1c2aaed99ddc05ec565274dd05792b70f`, based on main
`6999fa9e6dae4847c155ef4eacaece74f491d3ca`.

- Read `neuralese/README.md` at that pin. Bind producer model/artifact and decoder
  identities, dimension, dtype, canonical byte encoding and checksum explicitly.
  Different models/layers/decoder revisions are not interchangeable. The current
  producer is explicitly a hand-authored fixture, not a trained model.
- Treat every representation as untrusted data. Decode only into explicit Omega
  operations; preserve the envelope and decoded artifact. Feed the output through
  the ordinary compiler, kept runtime contracts and receiver's existing
  effect/capability admission policy. No decoder-created authority or privileged
  execution path is permitted.
- The reference codebook emits only pure scalar operations and uses `om2 debug`.
  Native host words use ambient process permissions, not general capability
  confinement. This patch is not a sandbox for arbitrary source. An effectful
  proposal without a checked receiver executor must be refused.
- First-party implementation and the existing host conductor are `.om`.
  `neuralese/decode.om` composes with `om/tls/sha2.om`; static independent byte
  fixtures do not introduce a host decoder or runtime dependency.

The scoped compiler `seed/out/omc` SHA-256 is
`d6290c7663c2b4bafb59eb3fdf812476e977f4c144def71095c6248ef9b96b84`;
`seed/omc/omc.om` SHA-256 is
`3f937160c823d3a90a6b4cc296f00f517f36ede7fd27a06505c998985ec62240`.
From a fresh Linux AArch64 clone at the implementation pin, with the scratch
directory absent:

```sh
mkdir -p build
if seed/out/omc < seed/qualification/om2/gate.om > build/neuralese-gate; then
  chmod +x build/neuralese-gate &&
  build/neuralese-gate seed/out/omc \
    d6290c7663c2b4bafb59eb3fdf812476e977f4c144def71095c6248ef9b96b84 \
    neuralese/host.cases build/neuralese-host
else
  exit 1
fi
```

Observed compiler/conductor exits: `0` / `0`;
`OM2GATE PASS cases=54 pass=54 fail=0 invalid=0 not_run=0`. Cases include exact
decoded-source and executed-result checks, seventeen guard-removal mutant kills,
a wrong-codebook control, type/purity refusals and runtime contracts. Evidence:
`neuralese/host-results.287fdc263f7af93107c378ba63c92202a0e6c1479cb1ff6971706b423b0b15fb.json`
at the candidate documentation pin. The learned-producer experiment remains
**PROPOSED / NOT_RUN**: same tasks and independent acceptance checks for source
text, structured operations and a pinned compatible learned producer/decoder.
No model, GPU, live service or compiler syntax change was used in this work.

### NL3/NL4 canonical latent boundary (pending source lanes)

The canonical LTP0 schema belongs to
[aien-protocols-mv draft #71](https://github.com/aien-dev/aien-protocols-mv/pull/71),
source `6fb728a14781eb4e5262ffbdc9c2e6be64974f36`, not a second Omega wire format.
Its `specs/latent-payload/validator.om` SHA-256 is
`e85d174839f7d0c3406f783ec0fd69d52ce5576bdcca4892c16b21b09f14b003`.
Compose after `om2 debug`, with the pinned Omega `om/tls/sha2.om`.
The native API is
`lpvalid(frame view uchar *, context view uchar *, now ulong, lastseq ulong) -> int`;
the comma notation here describes the API, not Omega source syntax.
The receiver owns the exact1024-byte context. Status0 means structural and
template conformance only: rank/shape/contiguous strides, precision/quantization,
finite f32 payload, time, seq and receiver challenge, identity/context fields,
reserved bytes and SHA256 integrity. It does not authenticate a sender, allocate
a challenge, consume a durable attempt, verify consent against current authority,
or grant a capability. The source lane's recorded133-case host result belongs
to that exact source/evidence pair; it is not production qualification.

LTP0 field101 is the producer-node effect class, not a receiver grant.
Fields192/224 are existing producer MJR1 DAG/input-binding fields.
Production relation is completed producer MJR1.output_digest ==
SHA256(whole LTP0) == independently admitted consumer CONTENT.digest, with
generation and objective/tenant/subject agreement. The consumer's verifier and
capability admission are independent. Do not invent a self-hashing producer DAG,
an omitted-operand binding, parent/child authority, or producer-supplied trust
booleans. Signed MJB/MJR relation and atomic durable replay admission are NOT_RUN.

[omega-mv #920](https://github.com/aien-dev/omega-mv/issues/920) consumes this
validator and NL1's fixed Hadamard unit, not another implementation:
train source `0e01c58a41f1a2fb976efa75c1cc554503aa2ee4`,
`native/neuralese/hadamard.om` SHA-256
`da4a1b91fcd72b4f5fa3a70461932626acdbc62a9a8d0b9876433f4a44f6ed20`.
Retain its AGPL-3.0-only license in composed executables. The native NL4
`nl4-dependencies.json` locks exact source versions, paths and licenses.
The numerical projection proposes explicit uint operands, output shape and
resource/effect bounds. Independent receiver expectation must match every typed
field before ordinary compiled `om2 debug` operations run.
Only closed pure host-synthetic operations are in scope; all nonzero effect
classes and native-model profiles refuse. No vector bytes become instructions.
`NL4R1` is a HOST_SYNTHETIC stream, not an authorized production receipt.
Never infer learned-channel quality from these deterministic fixed-codebook cases.

The absent bindings are owner proposals, not decoder-created authority:
[AEGIS #79](https://github.com/aien-dev/aegis-runtime-mv/issues/79) binds the
existing gateway/capability admission, and
[protocols #70](https://github.com/aien-dev/aien-protocols-mv/issues/70) binds
the existing atomic durable microjob reservation after `lpvalid`.
Effects remain refused until those existing owner paths are actually exercised.
No new compiler, scheduler, Cortex writer, authority or protocol is needed here.

Compiler hash resolution is source-backed: `d6290c76...` is the committed
`seed/out/omc` introduced by Omega
`1fe5976d398646b17acabbf6d9dfae96b017e439`, matching the full hash above.
Older `2c824805...` (82a399b9) and `49923639...` (ec07d25d) are distinct historical
binaries, not aliases. Do not compile this candidate with a stale mounted pin
and claim the result covers the candidate compiler.

NL4 exercised implementation pin:
`09b1394fe68287eaa10b0d90cf07a9dd00e6cb60`.
Its native unchanged OM2GATE conductor observed
`OM2GATE PASS cases=63 pass=63 fail=0 invalid=0 not_run=0`;
the inherited #919 suite separately observed54/54PASS.
The content-addressed HOST_SYNTHETIC receipt is
`neuralese/nl4-host-results.03b624d6ad393604d958f554fc55844885712ca49f425f98459974846e9e5280.json`.
Read `neuralese/README.md` and `nl4-dependencies.json` at that exact source for
reproduction and source/license staging. Do not concatenate the consumer before
the Hadamard globals: compose entry + Hadamard unit + proposal + validator + SHA2.
No examples in this guide were changed; no compiler or language pin was changed.
The protocol source/evidence head is
`01b5dc3c9f127d988d7c4e5c9ab10ef8dd140d2e`; its133/133 result is host structural
conformance, not MJB/MJR authority or production qualification.


## 3. Compile, execute, challenge

Use the target repository's documented commands. A host-only seed workflow on AArch64 Linux, in a disposable clone pinned to an exact revision, looks like this:

Current candidate source pin (2026-10-28): omega-mv
`1fe5976d398646b17acabbf6d9dfae96b017e439`, pending
[#911](https://github.com/aien-dev/omega-mv/pull/911) and incorporating
[#909](https://github.com/aien-dev/omega-mv/pull/909).
`seed/omc/omc.om`: 12,520 lines, sha256
`3f937160c823d3a90a6b4cc296f00f517f36ede7fd27a06505c998985ec62240`;
`seed/out/omc`: sha256
`d6290c7663c2b4bafb59eb3fdf812476e977f4c144def71095c6248ef9b96b84`.
The fresh-clone `qty`, wrap/refusal controls and `check`/`next` outputs are
recorded in `omega-encoding` section 0. Older pins below remain dated history.


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
- Rebuilding with `sh seed/omega-from-seed.sh` rewrites tracked files, including `seed/out/omc.sha256`. Record the expected digest independently beforehand; the rewritten sidecar is not a trust anchor. A self-rebuild fixpoint proves consistency, not correctness and not diverse compilation. At source `ec07d25d5506d33567adbd4d9b518177b7933bad` on 2026-10-26, the seed gate reproduced `499236398aabc652e2dae27fe4bdc4c5db3d3726314356e0dbef23e3345803ec`. Main later used `befb8199db8c4bca188a494dfaac2a6f0ee5c36fcea11188c75666c55fce8a1d`. Main `91b3c9326919faf1491e29ae9276929e8349b8c7` now carries batch compiler `189ada82e00bb440cb41351bd252cc68c54a1b1050051cc42a1df700081c3b54` (contracts, capacity and f32 returns); its host qualification is `evidence/BATCH-189ada82/README.md`.
- Gate5 bookkeeping for any compiler change: since main `91b3c932`, `seed/omc/gate5.sh` writes current outputs to `build/seed/r5/evidence/`, never to immutable `evidence/SEED-0/omc/`. The living ledger `seed/omc/size-history.tsv` keeps the old history byte for byte, then one tab-separated row per compiler change (`rewrite change source_lines functions words sha256`). At `91b3c932` the last row is `RM2-capacity-f32` (`189ada82…`). Gate5 checks that the last row's sha256 equals the freshly built `omc`, and it never appends a row itself. So a compiler change adds exactly one row by hand, with values measured from the new build, in the same commit as the change. Never edit an older row to make the check pass, and never point the check back at `evidence/SEED-0`.
  At the current candidate pin, `bash tests/gates/run.sh seed` also invokes
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
