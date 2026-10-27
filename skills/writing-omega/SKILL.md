---
name: writing-omega
description: Portable guide to writing Omega (.om) programs that the seeded omc compiler accepts and runs correctly: program shape, types, words, traps, compiler errors 1..30, common mistakes with the exact refusal, and tested examples. Use before writing or reviewing any .om program on any machine.
---

# Writing Omega programs

This guide is for any agent or person writing `.om` on any machine. It needs only a clone of
[aien-dev/omega-mv](https://github.com/aien-dev/omega-mv) and of this repository. Every rule
below was observed with a program at the pin in the table; the programs are in
[`examples/`](../../examples/) and [`examples/gate.sh`](../../examples/gate.sh) reruns all of them.
For compiler internals, register rules, wire formats and the reasons behind each rule, read
[`skills/omega-encoding/SKILL.md`](../omega-encoding/SKILL.md).

| Pin | Value |
|---|---|
| omega-mv commit | `e5b32567a7aff3ce594e3254a622a846a6cd190b` |
| compiler `seed/out/omc` | sha256 `2c82480596bbdc50cda9626866209ef662d5834f278f839ff4ce0250c495029f` |
| language reference | `docs/one_layer/OMEGESE.md` at that commit |
| host | Linux AArch64 only; omc writes a static AArch64 Linux ELF |

The language reference is normative. Where this guide and OMEGESE disagree, the compiler at the pin
decides; this guide records the known disagreements.

## 1. Compile and run

```sh
git clone https://github.com/aien-dev/omega-mv && cd omega-mv
git checkout e5b32567a7aff3ce594e3254a622a846a6cd190b
sha256sum seed/out/omc      # must print the pinned hash
sh seed/omega-from-seed.sh  # optional: rebuild omc from the 408-byte loader
seed/out/omc < prog.om > prog; echo "compile $?"
chmod +x prog && ./prog; echo "exit $?"
```

- omc reads the unit on stdin and writes the executable on stdout. Exit 0 means accepted. Any other
  exit is the error number, and stderr has one line `omc: line N: error K`.
- **Never run the output of a refused compile**: stdout may still hold bytes.
- A unit with `fn main` is program mode (an executable). A unit without it is function mode (raw
  machine words for a host to call); never execute function-mode output directly.
- Check behavior, not compile success: compare stdout bytes and the exit status with what you
  expected, and run one deliberately wrong variant to see the check fail.
- To check a whole set the way this repository does:
  `examples/gate.sh path/to/seed/out/omc` prints `PASS name` or `FAIL name: reason` per program and
  ends `GATE PASS examples=N ...` or `GATE FAIL ...` (exit 1); a different compiler hash exits 2.

## 2. Program shape

```
note hello: the smallest complete program
text msg "hello, world\n"
fn main to int
k = write 1 msg 0 len msg
if ne k long len msg
ret 1
end
ret 0
end
```

([`examples/hello.om`](../../examples/hello.om)). The rules it shows:

- **One statement per line.** No semicolons, no braces. Blocks (`fn`, `if`, `else`, `while`, `for`,
  `struct`) close with `end`. Indentation is optional.
- **Prefix words only.** Every operation is a word followed by its values: `add x 1`,
  `mul add a b c`, `lt i n`. Each word has a fixed number of values. Parentheses may group a value
  and add nothing: `(add a b)`.
- **Comments are `note` lines.** `//` and `#` are refused (error 16).
- **`fn NAME p1 T1 p2 T2 ... to R`**, result `R` a type or `void`. Parameters fill **6 slots**: an
  OM-2 unsized `view T *` / `edit T *` takes 2 (pointer and count), every other parameter 1 (a
  scalar, a sized array, a struct). Two unsized views plus three scalars is error 9
  ([`examples/om2_param_slots.om`](../../examples/om2_param_slots.om),
  `mistake_om2_param_slots.om`). `fn main to int` takes no parameters and its `ret` value is the exit
  status (0..255).
- **Assignment is `name = value`.** The first assignment creates the name with the value's type. The
  `=` is optional, which is a trap: `return x` is accepted as "assign x to a new name `return`".
- A function may call a function defined later in the unit, and program-mode functions may recurse.
- Top-level lines are `note`, `fn`, `text`, `global`, `const`, `struct`, and in OM-2 units
  `layout`, `abstract`, `holds` and `export fn`. `struct` and `layout` lines come before the first
  `fn` (error 9 after it). A `const` is visible only after its line.

## 3. Types and values

| Type | Meaning |
|---|---|
| `bool` | `true` / `false`, the result of comparisons |
| `uchar`, `ushort`, `uint`, `ulong` | unsigned 8, 16, 32, 64 bit (also spelled `u8 u16 u32 u64`) |
| `byte`, `short`, `int`, `long` | signed 8, 16, 32, 64 bit (also spelled `i8 i16 i32 i64`); `byte` is **signed**, and `char` is not a type (error 3) |
| `f32`, `f64` | IEEE binary32, binary64 |
| `void` | no result (function result only) |

- **No implicit conversion.** Both values of an operation have one type (error 7). Convert with the
  type word: `long x`, `uint b`, `f64 n`.
- **A bare number takes its type from context**: the other value of its operation, the parameter it
  is passed to, the result type at `ret`. With no context it is error 11: write `x = uint 5`, never
  `x = 5` for a new name.
- Integer literals are decimal with an optional leading `-`. A literal that does not fit its type
  is error 8. No character literals: write `uchar 65`, not `'A'` (error 3).
- Float literals need digits on both sides of the dot: `2.0f64`, `1.0e30f64`, `0.5f32`. The suffix is
  optional where context gives the type.
- **Hex only in OM-2** (section 9): lowercase `0x` and lowercase digits, value must fit
  (`int 0xffffffff` is error 8; write `int -1`).
- `sizeof T`, `sizeof S` and `offsetof S f` are compile-time `ulong` constants.

## 4. Words

| Kind | Words |
|---|---|
| arithmetic | `add sub mul div rem neg` |
| bits | `and or xor not shl shr` (`rotr rotl` in OM-2) |
| compare (give `bool`) | `eq ne lt le gt ge` |
| float | `add sub mul div neg`, `fsqrt fabs fmin fmax` |
| arrays | `new`, `get A i`, `put A i v`, `len A` |
| structs | `new S v1 v2 ...`, `get p f`, `put p f v` |
| control | `if` / `else` / `end`, `while COND bound N`, `for i T K`, `break`, `continue`, `ret`, `exit`, `trap N` |
| conversion | the type words, `bitcast T v` |
| I/O | `read fd A off n`, `write fd A off n`, `pread` / `pwrite fd A off n pos`, and the host words of OMEGESE section 10 (`fopen`, `fclose`, `argcnt`, `argread`, `envcnt`, `envread`, `monons`, `random`, `socknew`, `fork`, and more) |

- `and` / `or` on bools evaluate **both** sides; there is no short circuit
  ([`examples/mistake_short_circuit.om`](../../examples/mistake_short_circuit.om) traps 3 on the
  index the C reader would expect to be skipped). Test the index in an outer `if`.
- Comparisons follow the type: on `ulong`, `lt x y` with x = 2^64-1 and y = 1 is false.
- `div` and `rem` on signed values round toward zero; `rem` takes the dividend's sign;
  `shr` on a signed value keeps the sign.
- A function with a result is a value, not a statement: write `v = f 1`, never a bare `f 1`
  (error 7). A `void` function is called as a statement.
- A call is the name then its values with no brackets or commas: `ret f 1 2`.

## 5. Traps: what stops a running program

A trap ends the process with exit status 100 + N. Output already written stays written.

| Cause | Trap | Exit | Example |
|---|---|---|---|
| signed overflow in `add sub mul neg` | 1 | 101 | `examples/trap1_overflow.om` |
| `div` or `rem` by zero, and signed minimum divided by -1 | 2 | 102 | `trap2_divzero.om` |
| array index or I/O slice out of bounds | 3 | 103 | `trap3_bounds.om` |
| a loop past its bound | 4 | 104 | `trap4_loop.om` |
| an integer conversion whose value does not fit | 5 | 105 | `trap5_conv.om` |
| stack exhausted (8 MiB) by calls or local arrays | 6 | 106 | `trap6_stack.om` |
| shift or rotate amount outside 0..width-1 | 7 | 107 | `trap7_shift.om` |
| a contract check that fails (OM-2) | 9, 10 | 109, 110 | `contract_requires.om`, `contract_ensures.om` |
| `trap N`, N 1..14 | N | 100+N | `trap_code.om` (`trap 14`, exit 114) |

- Unsigned arithmetic wraps modulo 2^width and never traps
  ([`examples/unsigned_wrap.om`](../../examples/unsigned_wrap.om)).
- Integer conversions are checked: `uchar x` with x = 300 traps 5. Keep the low bits on purpose with
  a mask: `uchar and x 255`.
- Float to integer truncates toward zero and saturates at the type's limits; NaN gives 0; it never
  traps ([`examples/float_to_int.om`](../../examples/float_to_int.om)). No float operation traps:
  1/0 is infinity, 0/0 is NaN, and NaN compares false except with `ne`.
- `exit N` ends the process with status N directly, without the trap convention.

## 6. Loops and bounds

- `while COND bound N` runs the body at most N times, N 1..999999 (error 8 outside). Without
  `bound` the limit is 1000000. One more pass traps 4. Choose N from an argument you can write down
  in a `note`, for example "every pass consumes at least one byte of a 65536-byte input".
- `for i T K` runs i from 0 to K-1. `for` takes no `bound` (error 5); K is the limit.
- `break` and `continue` work in both loops. There is no `else if`: nest an `if` inside `else`.
- No loop runs forever and recursion is bounded by the stack.

## 7. Arrays, texts, globals, structs

- **Local array:** `a = new T N v`, N elements each v, only in a function's outermost block (error 9
  inside a block). N is a literal or an integer `const`; a variable N is error 16. There is no heap
  and no `free` (error 9).
