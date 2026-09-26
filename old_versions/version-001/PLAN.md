# quad — the plan

quad is an AI written in **satellite** that programs in **C++** first, and later in
satellite too.

C++ comes first because it is finished: every program quad writes can be checked with a
compiler, and there is plenty of C++ documentation to train on when we need it.

This plan is still being worked out. Nothing below is decided until it says so.

## Properties

quad's knowledge is held as **properties**. A property is a dotted name for one thing
quad knows about a language. There will be a `property` class, and quad's runtime is
the set of property objects made from it.

Examples of property names so far:

```
cxx.function.forward_declaration.<something>
cxx.function.no_forward_declaration.<something>   (the same things as .forward_declaration,
                                                  for when no forward declaration is needed)
cxx.namespace.std.cout.string
cxx.namespace.std.cout.number
cxx.namespace.std.cout.newline
```

In C++ almost everything lives inside a function, apart from forward declarations of
functions. That is why `cxx.function` is where quad's knowledge starts.

### What a property holds

**Decided:**

- Everything in C++ is broken down into individual objects. `std::cout << "hi" << "\n";` is
  `std::cout`, `<<`, `"`, the ASCII text, `"`, `<<`, `"\n"` and `;`.
- The semicolon is an object at the end of every command. It is a special class in quad, a
  special type of object.
- The objects are held in a `satellite.container.multiple<every object type>`.
- Properties have dotted names.
- This definition is not complete yet. The idea: quad flips through its properties and writes
  its own programs that way. Later it gets the ability to define new properties.

**Proposed** (not decided):

