---
name: omega-encoding
description: Write .om that the seeded Omega compiler actually accepts: syntax, types, traps, bounds, register limits, host I/O, byte encoding, compiler errors, verified examples. Use before writing, reviewing or explaining any Omega program.
---

# Omega encoding: writing `.om` that the seeded compiler actually accepts

Use this before writing, reviewing or explaining any `.om` program, before claiming what Omega can or
cannot express, and before encoding bytes, integers or host I/O in Omega.

## 0. Pin and how to re-check it

The general-language citations below retain their dated historical pin. The newer,
scope-specific pin immediately below is the tested IR/wasm candidate, not main.

| Item | Value |
|---|---|
| Repository | `github.com/aien-dev/omega-mv`, branch `main` |
| Commit | `55f9b0af47de70a3d06357985cf99ae3e45c016f` (2026-10-25, merge of PR #766) |
| Compiler source `seed/omc/omc.om` | 7,838 lines, sha256 `2e35db7d6c2066fb4a20d19b8a9579c44caca9c9937b0594a89422010b673637` |
| Committed compiler `seed/out/omc` | sha256 `734f0a39319fbfa930bcc62cb30dbf06d47d78e126372b74f5baf478e6126b61` |
| Seed loader input `seed/rung0/seed0.hex` | sha256 `baaf081fe6055a768485c5d25d7435681ce00b04cfa80b26c48169f78944e14d` |
| Language document `docs/one_layer/OMEGESE.md` | blob `89a27d2e68519eb45c103abb23dfe05a68d8ad69` |

Every citation links to that commit: `https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/<path>#L<n>`.

### Pending IR/wasm candidate (2026-10-26)

Source: omega-mv `8e083692ae1f597966c8d2eb6a92292bb278eb41`,
[draft #787](https://github.com/aien-dev/omega-mv/pull/787), stacked on the separate
boolean IR fix [draft #786](https://github.com/aien-dev/omega-mv/pull/786).
Pending until both changes land; do not describe this backend as available on main.

| Candidate item | Verified value |
|---|---|
| `seed/omc/omc.om` | sha256 `bd64bb6f20952818e155328f429ab6cd36bba213e3e00b60d796336f9d0c9343` |
| `seed/out/omc` | sha256 `befb8199db8c4bca188a494dfaac2a6f0ee5c36fcea11188c75666c55fce8a1d` |
| Prior main compiler at `ec07d25d5506d33567adbd4d9b518177b7933bad` | sha256 `499236398aabc652e2dae27fe4bdc4c5db3d3726314356e0dbef23e3345803ec` |
| `seed/wasm/irwasm.om` | sha256 `8dc7bf602eb882e26a18b9e2dc4cacdd3ad68827215e71874b14c4acb237ccf8` |

The candidate corrects IR-mode boolean AND/OR/XOR typing: all three previously
exited 7 and now compile to bytes identical to the C encoder, accepted by checker2.
Boolean arithmetic and shifts remain refused. The om4-built compiler and two
self-rebuilds are byte-identical; the full compiler referee/wrong-word-mutant gate
ended `GATE5 PASS`. Retained evidence:
`evidence/IR-BOOL-20261026/` in draft #786.

`seed/wasm/irwasm.om` is a seeded Omega backend for **scalar OSC IR v1**, not an
arbitrary foreign-source converter. It writes an import-free WebAssembly MVP module
and exports each function under its ASCII IR name. ABI tags bool/u8/u16/u32/i8/i16/i32
map to i32, u64/i64 to i64, void to no result. Internal canonical i64 locals preserve
width and signedness; signed overflow, invalid shifts/casts/division, explicit traps
and lowered loop guards become wasm runtime traps. Wasm does not expose Omega's
numeric trap code. Supported instructions: CONST/MOV/BIN/UN/CMP/CAST/CALL/TRAP/BR/CBR/RET.
Limits and exact mappings:
[`OMEGESE.md` section 11.1](https://github.com/aien-dev/omega-mv/blob/8e083692ae1f597966c8d2eb6a92292bb278eb41/docs/one_layer/OMEGESE.md#L914).
Versions other than 1, memory/reference/arena/pool/slice tags and operations refuse.
The module is buffered so parse/type/capacity refusals have empty stdout.

Host command: `nice -n 19 sh seed/wasm/gate-irwasm.sh`.
Observed on Linux AArch64, Node v24.21.0 used **only as a read-only oracle**:
`PASS parity cases=129 units=13 traps=29; every export exercised`.
The parity checker rejected an add-to-sub IR-byte mutant with exit 1; two translations
of every unit were identical, and named refusal exits were pinned with empty stdout.
Source-bound full logs/artifacts: `evidence/IRWASM-20261026/` in draft #787.
No browser integration, GPU qualification, merge or deployment was run.

Existing guide examples were rerun with this candidate compiler: FNV `abc` emitted
`0b e9 47 1a`, compile/run exit 0; `trap_narrow.om` compiled with exit 0 and ran with
exit 105. Removing only the unmasked store (negative control) changed the run exit
to 44. The historical examples and their old measured evidence below remain intact.


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

## 1. Which language is which

| Thing | Where | Status |
|---|---|---|
| **Omegese**, the `.om` language | `docs/one_layer/OMEGESE.md`, sections 1-11 | current |
| The seeded compiler `omc`, written in `.om` | `seed/omc/omc.om`, built `seed0 -> hex1 -> asm2 -> om4 -> omc` by `sh seed/omega-from-seed.sh` | on `main` since PR #696 ([README.md#L27](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/README.md#L27)) |
| Keyword table, the authority for reserved words | `fn kw` in `seed/omc/omc.om` ([#L375](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/omc.om#L375)); same list in `seed/omc/genkw.awk` ([#L16](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/omc/genkw.awk#L16)), parity checked by `seed/omc/gate18.sh` | gated |
| C translator `tools/omegese/omegese.c` and `omegese next` | OMEGESE sections 1-8 | legacy referee and token-mask tool, not the compiler |
| `spec/omega-language-v0.md` (`let x: u8 = ...`, `fn f(x: T) -> T { }`, `//` comments) | Visor console line language, lowered by C in `src/language/` ([#L1-L13](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/spec/omega-language-v0.md#L1-L13)) | history; never write `.om` in that syntax |
| C under `src/`, including `src/compiler/` | `main` | unfinished migration ([README.md#L50](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/README.md#L50)) |

OMEGESE sections 2-6 describe the C translator's rules; sections 9-11 describe `omc`. Where they
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

## 1a. Compile and run (Linux AArch64 host only)

```sh
OMC=omega-src/seed/out/omc                 # after checking its sha256 against section 0
"$OMC" < prog.om > prog 2> prog.err; echo "compile rc=$?"; cat prog.err
chmod +x prog && ./prog; echo "run rc=$?"
```

Compile rc 0 with empty stderr is acceptance; any other rc is the error number of section 11.
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
| 3 | unknown name or field | 14 | capacity: 1024 names, 512 functions, 512 nested blocks, 1024 nesting depth, 8 MiB input/output |
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
Capacity boundaries are gated in `seed/omc/gate12.sh` (e.g. 1024 globals accepted, 1025 refused;
511 functions plus `main` accepted, 512 plus `main` refused).

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
   it. Known decision-blocked gaps at this pin ([OM_COMPLETION.md#L62-L72](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/docs/one_layer/OM_COMPLETION.md#L62-L72)):
   packed structs (FR-2), imports (FR-3, units concatenate), threads/atomics (FR-6), C ABI/shared
   objects (FR-7), x86-64 (FR-9), heap (decision 24).
6. Documents disagree with the code at this pin; trust the code and name the conflict:
   OM_COMPLETION FR-5 still says mmap/ioctl are absent, and `seed/host/MAPPING.md`
   ([#L66-L68](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/seed/host/MAPPING.md#L66-L68)) still calls the device words a proposal not
   on `main`, while `omc.om` reserves `devctl mapnew ... mapbind` (ids 166-177) and README
   ([#L27](https://github.com/aien-dev/omega-mv/blob/55f9b0af47de70a3d06357985cf99ae3e45c016f/README.md#L27)) says they are on `main`. GPU launch and model execution through
   them are not qualified. README line 5 also still says the seeded compiler is only on candidate
   branches.

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