- **Global array:** `global NAME T N` at top level, zero-filled, program mode only. **A global is
  never an argument** (error 19); a function uses it by naming it.
- **Text:** `text NAME "bytes"` is a read-only `uchar` array. Escapes are only `\n \t \\ \"`
  (error 22 for others).
- **Array parameters** state type and exact length: `a view uchar 64` (read-only) or
  `a edit uchar 64` (writable). Another length is error 7. In OM-2, `a view T *` / `a edit T *` take
  any length. Writing to a view or text is error 20.
- `get A i` / `put A i v`: a literal index outside the array is error 21; every other index is checked
  at run time (trap 3). There are no pointers and no address arithmetic.
- **Structs:** `struct Pair a uint b long end`, before the first `fn`. Every field is an 8-byte cell,
  so a struct is never a wire or file layout. `p = new Pair 5 -12`, pass as `p edit Pair` or
  `p view Pair` ([`examples/structs.om`](../../examples/structs.om)).
- **Bytes on the wire:** build them with shifts and masks into a `uchar` array
  ([`examples/wire_le.om`](../../examples/wire_le.om)); OM-2 also has `layout`.

## 8. Names and limits

- Variable, parameter, global, text, const, struct and field names: at most 8 characters (error 13).
  Function names: at most 63, but see section 13 for names that start with an 8-letter keyword.
