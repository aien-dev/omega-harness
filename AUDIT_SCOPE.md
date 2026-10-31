# Audit scope

Written for an outside, independent security audit. Revision described:
`eb07daf4ae9a3aa4cab66049a60c3f3c2be2aced` on `main`, 2026-10-26. Everything below is taken from
the repository's own files at that revision; items marked NOT_RUN were not executed while writing
this document.

## What this repository is

Agent skills for working in Omega (the `.om` language of AienOS) and in the AIEN repositories.
Six skills live under `skills/<name>/SKILL.md` in the Agent Skills layout, and
`skills/omega-encoding/examples/` holds two runnable `.om` programs. AGENTS.md states that the
repository is documentation and procedure, not a compiler, a runtime, a gate, or evidence that any
claim is true. Nothing here ships as executable code; the examples are teaching material compiled
with the compiler from `aien-dev/omega-mv`. License: Apache 2.0.

## What an auditor can and cannot check here

Can check:

- Whether any skill text tells an agent to do something unsafe (leak a secret, weaken a gate,
  act without authority, rewrite evidence).
- Whether any file carries credentials, private addresses, home paths or usernames.
- Whether the two example programs compile and behave as `skills/omega-encoding/SKILL.md`
  section 13 records, against the pinned compiler.
- Whether the shell recipes in `README.md` and `AGENTS.md` do what the text says.

Cannot check here:

- The compiler, its seed, or its traps and bounds. Those live in `aien-dev/omega-mv`; this
  repository only cites them at a pin.
- Whether an agent actually follows a skill. The skills are guidance; no harness enforcement lives
  in this repository.
- Hardware behaviour. The examples are host programs only.

## What to review, in priority order

1. `skills/omega-harness/SKILL.md` section 4 "Effects and authority" and section 6 "Keep the
   guides current". These are the rules that tell an agent when it may act. Check that no
   wording authorises an effect from model confidence, a proposal, or a stale skill.
2. `skills/wolfram-math/SKILL.md` "Setup". It tells an agent to add an MCP server entry pointing
   at `https://agenttools.wolfram.com/mcp` and states that any credential is injected from a vault
   at launch and never written to a file. Check that no step leads an agent to store a token.
3. `skills/crumb-protocol/SKILL.md`. Crumbs are public files; section 9 item 5 forbids secrets in
   them. Check the rest of the skill does not contradict that.
4. `skills/omega-encoding/examples/fnv1a32.om` (55 lines) and `trap_narrow.om` (9 lines). The first
   reads stdin with a partial-read loop into a bounded buffer and refuses 65536 bytes with exit 2;
   check the bound and the partial-read and partial-write handling. The second shows a narrowing
   trap (exit 105).
5. `skills/aien-repositories/SKILL.md`. Routing and repository status. Check it names no private
   repository detail beyond what it marks as private, and no private paths.
6. `README.md` "Install" and "Check the examples yourself", and `AGENTS.md` "Omega pins". These are
   commands an operator copies. They clone public repositories and run a committed binary.

There is no `.github/workflows/` directory in this repository.

## Trust boundaries

- Input: the skill text is loaded into an agent's context and influences what it does. A
  malicious edit to a skill is the main risk; `main` refuses direct pushes
  (`skills/aien-repositories/SKILL.md` records that since 2026-10-25 this repository and one other
  are pull-request-only).
- The example programs read untrusted bytes from stdin.
- Trusted: the committed compiler `seed/out/omc` in `aien-dev/omega-mv` at the pin recorded in
  `skills/omega-encoding/SKILL.md` section 0, identified by sha256.
- External: the hosted Wolfram MCP endpoint named in `skills/wolfram-math/SKILL.md`.

## Threat model summary

- Assets: the integrity of the guidance agents follow; the absence of secrets and private
  machine detail in a public repository.
- Attackers considered: a contributor or agent who submits a skill change that misleads other
  agents; an accidental leak of private configuration.
- Defended: AGENTS.md forbids credentials, private addresses, home paths, SSH usernames and
  operator-only configuration in the skills, and forbids rewriting old evidence. `main` is
  pull-request-only.
- Not defended / explicitly out of scope: enforcement inside agent harnesses; the correctness of
  the compiler; hardware. A compile is not a run, a host run is not hardware qualification, and a
  merged PR is not a deployment (AGENTS.md).

## Build from source at a pin

There is no build. AGENTS.md: "There is no build and no test suite: the deliverables are Markdown
and two example programs."

To compile the examples you need a Linux AArch64 host and the committed compiler from
`aien-dev/omega-mv`. `skills/omega-encoding/SKILL.md` section 0 records the general pin:

- omega-mv commit `55f9b0af47de70a3d06357985cf99ae3e45c016f` (2026-10-25, merge of PR #766)
- `seed/omc/omc.om` sha256 `2e35db7d6c2066fb4a20d19b8a9579c44caca9c9937b0594a89422010b673637`
- `seed/out/omc` sha256 `734f0a39319fbfa930bcc62cb30dbf06d47d78e126372b74f5baf478e6126b61`
- `seed/rung0/seed0.hex` sha256 `baaf081fe6055a768485c5d25d7435681ce00b04cfa80b26c48169f78944e14d`

Later sections of the same skill record scoped pins for newer compiler work; check the one that
matches what you test.

## Run every test

There is no test suite. The only executable check is the example recipe from `README.md`
"Check the examples yourself" (Linux AArch64 only), NOT_RUN while writing this document:

```sh
git clone https://github.com/aien-dev/omega-mv.git omega-src
sha256sum omega-src/seed/out/omc
omega-src/seed/out/omc < skills/omega-encoding/examples/fnv1a32.om > fnv 2> fnv.err
chmod +x fnv
printf abc > fnv.in
./fnv < fnv.in > fnv.out
od -An -tx1 fnv.out
```

Expected, per `README.md` and `skills/omega-encoding/SKILL.md` section 13: compile exit 0 with empty
stderr, and output bytes `0b e9 47 1a` for input `abc`. Section 13 also lists the recorded results
for other inputs, the 65536-byte refusal (exit 2), `trap_narrow.om` (run exit 105) and the negative
controls. The clone above takes the current omega-mv `main`, so compare the printed sha256 with the
pin before trusting the result.

## Known limitations and NOT_RUN

- The recorded example results are host-only (2026-10-25, compiler `734f0a39...6b61`).
- The README recipe clones omega-mv `main` rather than checking out the pinned commit; results
  only match the record when the compiler digest matches.
- Each skill names the omega-mv commit it was checked against; README says skills list known
  disagreements and that historical sections keep their dated pins. A skill is unverified for a
  tree whose hashes differ (AGENTS.md).
- Agent compliance with the skills is not measured in this repository.

## Dependencies and provenance

- No lockfile, no vendored code, no package dependencies.
- The examples depend on `aien-dev/omega-mv` (cloned at run time) and a Linux AArch64 host.
- `skills/wolfram-math/SKILL.md` relies on the hosted Wolfram MCP endpoint for agents' arithmetic.
- No GitHub Actions.

## Evidence

Observed example results are recorded inline in `skills/omega-encoding/SKILL.md` section 13 and
in the dated pin sections of that skill. Older measurements stay as dated history and are not
rewritten (AGENTS.md, "Do not edit old evidence" in README).
