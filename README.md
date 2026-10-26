# Omega Harness

Agent skills for working in Omega, the `.om` language of AienOS, and in the AIEN repositories. An agent that loads these skills finds the actual source before acting, writes `.om` code that the seeded compiler accepts, runs what it builds, and reports its status without overstating it.

Skills can't make agents flawless. What these do is turn the mistakes agents keep repeating into checks they must run before they claim anything: stale syntax, guessed compiler limits, compiling without running, labeling host-only runs as qualified, and choosing the wrong repository.

| Skill | Use it for |
|---|---|
| [`omega-harness`](skills/omega-harness/SKILL.md) | The overall workflow: orient, implement in `.om`, compile, run, challenge, handle effects, learn from repeated errors |
| [`omega-encoding`](skills/omega-encoding/SKILL.md) | The language the seeded compiler accepts: prefix words, types, traps, bounds, registers, host I/O, byte layouts, error codes, plus two runnable examples |
| [`omega-communication`](skills/omega-communication/SKILL.md) | Status labels, reports, PR text, agent handoffs, receipts, wire formats, corrections |
| [`aien-repositories`](skills/aien-repositories/SKILL.md) | What AIEN builds, who decides what, and which repository owns a task |
| [`crumb-protocol`](skills/crumb-protocol/SKILL.md) | Directory coordination: read and respect crumbs, claim and release locks, keep committed crumbs fresh, answer `CRUMB-VERIFY` and `crumb-doctor` |
| [`wolfram-math`](skills/wolfram-math/SKILL.md) | Mandatory Wolfram for every computation: MCP setup an agent can do alone, verification calls, failure handling |

The skills use the Agent Skills layout (`skills/<name>/SKILL.md` with `name` and `description` frontmatter), so any agent that reads that layout picks them up.

## Install

Clone the repository, then put the skill directories where your agent looks for skills:

```sh
git clone https://github.com/aien-dev/omega-harness.git
```

| Agent | Skill directory |
|---|---|
| oh-my-pi (`omp`) | `~/.omp/agent/skills/` or a project's `.omp/skills/` |
| Claude Code | `~/.claude/skills/` or a project's `.claude/skills/` |
| Codex and other Agent Skills readers | `~/.agents/skills/` or a project's `.agents/skills/` |

```sh
cp -R omega-harness/skills/* ~/.agents/skills/   # pick the directory for your agent
```

`omp` can also read them in place. Add the clone's `skills` directory to `skills.customDirectories` in `~/.omp/agent/config.yml`. Once installed, agents load a skill automatically when the task matches its description. In `omp` you can also invoke one directly with `/skill:omega-harness`.

## Conversion

These skills cover working in Omega itself. Conversion between Omega and other languages belongs to [`omega-bridge`](https://github.com/aien-dev/omega-bridge). It converts a declared, checked subset and refuses to attempt universal conversion. You can run an MCP server in front of it if you want automated conversion, but that wrapper doesn't let it convert anything beyond that subset.

The merged [omega-mv WebAssembly backend (#787)](https://github.com/aien-dev/omega-mv/pull/787)
is a compiler backend for the existing scalar OSC IR v1 wire format, not a foreign-source
converter. Its seeded Omega writer and read-only Node oracle boundary are documented in
`omega-encoding` section 0 and `omega-harness` sections 2-3. Backend merge:
`3364a1dfd0ca50c1b82bf19ba34932d8bc10a7a9`; boolean IR prerequisite #786:
`87489da90a1a7a750fd4a7f9eeb005b1221721f6`.

## Check the examples yourself

The examples run on Linux AArch64 hosts only:

```sh
git clone https://github.com/aien-dev/omega-mv.git omega-src
sha256sum omega-src/seed/out/omc   # 734f0a39319fbfa930bcc62cb30dbf06d47d78e126372b74f5baf478e6126b61 at the pin
omega-src/seed/out/omc < skills/omega-encoding/examples/fnv1a32.om > fnv 2> fnv.err
compile_rc=$?
printf 'compile rc=%s\n' "$compile_rc"
cat fnv.err
if [ "$compile_rc" -eq 0 ]; then
    chmod +x fnv
    printf abc > fnv.in
    ./fnv < fnv.in > fnv.out
    run_rc=$?
    printf 'run rc=%s\n' "$run_rc"
    if [ "$run_rc" -eq 0 ]; then od -An -tx1 fnv.out; fi  # 0b e9 47 1a
fi
```

The `omega-encoding` skill (section 13) records the full set of observed results and the negative controls.

## Keeping it current

**The agent that makes a change updates the guides.** If your work changes the Omega language, the compiler or its pin, build or gate commands, the encoder or bridge, repository ownership or status, or this harness, update the matching skill in the same piece of work. Then open a PR here with the evidence. If you can't, open an issue saying what is now stale. `omega-harness` section 6 has the full map of which change goes in which skill.

Each skill names the omega-mv commit it was checked against (main `55f9b0af47de70a3d06357985cf99ae3e45c016f`, 2026-10-25) and lists the known places where the sources disagree. `omega-encoding` section 0 adds newer, scoped pins: the IR/wasm backend at `3364a1dfd`, the current main `19f014f21` with the OM-2 contract and the Omega 1.0 roadmap, and the executable-contracts candidate `b2869978` (not merged). When the compiler changes, recheck the cited lines, rerun the examples, then update the pins. Do not edit old evidence. Contributions follow the [shared AIEN agent rules](https://www.aienos.com/agents/RULES.md).

Written by Drake Stapleton with AIEN agents. Apache License 2.0.