- A new name may not repeat a visible name, a function or a struct (error 10). A name first assigned
  inside a block ends at that block's `end`; assign it before the block to keep it.
- Reserved words cannot be names, including `len read write text new free view edit global note
  exit const bound` and every host word (`stat`, `link`, `send`, `random`, `fork`, ...). `len` as a
  name is error 16, `free` is error 9.
- 16 registers per function hold parameters, live names, loop counters and the temporaries of the
  current statement. The 17th is error 12 (`examples/err12_registers.om`). The fix is structural:
  move part of the work into a helper function that takes a few scalars and returns one value, and
  end the life of names you no longer need by computing them inside a block.
- Size limits: an array up to 16777216 elements, 4 MiB of local arrays per function, 1 GiB of
  globals (error 23).

## 9. OM-2 units

A unit whose first line that is not blank or `note` is exactly `om2` is an OM-2 unit. OM-1 code keeps
its meaning inside it. OM-2 adds:

- lowercase hex literals; `rotr x n`, `rotl x n` on unsigned types (trap 7 for an amount outside
  0..width-1);
- unsized parameters `a view T *` / `a edit T *` with `len a` the run-time count;
- `part A off n`, only as an argument to an unsized parameter: the n elements of A from off,
  checked at run time (trap 3). A part of a global is error 19; as a value of its own it is error 9;
  to a sized parameter it is error 7;
- `for i T K unroll`, `fn ... inline`, `layout` (external byte layouts), `fresh` scratch arrays
  (error 27), `abstract` types with `seal` / `open` (error 28), `export fn` (at most 32, error 14);
- executable contracts: `requires` and `ensures` lines at the start of a function, `invariant` as the
  first line of a loop body. A failing `requires` traps 9 (exit 109), a failing `ensures` or
  `invariant` traps 10 (exit 110). A clause may not call a function, use floats or host words
  (error 30). The first line `om2 release` erases the checks
  ([`examples/contract_release.om`](../../examples/contract_release.om)).