- A `satellite.container.multiple` holds ONE value at a time (it is satellite's `std::variant`).
  So each position in a property is that multiple, and a list gives the order:

  ```
  satellite.container.list<satellite.container.multiple<word, symbol, header, open_quote, ascii, close_quote, newline, semicolon, hole>> objects
  ```

- Every object class extends one base spacesuit, `piece`, so all of them answer the same
  capsules: `call_text()`, `call_kind()` and `call_ends_command()`.
- A complete property holds eight things:

  | part | what it holds |
  |---|---|
  | NAME | the dotted name |
  | OBJECTS | the objects, in order |
  | PLACE | the kind of hole it may fill: `root`, `include`, `function`, `command`, `cout_item` |
  | HOLES | hole objects inside OBJECTS, each with the PLACE it takes and its least and most fillers |
  | NEEDS | other properties the file must also hold, e.g. `cxx.include.iostream` |
  | DOES | what it adds to the output: `ascii`, `number`, `line_end`, `holes`, `nothing` |
  | RECORD | `TRIED`, `PASSED`, `FAILED=` (g++'s first error line) |
  | STATUS, ORIGIN | seed, untested, proven or broken; made by the author or by quad; `MADE_OF` |

- A grammar is the set of rules for which pieces may go where. quad's grammar is PLACE plus
  HOLES. The dotted names already follow it: `cxx.namespace.std.cout` is a command with a
  `cout_item` hole, and `.string`, `.number` and `.newline` are what may fill it.
- Using a property never changes it. A program is built in a new list, the draft, and the
  values (the text between the quotes) go only into the draft.
- Properties are data, kept in one file, `properties.text.c4`. quad can add knowledge only by
  writing data, because satellite cannot load new code while it runs.

### The objects (proposed)

- `piece`: the base class. It holds the text, the kind, and the gap written in front.
- `word`: `std::cout`, `int`, `main`, `return`, `0`, `#include`.
- `symbol`: `<<` `(` `)` `{` `}`.
- `header`: `<iostream>`, as one object.
- `open_quote` and `close_quote`: the same `"` character, in two classes, because different
  things may follow each one.
- `ascii`: the text between the quotes. It is empty in a property and filled when quad writes
  a program. It may not hold `"`, `\` or a line break.
- `newline`: `"\n"` as one object.
- `number`: digits (M7).
- `hole`: a gap. It holds the PLACE it takes, least and most.
- `semicolon`: the special class. It is the only object whose `call_ends_command()` is true.
  - A `command` property ends in exactly one semicolon and holds no other.
  - It is the joint between commands.
  - The writer starts a new line after it.
  - Later it is where quad cuts C++ it reads into commands.
  - It ends commands, not lines: `#include` lines end at the line end, `}` ends a block, and
    the two `;` in `for ( ; ; )` will need their own class.

### An example (proposed)

`std::cout << "hi" << "\n";` is 8 objects: word, symbol, open_quote, ascii, close_quote,
symbol, newline, semicolon. In `properties.text.c4`:

```
PROPERTY=cxx.namespace.std.cout
PLACE=command
NEEDS=cxx.include.iostream
DOES=holes
OBJECT=word std::cout
OBJECT=hole cout_item 1 0
OBJECT=semicolon
ORIGIN=author
STATUS=seed
TRIED=0
PASSED=0
END
PROPERTY=cxx.namespace.std.cout.string
PLACE=cout_item
DOES=ascii
OBJECT=symbol <<
OBJECT=open_quote
OBJECT=ascii
OBJECT=close_quote
END
PROPERTY=cxx.namespace.std.cout.newline
PLACE=cout_item
DOES=line_end
OBJECT=symbol <<
OBJECT=newline
END
```

The seed properties for the first program are these three plus three more:

- `cxx.program`: `[include 0..] [function 1]`
- `cxx.include.iostream`: `#include <iostream>`
- `cxx.function.no_forward_declaration.main`: `int main ( ) { [command 0..] return 0 ; }`

### How quad writes C++ (proposed)

1. `goal.text.c4` holds the lines the program must print.
2. The draft starts as `cxx.program`'s objects. quad walks it from left to right.
3. Fixed objects are kept as they are.
4. At a hole, quad flips through every property and keeps only those:
   - whose PLACE is what the hole takes, and
   - whose DOES can make the front of what is still wanted.
5. If several fit, quad prefers:
   - proven over untested;
   - the one whose DOES covers more of what is wanted;
   - the narrower one (number before ascii);
   - the better record;
   - fewer objects.
6. If nothing fits, quad says which hole it could not fill.
7. The chosen objects go in where the hole was, and are walked next. An `ascii` is replaced by
   a new one that holds the text. A hole that may repeat stays until nothing more is wanted.
8. NEEDS fill the include hole last.
9. A shape check runs: no hole is left, brackets balance, and every command ends in its
   semicolon.
10. quad writes `program.cxx.c4`, and `program.expected.text.c4` with the output predicted from
    DOES.

### How quad checks a program (proposed)

A program is good only if it passes all three gates:

1. **Shape**: checked in satellite, before writing.
2. **Compiler**: `g++ -std=c++17 -Wall -Wextra` must print nothing. Warnings count as failure.
3. **Behaviour**: what the program printed must equal the prediction and the goal, byte for
   byte.

After the first PASS, a program may use at most one property that is not yet proven, so a
failure names its culprit.

Satellite cannot run another program yet: `satellite.system.run` gives S110. So between two
runs of `satl quad.satl`, the compile line is typed by hand:

```
rm -f program program.output.text.c4
g++ -std=c++17 -Wall -Wextra -x c++ -o program program.cxx.c4 2> program.errors.text.c4 && timeout 5 ./program > program.output.text.c4
```

quad reads the two result files at its next start, before it writes anything.

### Files quad adds (proposed)

- `properties.text.c4`: every property.
- `goal.text.c4`: the lines to print.
- `program.cxx.c4`: the C++ quad writes.
- `program.expected.text.c4`: the output quad predicts.
- `program.errors.text.c4` and `program.output.text.c4`: written by the compile line.
- `grammar.text.c4`: what may follow what (M8).

The full reasoning, the worked first program and a satellite sketch are in
[HOW_QUAD_WRITES.md](HOW_QUAD_WRITES.md).

## Files

- Every quad file ends in `.c4`.
- A text file (strings) is `.text.c4`.
- `version.text.c4` holds quad's version, revision and build:

  ```
  VERSION=001
  REVISION=01
  BUILD=0001
  ```

## Milestones

### M1 — quad starts (done)

`quad.satl` shows quad's name in ASCII art and the version, revision and build it reads
from `version.text.c4`. Every run raises `BUILD` by 1. Inside the program the version is
the float `quad_version` (`0.001`), shown as `001`.

```
satl quad.satl
```

### M2 — the objects (proposed)

In `quad.satl`: the base spacesuit `piece`, and nine classes that extend it: `word`,
`symbol`, `header`, `open_quote`, `ascii`, `close_quote`, `newline`, `hole`, and the special
`semicolon`, the only one whose `call_ends_command()` answers true. After the banner, quad
builds the 8 objects of `std::cout << "hi" << "\n";` by hand into a
`satellite.container.list<satellite.container.multiple<…every object class…>>` and joins their
text.

**Done when:** `satl quad.satl` shows, under the banner:

```
std::cout << "hi" << "\n";
8 objects, 1 ends a command
```

The 1 is counted by asking every object `call_ends_command()`. It is not typed.

### M3 — the property class, read from a file (proposed)

- The `property` spacesuit: NAME, PLACE, DOES, NEEDS and OBJECTS.
- `make_piece(kind, text)`: the one capsule every object is made through.
- `properties.text.c4`: the six seed properties, `cxx.program`, `cxx.include.iostream`,
  `cxx.function.no_forward_declaration.main`, `cxx.namespace.std.cout`, `.string` and
  `.newline`, written as `PROPERTY=` / `PLACE=` / `NEEDS=` / `DOES=` / `OBJECT=<class> <text>` /
  `END` blocks.

quad loads the file at every start and checks the semicolon rule.

**Done when:** `satl quad.satl` shows `quad knows 6 properties` and one line per property, e.g.
`cxx.namespace.std.cout  command  std::cout [cout_item 1..] ;`. A seventh block, a command
without its semicolon, shows `refused …: a command ends with its semicolon`, and the count
stays 6.

**Satellite:** a folder listing cannot be put into a list (S210), so there is one properties
file. `find()` stops the run when the text is missing (S420), so the loader checks with
`contains()` first.

### M4 — the first program, by fixed steps (proposed)

This milestone adds:

- the draft;
- splicing a property's objects in where a hole was (a hole that may repeat stays after them);
- filling the ascii with a new object;
- the NEEDS pass;
- the shape check;
- the writer, with spacing, line breaks and indent.

The choices are fixed in code for now: main, then `std::cout`, then `.string` with `hi`, then
`.newline`. quad writes `program.cxx.c4`. It uses `satellite.file.new` the first time;
afterwards it clears the file and opens it again.

**Done when:** `satl quad.satl` shows `wrote program.cxx.c4: 19 objects, 2 commands`. Then the
compile line prints nothing, and `./program` prints `hi`. A second `satl quad.satl` rewrites the
file instead of failing.

**Satellite:** it cannot run g++ (MISSING.md row 5), so the compile line is typed by hand.
`satellite.system.delete` is not built (S210).

### M5 — flipping through, toward a goal (proposed)

`goal.text.c4` holds the lines to print. The fixed choices go. At every hole, quad flips
through all properties and keeps those whose PLACE fits and whose DOES can make the front of
what is still wanted. The ascii rule refuses `"`, `\` and line breaks. quad also writes
`program.expected.text.c4`.

**Done when:**

- With goal `hi`, `satl quad.satl` shows its four choices
  (`hole function <- cxx.function.no_forward_declaration.main`, …) and writes the M4 program.
- With goal lines `one` and `two`, it writes `std::cout << "one" << "\n" << "two" << "\n";`.
- With goal `say "hi"`, it shows `quad: no property I know prints a quote` and writes nothing.

### M6 — quad judges its program (proposed)

Each property gets `TRIED=`, `PASSED=` and `FAILED=`. At every start, before writing, quad
judges a waiting program. It is PASS only when `program.errors.text.c4` is empty and the
`read_all` of `program.output.text.c4` equals `program.expected.text.c4`. quad then adds to
TRIED, and to PASSED on a pass, for every property the program used. It does this with
`file.replace`, the same way `BUILD=` is raised now.

**Done when:** running `satl quad.satl`, then the compile line, then `satl quad.satl` shows
`PASS: the program printed what quad wanted`, and all six properties read `TRIED=1 PASSED=1`.
With `std::cout` changed to `std::cuot` in `properties.text.c4`, the same three steps show
`FAIL: did not compile` with g++'s first error line.

### M7 — every property proves itself (proposed)

This milestone adds:

- `STATUS=` (seed, untested, proven, broken) and `ORIGIN=` (author or quad).
- The first PASS proves the six seeds together.
- After that, a program may use at most one property that is not proven. A FAIL marks that
  property `STATUS=broken` with `FAILED=`.
- With `goal.text.c4` empty, quad flips through every property and writes a test program for the
  next unproven one, using a sample value.
- The first new seed: the `number` class and `cxx.namespace.std.cout.number` (`<<` then a
  number, `DOES=number`).
- Choosing prefers proven, then the narrower property.

**Done when:**

- With an empty goal, `satl quad.satl` shows `testing cxx.namespace.std.cout.number`. After the
  compile line and one more run, it reads `STATUS=proven`.
- Goal `42` then shows `hole cout_item <- cxx.namespace.std.cout.number`.
- A property typed wrong on purpose ends `STATUS=broken` with a `FAILED=` line, and is never
  chosen again.

### M8 — quad works out its grammar (proposed)

From PLACE and holes alone, quad works out what may follow each object:

- inside a property: the next object;
- into a hole: the first object of every property that fits it;
- out of a hole: the last object;
- a hole that may repeat joins its last objects to its first;
- a hole that may be empty lets its neighbours meet.

Only proven properties count. quad writes the pairs to `grammar.text.c4`. This table is the
guard every property quad invents must pass.

**Done when:** after `satl quad.satl`, `grammar.text.c4` holds, among others, `std::cout -> <<`,
`<< -> "\n"`, `; -> }` and `{ -> return`. An untested property typed with an opening quote
straight after `std::cout` is shown as `new pair, not proven: std::cout -> "`.

### M9 — the first property quad defines (proposed)

After a PASS in which `.string` and then `.newline` filled the same cout hole side by side,
quad saves them as one filler:

- name `cxx.namespace.std.cout.string.newline`, with `PLACE=cout_item`;
- objects `<< " [ascii] " << "\n"`;
- `DOES=ascii` then `DOES=line_end`;
- `ORIGIN=quad`, `MADE_OF` both, `STATUS=untested`.

It must pass the M8 guard, then its own test program. quad never saves the same `MADE_OF`
twice. Choosing now prefers the property whose DOES covers more of what is wanted.

**Done when:**

- `properties.text.c4` holds a block with `ORIGIN=quad` that nobody typed.
- After its test, that block reads `STATUS=proven`.
- Goal `hello` then shows `hole cout_item <- cxx.namespace.std.cout.string.newline` and writes
  `std::cout << "hello" << "\n";`.

### M10 — functions quad names itself (proposed)

This milestone adds:

- two new seeds from the author: a void function (`void [name] ( ) { [command 0..] }`) and a
  call (`[name] ( ) ;`);
- a `name` object;
- holes in `cxx.program` for declarations and for functions before and after main.

quad wraps a proven body in a function it names (`say_0001`), calls it from main, and keeps
it only if the program prints what it printed before. Then it moves the definition after main,
with `void say_0001 ( ) ;` at the top.

**Done when:** one goal is met both ways, both compile cleanly and print the goal, and
`properties.text.c4` holds `cxx.function.no_forward_declaration.say_0001` and
`cxx.function.forward_declaration.say_0001` with `ORIGIN=quad`.

### Later (proposed, not yet milestones)

- Learn NEEDS from g++'s "not declared" errors.
- Removal tests: drop one object or need, and see whether the program still passes.
- Swap one object for another of the same class, and learn DOES by watching the output.
- Cut real C++ into objects and commands at each semicolon.
- quad sets its own goals.

## Satellite gaps (proposed)

- **Running another program.** `satellite.system.run` gives S110 on build 0103 (MISSING.md
  row 5). This blocks only the unattended loop; until then, a person runs the compile line.
  Asked for: `satellite.system.run(command)`, answering the exit code and output.
- **Calling capsules on a multiple.** A name or parameter declared as a multiple of two or more
  spacesuits cannot call capsules (S110). quad reaches objects as `objects[i]` or through
  `piece`.
- **Missing words**, and what quad does instead:
  - no `x.kind`: every object answers `call_kind()`;
  - no type alias: the long type is spelled out in full;
  - no folder creation and no folder listing into a list: one properties file, next to
    `quad.satl`;
  - no `system.delete`: clear the file, then open it;
  - no try/catch: check with `contains()` before `find()`;
  - no `&&` or `||`: nested ifs;
  - no command-line arguments: `goal.text.c4`.
- **Rules to write around:**
  - a name may be declared only once per capsule (S202);
  - an object must be made on its own line (S110);
  - a base spacesuit's constructor may take no arguments (S110).
- **No longer missing:** includes, break and continue, and the string methods `.at`,
  `.substring`, `.split`, `.contains` and `.starts_with`.
