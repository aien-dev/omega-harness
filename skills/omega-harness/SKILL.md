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
- `writing-omega`: the portable guide for writing a first correct `.om` program, backed by the tested
  programs in `examples/` and `examples/gate.sh`.
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
- When the language lacks something, build it (Drake's standing rule, 2026-10-26: "If .om doesn't have a feature, you need to make that feature"). Open an omega-mv pull request, coordinated with whoever owns the compiler lane, that carries the word or facility, its error code, gates with a negative control, parity against an independent oracle, and the guide update. Do not invent a word you have not implemented, and do not route around the gap with host code. Host code is a last resort only, recorded with its Omega successor and the omega-mv PR that will replace it. A gap that needs a decision Drake has not made (for example heap or imports, omega-mv #793 and #795) is put to him rather than built around.
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
- Route by the ownership rule (Drake's forwarded omega-bridge assessment, 2026-10-27; ADR 0042, Accepted by Drake 2026-10-30, merged as [aien-architecture-mv#342](https://github.com/aien-dev/aien-architecture-mv/pull/342) `6b924d4f8e`): **Omega owns computation, CUDApp owns GPU execution, Omega Bridge owns compatibility.** Language, compiler, compiler pin, semantics and kernel contracts go to omega-mv. Encoders, GPU code generation, cubin writing, disassembly, the driver client, submission and the GPU runtime go to `aien-dev/CUDApp`, even where omega-mv still carries a copy (CUDApp#106, #107, #108, #10, all open). Conversion, bindings, framework attachments and GPU artifact negotiation go to omega-bridge, which never encodes, launches or ships a compiler. A new copy of another repository's owned code needs an ADR amendment first.
- Existing code stays where it is (Drake, 2026-10-27: "keep the repos we've got, but then we make new repos that are connected to the bigger repos"). Do not move, split or rewrite code already in omega-mv, CUDApp, omega-bridge or aienos.com; the copies above stay until Drake decides otherwise. A new standalone server, kit, bench, library or app gets its own aien-dev repository (check `gh repo list aien-dev` for near-duplicates first) that pins its parent by tag or sha, names it in `.crumb`/`.goals.json` (`scaffold-project`, `crumb-protocol`), and is linked both ways (parent README "Connected repos" line). One tool, one job: stdin or files in, plain text or JSONL on stdout, diagnostics on stderr, meaningful exit codes. omega-mv takes only language, compiler and gate changes; aienos.com only the page that consumes a component.
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
- MERGED: the Omega Universal Translation adapters (omega-mv #624) from [#755](https://github.com/aien-dev/omega-mv/pull/755) (final head `b8957b878f`) are on omega-mv main via train r2 [#972](https://github.com/aien-dev/omega-mv/pull/972), squash `f5c9bd89ae604a65d7c777a5739c176f41a036ec`. They are seeded-Omega importers and exporters for profile S1 only (`spec/universal/translation-v1.md`): one pure function with 0..6 `u64` inputs, built from constants and `+ - *` modulo 2^64, in C17, Rust, Python, C++17, Zig and Go, plus `retarget` and `lower` (lowering through omc `734f0a39...6b61`, the `T624_OMC_SHA256` pin in `mk/translation.mk`, to a native unit bound by omega-bridge `tools/native`). Each tool reads one unit on stdin and writes stdout only on success. A refusal names itself on stderr and exits with the spec's status (64 bad command or name, 65 malformed envelope, 67 LIMIT, 68 PROFILE_MISMATCH). The tools are not reentrant, so run one process per call. Coverage is 12 directions and 30 directed compositions. MEASURED 2026-10-31 on the host (AArch64, cores 5-9 nice 19) at main `91c83438daec0bdb88bf5c077cae10f3d15d365d`, with `OMEGA_NATIVE_OMC` sha256 `734f0a39...6b61`, `OMEGA_BRIDGE_REPO`=`SA_SEED_REPO` an omega-bridge clone holding `0fc7d866`, Go 1.27.3 and Zig 0.17.0: the G3 chain run stage by stage (`make -f mk/translation.mk test-translation-g2`, then `test-source-aware-v2`, `test-retarget`, `test-cpp-u64`, `test-zig-u64`, `test-go-u64` and `make -f mk/translation-validate.mk test-translation-compose-v2`) all exited 0; compose-v2 reported 30 directed compositions plus 6 same-language round trips, `pass 636 fail 0 not_run 0`. Controls: without `ZIG` the Zig stage refuses (exit 2), and Go 1.25.14 against the `go >= 1.27` module fails the Go stage (exit 2) rather than passing. The earlier independent #623 validation passed at head `2d4c6407`. Anything outside S1 is refused, not approximated. The next profile, S2 (signed i64, comparisons, select), is specified in `spec/universal/translation-s2-v1.md` ([#822](https://github.com/aien-dev/omega-mv/pull/822), merged `56d979c4`) with status PROPOSED and no implementing tool on main; its C import, Rust export and lowering are on open [#839](https://github.com/aien-dev/omega-mv/pull/839) (host-tested, not qualified, not merged). The bridge statement above still holds for the bridge itself.
- The scalar IR-to-WebAssembly backend is owned by omega-mv, not a foreign-source import:
  [#787](https://github.com/aien-dev/omega-mv/pull/787) merged as
  `3364a1dfd0ca50c1b82bf19ba34932d8bc10a7a9`, after boolean IR fix
  [#786](https://github.com/aien-dev/omega-mv/pull/786) merged as
  `87489da90a1a7a750fd4a7f9eeb005b1221721f6`. It is written in seeded Omega.
  See `omega-encoding` section 0 for tested hashes, exact limits and refusal boundary.
  Source merge status does not establish browser deployment.
- No outside dependency is adopted silently (Drake, 2026-10-25). When the work needs one, code it in seeded Omega and route any conversion through `aien-dev/omega-bridge` — which, per `docs/CAPABILITIES.md` and `docs/NATIVE-ABI.md`, emits checked C and Python from a bounded `.om` subset and lets foreign callers call compiled Omega units. It does **not** import arbitrary C or Python (`Python/C input conversion is not implemented`), so an existing host implementation cannot be pulled in; a missing converter is a concrete extension proposal, not an assumption. A host binary, library or language runtime that remains in the build or production path gets its exact version, what it does for us, and the Omega replacement recorded next to the artifact. Verification oracles are exempt: the pinned compiler, QEMU/OVMF and read-only checkers (mtools, sgdisk, fsck.vfat, git, gh) stay in use.

### Neuralese supplement: merged reference, not a second execution language

Keep **Omega**. Neuralese is a **PROPOSED** learned-representation supplement,
not a rename, new compiler mode or latent-reasoning claim. The bounded native
reference from omega-mv #919 is **MERGED** and **IMPLEMENTED / EXERCISED (host)**,
not QUALIFIED or DEPLOYED: train r1 [#971](https://github.com/aien-dev/omega-mv/pull/971)
merged it as `f56a372aec4909f0601bb4f74ce07b066dd54cf2`
([#919 close comment](https://github.com/aien-dev/omega-mv/pull/919#issuecomment-6529992046)).
Current pin: omega-mv main `9920aab1413983c2867d02d75b2276341595adb8`. The original
evidence pins were documentation `e79566c191ea8ef11888e69ce50d8fadf375d4fd` and clean
implementation `39e89fb1c2aaed99ddc05ec565274dd05792b70f` on main `6999fa9e6`.

- Read `neuralese/README.md` at the current pin. Bind producer model/artifact and decoder
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

The original run used compiler `seed/out/omc` SHA-256
`d6290c7663c2b4bafb59eb3fdf812476e977f4c144def71095c6248ef9b96b84` (receipt
`neuralese/host-results.287fdc263f7af93107c378ba63c92202a0e6c1479cb1ff6971706b423b0b15fb.json`,
kept unedited on main). The conductor refuses any compiler other than the pinned one,
so at main `9920aab1` use the current omc SHA-256. From a fresh Linux AArch64 clone,
with the scratch directory absent:

```sh
mkdir -p build
if seed/out/omc < seed/qualification/om2/gate.om > build/neuralese-gate; then
  chmod +x build/neuralese-gate &&
  build/neuralese-gate seed/out/omc \
    be8c51301307555254fc59e351d281aae1aa096d91521777e33a41c8b446448a \
    neuralese/host.cases build/neuralese-host
else
  exit 1
fi
```

MEASURED 2026-10-31 at main `9920aab1` (omc `be8c5130...448a`, clean detached worktree):
`OM2GATE PASS cases=54 pass=54 fail=0 invalid=0 not_run=0` (exit 0), the same count as at
`3a163bc5` (omc `24caacf4...abb7`), at `91c83438` (omc `2aec185d...d8d5`) and in the original
`d6290c76` receipt; passing the previous pin hash `24caacf4` to the same conductor gives
`OM2GATE INVALID cases=54 pass=0 fail=0 invalid=0 not_run=54` (exit 3). Earlier negative control (2026-10-31 at `91c83438`): one byte appended to
`neuralese/fixtures/positive-1-7.om` gives `decode-positive-1-7 FAIL ... stdout differs
at byte 128` and `OM2GATE FAIL cases=54 pass=53 fail=1`. Cases include exact
decoded-source and executed-result checks, seventeen guard-removal mutant kills,
a wrong-codebook control, type/purity refusals and runtime contracts. The learned-producer experiment remains
**PROPOSED / NOT_RUN**: same tasks and independent acceptance checks for source
text, structured operations and a pinned compatible learned producer/decoder.
No model, GPU, live service or compiler syntax change was used in this work.

### NL3/NL4 canonical latent boundary (source lanes merged)

The canonical LTP0 schema belongs to
[aien-protocols-mv #71](https://github.com/aien-dev/aien-protocols-mv/pull/71),
reviewed source `6fb728a14781eb4e5262ffbdc9c2e6be64974f36`, merged as squash
`703eb1b2fdf72cc03ef9851957d70054957e01fe`; not a second Omega wire format.
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
`09b1394fe68287eaa10b0d90cf07a9dd00e6cb60`, merged to omega-mv main by train
[#971](https://github.com/aien-dev/omega-mv/pull/971) squash
`f56a372aec4909f0601bb4f74ce07b066dd54cf2` (omega-mv #922).
Its native unchanged OM2GATE conductor observed
`OM2GATE PASS cases=63 pass=63 fail=0 invalid=0 not_run=0`;
the inherited #919 suite separately observed 54/54 PASS.
Rerun 2026-10-31 at omega-mv main `91c83438daec0bdb88bf5c077cae10f3d15d365d`,
omc sha256 `2aec185d4ed6d7a6a5779ed656b4e56904742bde5551b02893fb5f73916ad8d5`,
validator taken from protocols merge `703eb1b2` (byte-identical, same SHA-256):
`OM2GATE PASS cases=63 pass=63` (exit 0); one byte appended to the
`f32-double0` expected fixture gives `OM2GATE FAIL cases=63 pass=62 fail=1` (exit 1).
The content-addressed HOST_SYNTHETIC receipt is
`neuralese/nl4-host-results.03b624d6ad393604d958f554fc55844885712ca49f425f98459974846e9e5280.json`.
Read `neuralese/README.md` and `nl4-dependencies.json` at that exact source for
reproduction and source/license staging. Do not concatenate the consumer before
the Hadamard globals: compose entry + Hadamard unit + proposal + validator + SHA2.
No examples in this guide were changed; no compiler or language pin was changed.
The protocol source/evidence head was
`01b5dc3c9f127d988d7c4e5c9ab10ef8dd140d2e`; at merge `703eb1b2` its
`specs/latent-payload/evidence/host-final.log` still records
`OM2GATE PASS cases=133 pass=133`. That is host structural conformance, not
MJB/MJR authority or production qualification. The independent NL6 replay is
[benchmarks #56](https://github.com/aien-dev/benchmarks-mv/pull/56), merged as
`b1cefe485099f8c7092055274b1844edcb02726f`.


## 3. Compile, execute, challenge

Use the target repository's documented commands. A host-only seed workflow on AArch64 Linux, in a disposable clone pinned to an exact revision, looks like this:

Current main source pin (checked 2026-10-31): omega-mv
`9920aab1413983c2867d02d75b2276341595adb8` (squash of
[#994](https://github.com/aien-dev/omega-mv/pull/994), `scrub` keeps live float registers, after
#857, #967, #948, #875, #918, #853 and #848; the list is in `omega-encoding` section 0).
`seed/omc/omc.om`: 16,432 lines, sha256
`d4b4b89d8a58cc90d30d15a525b417e4b31e5bfe5288d21d665f1c7958a330f6`;
`seed/out/omc`: sha256
`be8c51301307555254fc59e351d281aae1aa096d91521777e33a41c8b446448a`, 517,856 bytes, equal to
`seed/out/omc.sha256`, and reproduced by `sh seed/omega-from-seed.sh` in a clean detached
worktree (exit 0). Example reruns at this compiler are in `omega-encoding` section 0. The earlier
main pins `3a163bc5e` (#857, omc `24caacf4…abb7`), `91c83438d` (#965, omc `2aec185d…d8d5`),
`bf54b19fa` (#923, omc `62775d2a…712b`, unchanged through `4376c2112`), `1db941627` (#970, omc
`7a8b84fc…`), the candidate pin `1fe5976d` (#911, compiler `d6290c76...6b84`) and older pins below
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
- Pin sites to move together when `seed/out/omc` changes (checked at main `91c83438daec0bdb88bf5c077cae10f3d15d365d`, 2026-10-31; every site there holds `2aec185d…d8d5`; list them with `git grep -l <old digest> -- ':!evidence'` and read each hit): `seed/out/omc` and `seed/out/omc.sha256:1`; one new `seed/omc/size-history.tsv` row; `om/units.sh:21` `OMC_PIN=` (enforced at :30 with `NOT_RUN: omc is not the pinned compiler`, exit 3); the files `om/units.sh` regenerates and `--check` compares (`src/omega_om_units.c:196`, `src/omega_native.h:18`, `src/omega_om_units.ident:1-3`, per its header :7-17); the parity runners' `OMC_PIN=` lines, each refusing with exit 3 on a mismatch: `om/evidence/parity/run.sh:17` (added by #761, checked at :22), `om/aarch64/parity/run.sh:15` (:20), `om/codec/parity/run.sh:12` (:17), `om/gpumath/parity/run.sh:13` (:18), `om/receipt/parity/run.sh:14` (:19); the IR gates: `seed/json/tests/gate-json.sh:9` `PIN=${JSON_OMC_PIN:-…}` (checked at :17), `seed/equiv/gate-irequiv.sh:9` `PIN=` (:10), `seed/nki/gate-irnki.sh:22` `pin=` (:24), `seed/spirv/gate-irspirv.sh:24` `pin=` (:26), `seed/stablehlo/gate-irstablehlo.sh:22` `pin=` (:23), `seed/tensix/gate-tensix.sh:33` `OMC_PIN=` (:86), `seed/wasm/gate-irwasm.sh:11` `pin=` (:12); `native/orig4/orig4.pin:14` `omc` (checked by `tests/orig4/run.sh:67`); plus prose mentions such as `om/evidence/README.md:63`, `om/receipt/README.md:64`, `om/quantity/README.md:45`, `om/codec/README.md`, `om/gpumath/README.md` and `neuralese/README.md`. At `91c83438d`, 25 tracked files outside `evidence/` name `2aec185d`; `CONTRIBUTORS.md` and earlier `size-history.tsv` rows keep `7a8b84fc` and `62775d2a` as history. Between `62d1d0d66` (2026-10-30, 11 files outside `evidence/` named the digest) and `4376c2112` (42 files), the train merges #971 (`f56a372ae`) and #972 (`f5c9bd89a`) added the aarch64, codec, gpumath and receipt parity runners, the equiv, nki, stablehlo and tensix gates and `orig4.pin`. Precedents: #965's merge `91c83438d` and #923's merge `bf54b19fa` moved all of these that existed then, and `82a399b9f` ("build(omc): batch 2 pin 2c824805") moved the `om/units.sh`, `src/` and `seed/out` subset. A draft that still adds `seed/spirv/gate-vkspirv.sh:29` `pin=` (omega-mv#876 head `8a1126f2`, open on 2026-10-31) adds one more site when it lands. Produce the new digest only with `sh seed/omega-from-seed.sh`, which runs om4 then stage1..stage3 and refuses unless stage2 equals stage3 (`seed/omega-from-seed.sh:43-60`, `PASS omc fixpoint`); a self-fixpoint is still consistency, not correctness. A PR branched before #761 (for example omega-mv#918, base `6999fa9e6`) or before #971 lacks some of these sites, and its stale pins elsewhere must be caught when it is brought up to date.
- At main `bf54b19f` (and the earlier candidate pin), `bash tests/gates/run.sh seed` also invokes
  `sh seed/omc/gate5.sh` after the seed build and host checks. Keep one size-history
  row for each distinct compiler change: the qty compiler and the check/next
  compiler have separate rows, even when they arrive in the same PR. A final
  matching row does not excuse omitting an intermediate compiler change.
- **Seed-chain bootstrap challenge (merged in [omega-mv #863](https://github.com/aien-dev/omega-mv/pull/863), merge `52eba8831aa0b8db9fa2f2e13289920be4fa85f6`).** The public command that rebuilds omc from the 408-byte seed without rewriting tracked files is `git clone https://github.com/aien-dev/omega-mv && cd omega-mv && sh seed/challenge/run.sh`. The script extracts `git archive 369dbc96` (seed/rung0..4, seed/omc) into `build/seed/challenge/src`, then builds seed0 → hex1 → asm2 → om4 → omc stage1/2/3, where each rung is built by the freshly built rung below it. It checks every input and rung against `seed/challenge/expected.txt`, whose values were read from the committed files with `git show <pin>:<path> | sha256sum` (omc `2c824805…`, 325,524 bytes). Exit codes: 0 `CHALLENGE PASS`, 1 FAIL, 2 usage, 3 NOT_RUN (not AArch64 Linux, or pin missing). In a `git clone --depth 1` the pin is missing; first run `git fetch --depth 1 origin 369dbc96daa1684db2d7c2c7085bbf16b3fd77ee` (omega-mv `seed/challenge/CHALLENGE.md`, commit `d764f076`). `--tree DIR` and `--expect FILE` run negative controls. Observed 2026-10-27: a pass on the Spark from a fresh clone of merged main `52eba883` (challenge 0.53 s wall, chain 362 ms), and a pass on GitHub `ubuntu-24.04-arm` (chain 359 ms) together with three controls that fail as required (tampered hex1, wrong omc value, forged log). The public runner and result inbox are in [aien-dev/omega-seed-challenge](https://github.com/aien-dev/omega-seed-challenge): `challenge.yml`, the "I reproduced it" issue form, and `results/*.log` checked by `check-result.sh`. A pass is a fixed-output reproducibility match. It is not general self-hosting and not diverse double compilation, and a log check proves consistency, not that the run happened.
- Run the changed path and compare exact expected bytes, status and state; a successful compile proves nothing about behavior. Also exercise the failure or refusal boundary most likely to regress. If that is impractical, say exactly what was not run.
- Run a negative control: change an expected value or feed invalid input, and confirm the check fails. A verifier that also accepts its negative control has not established anything.
- Qualification case files (`.cases` in omega-mv) are executed by a conductor, not run directly. More than one conductor exists under `seed/qualification/`, and picking the wrong one looks like a failing gate. Read the conductor's `main` for its argument count first. Observed 2026-10-25 with compiler `734f0a39319fbfa930bcc62cb30dbf06d47d78e126372b74f5baf478e6126b61`: `seed/qualification/om2/gate.om` takes four arguments (`omc pinhash casesfile scratchdir`) and summarises as `OM2GATE PASS cases=N pass=N fail=N invalid=N not_run=N`; `seed/qualification/security673/gate669.om` accepts at most one argument and exits 90 under a four-argument call. A usage status such as 90 from a conductor is INVALID setup, never a qualified refusal.
- Executable contracts reuse that conductor. `seed/omc/contracts/contracts.cases` (44 rows) must end `OM2GATE PASS`. `seed/omc/contracts/mutant.cases` builds two broken compilers, one with enforcement disabled and one whose prover accepts any constant predicate, and must exit 1 with `OM2GATE FAIL cases=2 pass=0 fail=2 invalid=0 not_run=0` plus a `VERDICT … FAIL run exit=0 expected=109` line for each. For that suite, exit 1 with the exact verdict lines is the expected outcome; a pass means the negative control failed. `seed/host/check.sh` runs both.
- Capacity expansion intentionally changes three historical C2 refusals at main `91b3c932`: c2-00304 and c2-00675 (1025 names) and c2-00306 (513 functions). Report frozen `manifest-c2.txt` as 5873/5876 and run the additive `manifest-c2-capacity.txt` (3/3) separately; the pre-capacity compiler must fail that delta. Never edit the frozen manifest or call it a clean pass.
- `seed/wasm/gate-irwasm.sh` and the OM-2 `gate.om` suites are merged but not in any `tests/gates/*.gate` manifest at main `19f014f21`, so `tests/gates/run.sh seed` does not cover them (omega-mv #790). Run and report them separately.
- A benchmark can need a seekable stdin. A checker replay that seeks back over its input 101 times exits 90 when stdin is a pipe, and the same command exits 0 when stdin is an opened regular file. Supply a file, and do not count the setup-status exit as a control result.
- Native callers of compiled Omega units are qualified per language, and only by execution. Since omega-bridge main `adb26cc654656926e76ab1839adae63e92909e88` (PR #18, merged 2026-10-26), `tests/native/run.sh` prints `NATIVE-CALL PASS` only after all seven required callers (`c`, `c-bti`, `cpp`, `python`, `rust`, `zig`, `go`) built and ran. A present toolchain whose build fails prints `NATIVE-CALL FAIL` and a missing toolchain prints `NATIVE-CALL NOT_RUN`; both exit nonzero. Check the refusal paths with `OMEGA_NATIVE_OMC=<pinned-omc> sh tests/native/test_build_failures.sh`: it injects failing Zig and Go toolchains, hides both for the missing-toolchain control, then requires the real Zig and Go callers to pass. It needs real Zig 0.13.0 and Go binaries (`ZIG`, `GO`) and prints `NOT_RUN` without them. The historical `evidence/native-call-v1/run.log` predates this runner and stays unchanged; do not cite it as a current all-caller PASS.
- Since omega-bridge PR #26 (merged 2026-10-30 as `f637021f36f0cfff39240a62341a47e442ae05c3`) the native call generator also binds `edit`/`view` arrays of `f32` (4-byte, 4-aligned) and `ushort` (2-byte, 2-aligned, e.g. binary16 bits converted inside Omega), besides `ulong`/`uchar`; other array kinds (e.g. `view uint`) are still refused with exit 110 and the previous output left untouched. Check them with `OMEGA_NATIVE_OMC=<omc 734f0a39…> sh tests/native/kinds.sh` (C caller vs `_Float16`-computed values, six status-32 refusals, an `add -> sub` mutant that must fail; `NATIVE-KINDS PASS` rerun 2026-10-31 at `f637021f`, at bridge main `654305fa` and at bridge main `9532044e`; any other compiler prints `FAIL omc sha256`). The omc pinned by `tools/native` (`734f0a39…`, still the pin at bridge main `9532044e`) rejects functions that return `f32` (error 5); omega-mv omc `2aec185d…` (pin `91c83438`) accepts them (`seed/omc/tests/float/f32_return.om` compiles and exits 0), but bridge units must compile under the pin, so have helpers return `uint` bits and `bitcast` in the caller. First consumer: [aien-dev/ggml-omega](https://github.com/aien-dev/ggml-omega), a llama.cpp backend whose op kernels are Omega units; its per-op `test-backend-ops` table and end-to-end decode are its evidence, not the bridge's.
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
- om/tls coverage commands (merged: omega-mv #862 `audit/tls-coverage`, landed in train #979 `4376c211`; observed at main `91c83438`, omc sha256 `2aec185d4ed6d7a6a5779ed656b4e56904742bde5551b02893fb5f73916ad8d5`, two pins before the current `9920aab1`; not rerun at `24caacf4` or `be8c5130`). `sh om/tls/test/wycheproof/ledger.sh WYCHEPROOF_DIR LOGDIR` accounts for every test of every C2SP/wycheproof `testvectors_v1` file at `d17cc89f` (347 rows in `ledger.tsv`) as pass, fail, na (with the written reason), not_run or unexplained, reading `LOGDIR/SUITE.log` from the primitive suites (`aesgcm chacha sha2 x25519 ec rsa mlkem ecdh`, each `sh om/tls/test/SUITE/run.sh OMC`); exit 0 PASS, 1 FAIL (a failure, an unexplained test or a ledger error), 3 NOT_RUN (applicable tests absent). A skipped or absent suite is NOT_RUN, never a pass. `sh om/tls/test/wycheproof/ledger-nc.sh WYCHEPROOF_DIR LOGDIR` is its negative control: a deleted row, a planted `FAIL` and an altered header sha256 must each make the ledger exit 1. Observed 2026-10-31 at main `91c83438` with omc `2aec185d`, Spark cores 5-9 at nice 19: all eight suites exit 0 (`SHA2 PASS`, `CHACHA PASS`, `AESGCM PASS`, `X25519 PASS`, `EC PASS`, `RSA PASS`, `MLKEM PASS`, `ECDH PASS`, every mutant detected); `ledger.sh` exit 0 with `WYCHEPROOF-LEDGER PASS files=347 tests=145057 pass=13804 fail=0 na=131253 not_run=0 unexplained=0 errors=0`; `ledger-nc.sh` exit 0, each of row, fail and header `rc=1 detected`; control: `ledger.sh` with an empty LOGDIR exits 3 (`WYCHEPROOF-LEDGER NOT_RUN ... pass=0 ... not_run=13804`). `sh om/tls/test/bogo/run.sh OMC BORINGSSL_DIR GO` builds the pure-.om BoGo client shim and the pinned BoringSSL runner (`om/tls/test/bogo/PINS`: commit, Go version, URL and sha256; Go is a test oracle only) and prints `BOGO-RESULTS tests= pass= fail= na= skip=` plus `na-reason`/`fail-reason` lines. `na` is only two runner-side refusals of a TLS 1.3-only client (`tls12-only`: runner MaxVersion below TLS 1.3; `record-301`: BoringSSL demands 0x0301 on the initial record, which RFC 8446 5.1 does not require); na is never counted as a pass. The runs use `-loose-errors`, so expected-failure tests pass on any error; the negative control therefore re-runs passing candidates without `-loose-errors`, keeps only success-expecting ones, runs them against an echo-corrupting shim and requires `BOGO-NC ... still_pass=0`. Exit 0/1/3 as above. The runner's leaf certificates have an empty subject with a non-critical SAN, which `x509.om` refuses per RFC 5280 4.2.1.6, so run.sh applies `runner-critical-san.patch` to the runner fixture by default; with `BOGO_UNPATCHED=1` handshakes using those leaves fail with `X509_ERR_MALFORMED` (an explained policy difference, not a pass). The BoGo run was not rerun for this refresh (NOT_RUN): heavy runs go to GitHub `ubuntu-24.04-arm` runners, not the Spark.

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

### Developer kernel template (GitHub, CPU only)

[aien-dev/omega-kernel-template](https://github.com/aien-dev/omega-kernel-template) is the
public on-ramp for this backend: a developer creates a repo from it
(`gh repo create my-kernel --template aien-dev/omega-kernel-template --public --clone`), edits
`kernel.om` and `reference.py`, and pushes. A free `ubuntu-24.04-arm` Actions job then fetches the
pinned omega-mv `omc`, `irspirv.om`, `opencl-host.c` and `parity.py` (commit `369dbc96`, sha256 in
`toolchain.lock`), runs every case natively on AArch64 and as SPIR-V on a PoCL 7.2 CPU device built
by the repo's own `toolchain.yml`, compares with `reference.py`, and uploads an
`omega-kernel-receipt` artifact (`receipt.json`, `SHA256SUMS`).

- Evidence: `main` run 40426781699 PASS 138/138; negative control `demo/wrong-kernel` (overflowing
  average) run 40426783628 FAIL 20/138. This is CPU-EXERCISED evidence only: no GPU, sm_121 SASS or
  CUDA++ path runs, and the receipt says so in `not_run`.
- When omega-mv changes `omc` or `seed/spirv/`, a template pin bump needs a passing template run
  first; the PoCL asset changes only through `toolchain.yml`, whose self-test must pass.
- Its `tools/check.sh`, `tools/fetch.sh` and `tools/cases.py` are INTERIM host glue whose successor
  is an Omega conductor unit.

### Aienux SMMU probe event qualification (path 1)

- Merged: [aienux#469](https://github.com/aien-dev/aienux/pull/469) landed on main
  2026-10-30 as `dfdd5a72bbc30c05aa32da79f63f460e763d18b9` (exercised source
  `7d555ade734bd56787d4bfd477caa0e79d0a1134`, 2026-10-28). It adds
  `CK_B7B_PROBE=1 CK_B7B_EVENT_READOUT=1`. It prints all SIDs/raw event words,
  every accepted RMR identity exception, and a bounded final drain of each
  already-live SMMU queue before the final report; overflow/pending and
  inactive/failed instances are explicit. No event is not proof of no DMA.
- Exercised commands (on the source 2026-10-28, rerun 2026-10-31 at `dfdd5a72`):
  `make -C native/kernel test sanitize`,
  `make -C native/kernel iort-mutants smmu-l2-mutants`, and
  `bash scripts/qemu_ck_b7b_probe_test.sh` with `--self-test`, no flag,
  `--positive`, and `--events`, all exit 0. The new events mode observes actual
  emulated NVMe out-of-window DMA, matches decoded SID/address, intact canary
  and recovery read. DROP_EVENT is killed; missing/wrong-SID/wrong-address
  event, overflow and pending controls fail as required.
- A fixture-provisioned full hardware-staging-format artifact is **NOT
  boot-admissible**. Prerequisite 0 is the owner's actual public-key/machine
  identity ceremony; never generate substitute material. The non-staging
  core-only probe requires no Store identity, retains probe/readout code, and
  excludes device/security/Store stages; it is not the combined input/witness
  physical candidate. No staging gate was relaxed.
- Physical GB10/HDA0 faults and GPU DMA/cache visibility/recovery are NOT_RUN.
  The designed GPU test requires a native capability-bound map/submit,
  system-visibility fence, uncached completion and verified-quiesce endpoint;
  a Linux/VFIO protocol simulator does not supply it. Follow aienux
  `docs/campaigns/B7B-PHYSICAL-BOOT-PREP.md` at `dfdd5a72` or later and fresh operator
  authorization. This guide is not permission to stage, stop a service or boot.

### Path 3: bounded seeded Omega NVMe read protocol

Merged: [aienux #472](https://github.com/aien-dev/aienux/pull/472) landed on main
2026-10-30 as `aab6de6d4df4b4d99e764e91303a13d10684f5fe`; its confinement-retention
[fix #470](https://github.com/aien-dev/aienux/pull/470) landed as
`2b77170de9693a3204b65dff059e24c2cdecc197`. `native/disk` at the merge commit is
byte-identical to the exercised source `62393fb2ab03766c4be0273a1388af6d2f7247f8`.
This is HOST/QEMU EXERCISED, not physical qualification, a bootable storage stack
or a TRUST-1 Gate 7 pass.

- Use the existing driver compiler pin: omega-mv
  `397e67040fb74853c6814097fe02ae4e1ebd6f66`, omc SHA256
  `bcfa582892afca8f65e551fe56867d918d86e84540c7381be8fa9ef18d0cf0f6`.
  This compatibility pin does not replace the general compiler pin above.
- From aienux at `aab6de6d`, run `sh native/disk/tests/omega_nvme_test.sh`,
  with `AIEN_OMC` set to the absolute pinned compiler and
  `OMEGA_NVME_EVIDENCE` set to a new absolute evidence directory.
  Observed exit 0 (2026-10-28 on the source, rerun 2026-10-31 at `aab6de6d` with an omc
  rebuilt from seed at that pin): 37 host cases, seven killed guard-removal mutants and ten
  disposable-QEMU cases. The C driver is the independently compiled data oracle;
  the hosted C/QTest relay performs raw memory/register accesses, not the NVMe
  protocol. QEMU 8.2.2 and GCC 13.3.0 are verification-only dependencies. Any other
  compiler is refused before a case runs (`compiler pin mismatch`, exit 2).
- The Omega code owns Identify/manifest parsing, bounded queues and PRPs,
  head/tail/phase wrap, CID/SQID/SQ-head correlation, successful copy-out and
  fail-stop DMA retention. The central opcode guard refuses writes and other
  IO commands; removing it makes a hostile Write reach the model and fails
  the negative control. Identity, extent, format, completion and timeout
  refusals remain separately exercised.
- See `native/disk/tests/omega_nvme_receipt.txt` and
  `native/disk/OMEGA_NVME_READ.md` for hashes, manifest layout, reuse search,
  native adapter contract and the exact measurement-first physical procedure.
  The volatile/MMIO, DMA coherence/lifetime, timer and ownership adapter is an
  explicit integration prerequisite, not an excuse to call the C kernel binder.
  The existing binder reaches a write/flush/restore probe and is not read-only.
- Physical execution stays NOT_RUN until that adapter is exercised and the
  operator approves the exact device/namespace/extent, recovery/evidence media
  and service downtime. Do not open the live root block device or transfer its
  controller while Linux owns it. Capture latency/throughput before any tuning;
  host/QEMU results are not SSD measurements or hardware safety evidence.

### CUDApp portable backend (CUDApp#138, merged `0c782c44af`)

Paths below are on CUDApp main (present at `cc0908bdc0`).

- Target status lives in one place: `backend/targets.om` (`targets doctor`, printed by
  `make doctor OMC=/abs/omc`). Only the GB10 (sm_121) is SUPPORTED. Every other GPU row (other NVIDIA
  generations, AMD gfx9-12, Intel Xe, Apple, Qualcomm, Arm, Imagination, Broadcom, Vortex) is PLANNED,
  hardware NOT_RUN. PoCL and lavapipe rows are HOST-ONLY CPUs, never GPU evidence.
- `targets load FORMAT FILE [ENTRY|gfx]` is the `ob_code_load` admission check (SPIRV_CL, SPIRV_VK,
  AMDGPU_HSACO, SM121_RAW; MSL_SOURCE/AARCH64 refused as PLANNED, CUBIN reserved).
- FP32 SPIR-V comes from `backend/spirv/emit.om` (hand-built words; omc IR mode has no floats until E3,
  omega-mv#883), unlike the scalar-ulong irspirv above. Its CPU launcher is migration-pending C until
  E2 (checked shared-library calls, omega-mv#888).
- The gfx1151 HSACO on CUDApp main is an LLVM ORACLE fixture for the loader, not an AMD encoder.
- Gate: `make test-multitarget OMC=…` with negative controls; receipts
  `evidence/MULTITARGET-20261027T0638Z/` (host EXERCISED, 63 checks, CUDApp `0ec44766`). Never say a
  vendor is "supported" or "runs" from these host runs.

## 4. Effects and authority

- A proposed program or effect description does not authorize running it. Confirm capability and privilege at the execution boundary. Model confidence and semantic identity grant no authority.
- Do not weaken a gate, add a new trust path, publish secrets or fabricate hardware receipts. Never edit a historical receipt to make a new claim pass.
- Keep private machine targets, addresses, usernames and home paths in ignored local configuration. Keep intentional public attribution intact.
- Hardware and live services need the operator's current authorization procedure. Host and emulator checks never qualify hardware. Machine-specific operator rules belong in that machine's local instructions, not in public guidance.
- Escalate to an independent reviewer for hard-to-reverse changes to authority, protocols or gates, for contradictory evidence, or when two distinct fixes have failed. A model review is not an execution receipt.
- For the native GPU swarm, follow the [realization-record evidence boundary](../omega-communication/SKILL.md#native-gpu-realization-records-2026-10-28). A unit test calling an isolated cache or selector is not proof that the existing runtime records the selected code. Exercise the actual generated binding and consuming cache/command-list path, with stale-generation, wrong-artifact and missing-qualification controls. Keep device and whole-model tests NOT_RUN until their authorized execution; never populate production candidates from synthetic host records.
- **Fix it, don't file it** (Drake, 2026-10-25). A defect you can reproduce is a defect you chase in the same piece of work, including one you found while reviewing someone else's lane: read the code, build the smallest reproduction, change it, prove it, ship it. Reporting a blocker you have not tried to fix, or handing a reproduced bug to an "owner", is unfinished work. Stop only for a real external gate (authorization, a user decision, an exhausted check) and name it after the attempt.

### NVIDIA tools and licensed inputs (provenance)

Hardware facts for NVIDIA back ends follow `PROVENANCE.md` in aien-dev/CUDApp (CUDApp#110, merged as `a8878c955ac16c7b507a46e530e4ca102b935b9a`) and aien-dev/omega-mv (omega-mv#874, merged through the train omega-mv#972 as `f5c9bd89ae604a65d7c777a5739c176f41a036ec`; present at omega-mv main `91c83438daec0bdb88bf5c077cae10f3d15d365d`, omc sha256 `2aec185d4ed6d7a6a5779ed656b4e56904742bde5551b02893fb5f73916ad8d5`), which quote the clauses (CUDA Toolkit EULA, January 2025 revision, §1.2(1), §1.2(8), §1.1.2(4), §2.1; NVIDIA driver licence §2.1, §2.2; MAX License §1.4, October 21, 2026). Engineering rule, not legal advice:

- Cite public documentation (PTX ISA, CUDA Binary Utilities, open-gpu-doc), permissively licensed source (NVIDIA open kernel module headers, Mesa NAK/NVK, LLVM `ELF.h`, CUTLASS) or a chip receipt for each fact.
- `nvcc`, `ptxas`, `cuobjdump` and `nvdisasm` are proprietary, internal-use SDK tools. Run them only on inputs we wrote (our probe kernels, our encoder's bytes), only for NVIDIA targets, and record the version and command. Never ship or wrap them.
- Never disassemble, extract or read the machine code of an NVIDIA binary: libcuda, cuBLAS, cuDNN, TensorRT, NCCL, driver, firmware, or cubins inside NVIDIA libraries, on any machine including rentals.
- Never carry a fact learned from NVIDIA tool output into a non-NVIDIA back end.
- Never give MAX source to an AI system to build a substitute for MAX; reading it to interoperate is allowed.
- Record every stored NVIDIA-tool listing in the repository's `PROVENANCE.md`. A golden copied from tool output is a regression pin, not independent evidence.
- In public text, describe the work as independently implemented from public documentation and open source, checked with NVIDIA's tools on my own code. Avoid "reverse-engineered NVIDIA's compiler/driver", "CUDA clone" and an unqualified "byte-identical to nvcc".

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
| Omega syntax, keywords, types, traps, limits, error codes, host or device words, the compiler or its pin | `omega-encoding` and `writing-omega`, plus the pins in `omega-harness`; add or update the program in `examples/` and rerun `examples/gate.sh` |
| Build, compile, gate or verification commands, or the seed procedure | `omega-harness` sections 3 and 4 |
| Conversion, encoder or bridge behavior, or what it supports | the conversion note in `omega-harness` section 2, and the README |
| Status vocabulary, receipts, report or handoff conventions, wire formats | `omega-communication` |
| Repository purpose, ownership, names, archive state, plan authority, merge or Alpha policy, a resolved or new contradiction | `aien-repositories` |
| Any aien-dev repository's purpose, status or links, or a new repository | that repository's entry in [aien-dev/sovereign-intelligence](https://github.com/aien-dev/sovereign-intelligence) `index/curated.yml` (layer, evidence, one-line purpose), in the same piece of work |
| A new source of hardware facts, a new use of an NVIDIA SDK tool, or stored NVIDIA-tool output | that repository's `PROVENANCE.md`, and section 4 here if the rule changed |
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