Examples: [`om2_tour.om`](../../examples/om2_tour.om), [`om2_views.om`](../../examples/om2_views.om),
[`om2_hex_rotr.om`](../../examples/om2_hex_rotr.om), [`contracts_ok.om`](../../examples/contracts_ok.om).

## 10. Host I/O

- Every host word returns a `long`: a count, or minus the Linux errno. `read` may return fewer bytes
  than asked, 0 at end of input, or a negative error; `write` may write fewer. Loop until done and
  handle `k < 0` and `k == 0` separately
  ([`examples/stdin_count.om`](../../examples/stdin_count.om)).
- Descriptors, flags and modes are `int`; offsets, counts and capacities `ulong`; buffers `uchar`
  arrays. Slices are checked before the kernel call (trap 3).
- `main` takes no parameters: read arguments with `argcnt` and `argread`
  ([`examples/echo_args.om`](../../examples/echo_args.om)) and the environment with `envcnt` and
  `envread` ([`examples/env_read.om`](../../examples/env_read.om)). Short capacity returns -105.
- Host words carry the process's own permissions; they are not a sandbox.

## 11. Compiler errors

Exit status = error number; stderr `omc: line N: error K`. One refusal per code is in
`examples/errNN_*.om` with its exact stderr line in the matching `.expect`.

| Code | Meaning | Code | Meaning |
|---|---|---|---|
| 1 | could not read input | 16 | syntax: unknown word or misplaced form |
| 2 | could not write output | 17 | a line after `ret`, `break`, `continue`, `trap` or `exit` |
| 3 | unknown name or field | 18 | a result function whose `end` is reachable |
| 4 | line ends too early | 19 | a global as an argument, or one array twice in a call |
| 5 | extra words on the line | 20 | writing to a text or view |
| 6 | malformed number | 21 | literal index out of range |
| 7 | type mismatch | 22 | bad text escape |
| 8 | number out of range | 23 | array or pool size limit |
| 9 | word in a disallowed place | 24 | branch displacement (not reproduced at the pin) |
| 10 | duplicate name | 25, 26 | not emitted at the pin |
| 11 | number with no type | 27 | OM-2 `fresh` element read before written |
| 12 | register pool exhausted | 28 | OM-2 `seal` outside the constructor |
| 13 | name too long | 29 | OM-2 `layout` field out of size or overlapping |
| 14 | compiler capacity (names, functions, exports, expansion) | 30 | OM-2 contract clause not allowed |
| 15 | missing `end` | | |

## 12. Common mistakes

Each row is a program in `examples/` with its observed refusal.

| Mistake | Write instead | Refusal | Example |
|---|---|---|---|
| `y = x + 1` | `y = add x 1` | error 5 | `mistake_infix.om` |
| `f(1, 2)` | `f 1 2` | error 6 | `mistake_c_call.om` |
| `if x > 0` | `if gt x 0` | error 7 | `mistake_compare_operator.om` |
| `'A'` | `uchar 65` | error 3 | `mistake_char_literal.om` |
| `// comment` | `note comment` | error 16 | `mistake_slash_comment.om` |
| `x = 1;` | `x = uint 1` | error 6 | `mistake_semicolon.om` |
| `return x` | `ret x` | error 18 (silent assignment) | `mistake_return.om` |
| `else if c` | `else`, then a nested `if c` | error 5 | `mistake_else_if.om` |
| `fn main argc int to int` | `argcnt`, `argread` | error 9 | `mistake_main_args.om` |
| `for i ulong n bound 10` | `while lt i n bound 10` | error 5 | `mistake_for_bound.om` |
| `while c bound 1000000` | a bound 1..999999 | error 8 | `mistake_bound_range.om` |
| `a = new uchar n 0` | a literal or `const` length | error 16 | `mistake_new_var_len.om` |
| `free a` | nothing: no heap | error 9 | `mistake_free.om` |
| `len = ...` | another name | error 16 | `mistake_reserved_name.om` |
| a bare call `f 1` with a result | `v = f 1` | error 7 | `mistake_value_stmt.om` |
| a 7th parameter | a struct or an array | error 9 | `mistake_seven_params.om` |
| 2 `view T *` plus 3 scalars (7 slots) | a sized view, a struct, or fewer scalars | error 9 | `mistake_om2_param_slots.om` |
| `emit buf` with `buf` a global | name the global inside the function | error 19 | `err19_global_arg.om` |
| `a view uchar 8` given a 16-element array | the exact length, or OM-2 `view uchar *` | error 7 | `mistake_param_length.om` |
| `trap 15` | `trap 1` .. `trap 14` | error 8 | `mistake_trap_range.om` |
| `struct` after a `fn` | every `struct` before the first `fn` | error 9 | `mistake_struct_after_fn.om` |
| `rotr` or hex without `om2` | an `om2` first line | error 3 | `mistake_om1_rotr.om` |
| `0X10` or `0xFF` | `0x10`, `0xff` | error 6 | `mistake_om2_upper_hex.om` |
| `int 0xffffffff` | `int -1` | error 8 | `mistake_om2_int_hex.om` |
| `part` of a global | `part` of a local | error 19 | `mistake_om2_part_global.om` |
| `part` to a sized parameter | an unsized `view T *` parameter | error 7 | `mistake_om2_part_sized.om` |
| `s = part a 0 2` | `part` only as an argument | error 9 | `mistake_om2_part_statement.om` |
| `if and lt i n eq get a i 0` (C short circuit) | an outer `if lt i n` | trap 3, exit 103 | `mistake_short_circuit.om` |

## 13. Known compiler defects at the pin

- **`bitcast` across signedness below 64 bits** (fix proposed in draft omega-mv #896, not in the
  pin): the result does not compare equal to the reinterpreted value. Observed wrong: `uint` to
  `int` (4294967295 is not -1, and adding 2 traps 1), `int` to `uint`, `ushort` to and from
  `short`, `uchar` to and from `byte`, and `f32` to `int` (-1.0 is not -1082130432 and not below 0).
  Observed correct: `ulong` to and from `long`, `f64` to `long`, `f32` to `uint`, `int` to `f32`.
  Until a fixed compiler is pinned, do not `bitcast` between narrow integer types or from `f32` to
  `int`; decode signed 8, 16 and 32-bit values arithmetically: load into `long`, then subtract 2^32
  (or 2^16, 2^8) when the value is at least 2^31 (2^15, 2^7), as
  [`examples/wire_le.om`](../../examples/wire_le.om) does. For float bits, `bitcast uint` of an
  `f32` is correct.
- **Empty text through `view uchar *`** (omega-mv issue #775): the parameter does not arrive with
  `len` 0 (observed 60 and 28, depending on the other texts in the unit), so a loop over it runs past
  the text (a crash has been reported; not reproduced here). Workaround: pass `part e 0 0`, which
  arrives with `len` 0
  ([`examples/empty_text_part.om`](../../examples/empty_text_part.om)).
- **Function names that begin with an 8-letter keyword:** a function name longer than 8 bytes whose
  first 8 bytes are a keyword (`continue_x`, `truncate_number`, `sockname_x`, `execprog_x`,
  `peercred_x`) is error 16, although function names may be 63 characters. Shorter keyword prefixes
  are fine (`breakx`, `whilex`, `getpid_x`). Workaround: rename (`cont_x`, `trunc_num`)
  ([`examples/defect_keyword_prefix.om`](../../examples/defect_keyword_prefix.om)).
- A fix for both is in progress in omega-mv; until a new compiler is pinned, use the workarounds.
- OMEGESE 9.9 lists `+ - * /` for floats; the compiler refuses them. Write `add sub mul div`.

## 14. Example index

All in [`examples/`](../../examples/); each `NAME.om` has `NAME.expect` (compile status, exit status
or stderr line) and, when it prints, `NAME.stdout` with the exact bytes. Wolfram checks of the
arithmetic expectations are in [`examples/WOLFRAM.md`](../../examples/WOLFRAM.md).

- Start here: `hello`, `exit_status`, `print_decimal`, `functions`, `loops`, `arrays`, `text`.
- Arithmetic: `int_arith`, `unsigned_wrap`, `checked_conv`, `float_arith`, `float_to_int`, `consts`.
- Data: `globals`, `structs`, `wire_le`.
- I/O: `stdin_count`, `echo_args`, `env_read`, `tmp_files`.
- Traps: `trap1_overflow` .. `trap7_shift`, `trap_code`.
- OM-2 and contracts: `om2_tour`, `om2_views`, `om2_hex_rotr`, `om2_part_trap`, `om2_param_slots`,
  `contracts_ok`, `contract_requires`, `contract_ensures`, `contract_invariant`, `contract_release`.
- Known defects and their workarounds: `empty_text_part`, `defect_keyword_prefix`.
- Refusals: `err01_input` .. `err30_contract`, `mistake_*`.
