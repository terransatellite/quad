# How quad begins to write C++ from properties

*Written 2026-09-26, in answer to: "from there how does quad ai begin to write in C++?" Proposed, not decided. PLAN.md holds the short version.*

## The short answer

Your definition says what a property **is**: its C++ objects, in order. It does not yet say five things quad needs before it can write anything:

1. where the property may go,
2. what may fill its gaps,
3. what else the program must contain,
4. what it prints when the program runs,
5. whether it has been proven to work.

Once those are added, "flipping through properties" becomes one exact step. At each gap in the program, quad looks at every property, keeps only the ones allowed there, and takes the one that prints what was asked for. Then g++ and a test run judge the result. From six properties, quad's first program is `std::cout << "hi" << "\n";` inside `main`. New properties come later: quad saves, renames and varies things that have already passed. It keeps each new one only after a program that uses it passes.

Everything below keeps what you decided:
- each C++ piece is its own object;
- the semicolon is a special class;
- the objects are held in a `satellite.container.multiple<every object type>`;
- properties have dotted names.

Every satellite fact below was checked by running satl 004 revision 08 build 0103. Nothing here is decided until PLAN.md says so.

---

## 1. One correction: what a multiple holds

A `satellite.container.multiple` holds **one** value at a time, of any one of the types it lists. It is satellite's `std::variant`, not a container.
- The help text says it "allows one name to hold any one of several types, the way a std::variant does. It is not a container".
- `type_shape.cpp` says "ANY ONE OF THEM".
- `multiple<string, number> seq = {"std::cout", "<<"}` is refused with S301.
- The tuple-like multiple in `old_versions/second_satellite/ACCESS_PLAN.md` (section A3) was an older plan. Satellite 004 did not build it.

So your multiple is exactly right for **one position** in a property: that position may hold any object type. A list around those positions gives the order:

```
satellite.container.list<satellite.container.multiple<word, symbol, header, open_quote, ascii, close_quote, newline, semicolon, hole>> objects
```

This works today. Two details make it easy to use:
- **One shared base class.** Every object class extends one base spacesuit, `piece`, so they all answer the same capsules: `call_text()`, `call_kind()` and `call_ends_command()`. Satellite cannot ask a multiple which type it holds, so each object reports its own kind.
- **How to reach one object.** A name or parameter declared as a multiple of two or more spacesuits cannot call capsules (S110). So quad reaches one object as `objects[i]`, or through a name or parameter of type `piece`. Both work.

## 2. What is missing, and the complete property

Without the missing parts, quad can only glue objects together blindly. With 9 object classes, the 8 positions of your cout line alone can be filled 9^8, about 43 million, ways, and almost none of them compile. That is token soup. The missing parts are what let quad choose.

One term first. A **grammar** is the set of rules for which pieces may go where. quad's grammar lives inside the properties, in PLACE and HOLES below.

A complete property holds eight things:

| # | part | what it is | why it is needed |
|---|---|---|---|
| 1 | NAME (yours) | the dotted name, e.g. `cxx.namespace.std.cout.string` | how you and quad refer to it; the dots group families |
| 2 | OBJECTS (yours) | the list of multiples above | the C++ itself |
| 3 | PLACE | the kind of gap it may fill: `root`, `include`, `function`, `command`, `cout_item` | without it, quad could drop a cout item where a function belongs |
| 4 | HOLES | gap objects inside OBJECTS; each names the PLACE it takes and how many fillers, least to most | how properties nest, and where the choices are |
| 5 | NEEDS | other properties the file must also contain, e.g. `cxx.include.iostream` | `std::cout` without the include does not compile, and nothing in the object sequence says so |
| 6 | DOES | what it adds to the output: `ascii` (prints its text), `number`, `line_end`, `holes` (prints whatever fills its holes), `nothing` | the only way quad can pick the property that does what was asked, and predict what the program will print |
| 7 | RECORD | `TRIED`, `PASSED`, and `FAILED=` (g++'s first error line) | experience: what has worked |
| 8 | STATUS, ORIGIN | seed / untested / proven / broken; made by the author or by quad; `MADE_OF` its parents | trust: only proven properties serve goals, and you can trace everything quad invents |

Two rules go with it:
- **Using a property never changes it.** Writing a program builds a new list, the *draft*. The text "hi" goes into a new `ascii` object in the draft, not into the property. Satellite lists share spacesuit objects by reference (MISSING.md section 4), so filling a property in place would change it for every later program.
- **Properties are data** in one file, `properties.text.c4`. Satellite cannot load a new class or capsule while it runs ("a capsule held in a variable" is "not planned"). So writing data is the only way quad can ever add knowledge. That is what makes "quad defines new properties" possible at all.

### The object classes

Every class extends `piece`, which holds the text, the kind, and the gap written in front of the object.
- `word`: a fixed name, keyword or literal: `std::cout`, `int`, `main`, `return`, `0`, `#include`.
- `symbol`: `<<` `(` `)` `{` `}`.
- `header`: `<iostream>` as one object. Spaced out as `#include < iostream >`, g++ fails with "fatal error: iostream : No such file or directory".
- `open_quote`, `close_quote`: the same `"` character in two classes, because different things may follow each one.
- `ascii`: the text between the quotes. It is empty inside a property and filled when quad writes a program. Its fill rule: no `"`, no `\`, no line break.
- `newline`: `"\n"` as one object of four characters, as you wrote it. In satellite source it is spelled `"\"\\n\""`, because a plain `"\n"` there is a real line break.
- `number`: digits. It arrives in M7.
- `semicolon`: the special class.
- `hole`: a gap. It names the PLACE it takes, plus least and most.

### Why the semicolon is special

- It is the only class whose `call_ends_command()` answers true. quad counts commands by asking every object that question.
- A property whose PLACE is `command` must end in exactly one semicolon and contain no other. quad refuses to load one that breaks this rule.
- It is the joint between commands. After it comes either the first object of another command or the `}` that closes the block. It is the one point where the writer steps back out to decide what comes next.
- The writer starts a new line after it.
- Later it becomes the knife: when quad reads C++, it cuts the text into commands at each semicolon.

It ends every **command**, not every line:
- `#include <iostream>` ends at the end of its line;
- `int main() {` and `}` are not commands;
- the two `;` inside `for ( ; ; )` end nothing, so they will need their own class when loops arrive;
- a struct ends in `};`.

A forward declaration such as `void say_hi();` does end in a semicolon. That is PLAN's `cxx.function.forward_declaration`.

## 3. One property, broken into its objects

Your line `std::cout << "hi" << "\n";` is exactly 8 objects:

| # | object | class | from property | what may follow it |
|---|---|---|---|---|
| 1 | `std::cout` | word | `cxx.namespace.std.cout` | `<<` |
| 2 | `<<` | symbol | `…cout.string` | opening `"` or `"\n"` |
| 3 | `"` | open_quote | `…cout.string` | the ascii |
| 4 | `hi` | ascii | filled from the goal | closing `"` |
| 5 | `"` | close_quote | `…cout.string` | `<<` or `;` |
| 6 | `<<` | symbol | `…cout.newline` | opening `"` or `"\n"` |
| 7 | `"\n"` | newline | `…cout.newline` | `<<` or `;` |
| 8 | `;` | semicolon | `cxx.namespace.std.cout` | the first object of any command, or `}` |

Your dotted names already are the grammar:
- `cxx.namespace.std.cout` is a command: `std::cout`, then a hole that takes cout items, then the semicolon.
- `.string`, `.number` and `.newline` are the items that may fill that hole.

Here they are in `properties.text.c4`. quad reads this file as raw text, so `"\n"` needs no escaping there.

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

`hole cout_item 1 0` means the hole takes cout items: at least 1, with no upper limit. The last column of the table above answers your "after std::cout comes <<", and nobody types it by hand. quad works it out from the properties (M8).

The other three seed properties for the first program:
- `cxx.program` (PLACE=root): `[include 0..] [function 1]`
- `cxx.include.iostream` (PLACE=include): `#include <iostream>`
- `cxx.function.no_forward_declaration.main` (PLACE=function, DOES=holes): `int main ( ) { [command 0..] return 0 ; }`

## 4. How quad writes C++, step by step

**The goal.** `goal.text.c4` holds the lines the program must print. "What is still wanted" starts as the whole goal. Goals come from a file because satl does not pass command-line arguments to a program yet.

**The draft.** quad copies the root property's objects into a new list. Then it walks that list from left to right:

1. **Fixed objects.** A word, symbol, quote or semicolon is kept as it is.
2. **Flipping through.** At a hole, quad flips through **every** property and keeps only the ones whose PLACE is what the hole takes. From those, it keeps the ones whose DOES can make the start of what is still wanted:
   - text → `ascii`;
   - a line end → `line_end`;
   - digits → `number`;
   - `holes` fits whenever something is still wanted.
3. **Choosing.** If several fit, quad prefers, in this order:
   - proven over untested;
   - the one whose DOES covers more of what is wanted;
   - the narrower one (number before ascii);
   - the better record;
   - fewer objects.
4. **Nothing fits.** quad stops and says which hole it could not fill. That is its honest "I don't know how yet".
5. **Splicing.** The chosen property's objects go in where the hole was.
   - An `ascii` among them is replaced by a new `ascii` holding the text, once the fill rule passes.
   - A hole that may repeat stays after the new objects, to be asked again. A hole that takes exactly one filler goes away.
   - A repeating hole closes once it has at least its minimum number of fillers and nothing more is wanted.
6. **Nesting.** quad does not skip past the new objects. It walks them next, so their own holes get filled too.
7. **NEEDS.** quad notes the chosen property's NEEDS, and fills the include hole from them last.

**Check and write.** Before writing, quad runs a shape check:
- no hole is left;
- the brackets balance;
- every command property ends in its semicolon.

Then it joins the objects with their gaps. A new line starts after `;`, `{`, `}` and a header line, and each `{` adds four spaces of indent. quad writes two files:
- `program.cxx.c4`: the C++. PLAN says every quad file ends in `.c4`, and `g++ -x c++` compiles it.
- `program.expected.text.c4`: the output quad predicts by adding up the DOES of every property used.

If a choice leads to a hole that nothing can fill, quad steps back and tries the next candidate. That is a small search with a depth limit. The first programs never need it.

## 5. How quad knows the program is good, and who runs g++

A program is good only if it passes three gates, checked cheapest first:

1. **Shape**, inside satellite, before anything is written. These are the checks above, plus the ascii fill rule and every NEED present. This gate can prove a program bad, but never good.
2. **Compiler**: `g++ -std=c++17 -Wall -Wextra`. It passes only if g++ prints nothing, so a warning counts as a failure.
3. **Behaviour.** The program runs with a 5-second limit. What it printed must equal both the prediction and the goal, byte for byte. `read_all` keeps the final line end (probed: size 3 for `hi` plus a line end), so the comparison is exact. If the program compiles but prints something else, then some property's objects or its DOES are wrong.

**Blame.** The first program proves the six seed properties together. After that, a program may use at most **one** property that is not yet proven, so a failure points at exactly one property. quad marks it `STATUS=broken` and stores g++'s first line as `FAILED=`.

**Who runs the compiler.** Satellite cannot start another program.
- On build 0103, `satellite.system.run("g++ --version")` is refused with S110 "satellite.system has no word named run". (MISSING.md row 5 recorded S0521 earlier.)
- The only words under `satellite.system` are delete, environment, home, memory, threshold and persist.
- No milestone promises a run word.

So each `satl quad.satl` run is one **turn**, and the compile happens between turns. These two lines do it:

```
rm -f program program.output.text.c4
g++ -std=c++17 -Wall -Wextra -x c++ -o program program.cxx.c4 2> program.errors.text.c4 && timeout 5 ./program > program.output.text.c4
```

The `rm` matters: if compiling fails, an old program must not answer in its place. On its next run, before writing anything new, quad reads the two result files and decides PASS or FAIL.

MISSING.md row 5, written for the earlier QUAD, calls running programs "the one gap QUAD must not paper over with a shell script". So:
- **Now:** type the two lines yourself.
- **Your call:** whether to save them as a `check.sh`. It would decide nothing; every decision stays in quad.satl.
- **The real fix:** a `satellite.system.run(command)` word in satellite that answers the exit code and the output. The result files already hold exactly what that word would hand back, so nothing else in quad changes when it lands.

## 6. quad's first complete program

The goal is `hi`, so what is wanted is `hi` plus a line end. `[x]` marks a hole.

0. The draft is `cxx.program`: `[include 0..] [function 1]`
1. **Include hole:** left for last.
2. **Function hole:** of the six properties, only `…no_forward_declaration.main` has PLACE=function. The draft becomes:
   `[include] int main ( ) { [command 0..] return 0 ; }`
3. `int main ( ) {` is kept.
4. **Command hole:** something is still wanted, and the only PLACE=command property is `cxx.namespace.std.cout`. quad notes that it NEEDS `cxx.include.iostream`. The command hole can repeat, so it stays after the new objects:
   `… { std::cout [cout_item 1..] ; [command 0..] return 0 ; }`
5. **cout_item hole:** what is wanted starts with text, so quad looks for DOES=ascii and finds `.string`. `hi` passes the fill rule. What is wanted is now a line end:
   `std::cout << " hi " [cout_item] ;`
6. **cout_item hole:** what is wanted starts with a line end, so quad picks `.newline`. Nothing more is wanted.
7. **cout_item hole:** it has 2 fillers (it needed at least 1) and nothing more is wanted, so it closes. The semicolon ends the command.
8. **Command hole:** nothing is wanted, so it closes. `return 0 ; }` is kept.
9. **Include hole:** takes what was needed, `#include <iostream>`.
10. **Shape check** passes: 19 objects, and 2 of them answer `call_ends_command()` true.
11. quad writes `program.cxx.c4`:
    ```cpp
    #include <iostream>
    int main() {
        std::cout << "hi" << "\n";
        return 0;
    }
    ```
    and `program.expected.text.c4`, which holds `hi` and a line end.
12. **You compile:** you run the two lines. g++ prints nothing, and the program prints `hi`.
13. **Next run:** on the next `satl quad.satl`, the errors file is empty and the output equals the prediction, so quad shows `PASS`. All six properties go to `TRIED=1 PASSED=1`, and they are now proven.

This walk has been run end to end on satl build 0103. g++ 13.3 with `-Wall -Wextra` printed nothing, and the program printed exactly `hi` and a line end. Two other goals on the same build:
- goal lines `one` and `two` gave one command: `std::cout << "one" << "\n" << "two" << "\n";`;
- the goal `say "hi"` was refused with "no property I know prints a quote", and nothing was written.

## 7. From here to quad defining its own properties

Each stage adds one way of inventing. Every invented property starts as `ORIGIN=quad`, `STATUS=untested`. Before any goal may use it, it must pass two checks:
- **The grammar guard.** Every pair of neighbouring objects in it must already appear in the may-follow table built from proven properties, its brackets must balance, and a command must end in its semicolon.
- **Its own test program,** in which it is the only unproven property.

The stages:

1. **Proving (M7).** With no goal, quad flips through every property and writes a test program for the next unproven one, with a sample value (text `quad`, number `7`). This is the plainest form of "flip through all the properties".
2. **Joining (M9): the first property nobody typed.** Suppose a program passed with `.string` and then `.newline` filling the cout hole side by side. quad saves the pair as one filler:
   - `cxx.namespace.std.cout.string.newline` = `<< " [ascii] " << "\n"`;
   - DOES = ascii then line_end;
   - MADE_OF the two.
   For the next goal, `hello`, one choice writes your own line: `std::cout << "hello" << "\n";`. To be honest, this is memory, a shortcut. It lets quad write nothing it could not write before.
3. **Wrapping and moving (M10).** You give quad two seed shapes: a `void` function and a call. quad puts a proven body into a function it names itself, and checks that the program still prints the same thing. That gives `cxx.function.no_forward_declaration.<name>`. Moving the definition after `main`, with a forward declaration at the top, gives `cxx.function.forward_declaration.<name>`. These are your two PLAN names, filled with names quad chose.
4. **Learning by experiment.**
   - Drop a NEED and compile. If the program still passes, the need was false.
   - When g++ says `'cout' is not a member of 'std'`, try each include property until the program compiles, then write the NEEDS line.
   - Drop one object. If the program still passes, the object was optional. This is how quad would find that `return 0;` is optional in `main`.
   - Swap one object for another of the same class (`std::cout` for `std::cerr`) and record DOES from what the program actually printed.
5. **Reading C++.** quad cuts real C++, first your examples and later documentation, into objects, and into commands at each semicolon. A run of objects that matches no property becomes a candidate, with holes wherever two examples differ: `int x = 5;` and `int y = 7;` give `int [name] = [number] ;`. Satellite now has the string methods this needs (`.at`, `.substring`, `.split`).
6. **Its own goals.** For example:
   - "test every untested property";
   - "meet every goal already met again with fewer choices";
   - "use two proven properties together that have never been used together".
   Later: goals just past what it can already do.

**What this amounts to, honestly.** The "living" part is this loop running:

want → flip through properties → write → compile → run → compare → keep or drop → new goals

quad's store of properties grows, and everything in it has earned its place by passing g++ and a run. That is real, checkable learning, in a narrow sense.

What it is not:
- **It is search plus testing.** It grows only through the operations it has (join, wrap, move, swap, drop, read) and through goals, whether you give them or it sets them by simple rules. It will not invent a loop or a variable until it has seen one or has an operation that can reach one. Each new operation is a decision you make.
- **DOES only describes printed output for now.** Once programs go past printing (variables, loops, input), quad can only judge by running and comparing. Choosing then becomes a real search, and that search needs limits.
- **It does not know what a program is for.** "Wanted" means "prints this text".
- **It cannot run without a person** until satellite can start g++.
- **Safety.** Once quad can swap objects and read C++, some programs it writes could loop forever or touch files. Run them only in quad's own folder, with the time limit, and never as root.

## 8. Milestones

Each milestone is small and ends with something `satl quad.satl` shows. Where a milestone needs a compile, "the compile line" means the two lines in section 5, typed by you. The full PLAN.md text is in the plan section.

- **M2: the objects.** In quad.satl: the base class `piece`, and the nine classes that extend it. quad builds your 8 objects by hand into a list of multiples.
  *Done when* `satl quad.satl` shows `std::cout << "hi" << "\n";` and `8 objects, 1 ends a command`. The 1 is counted by asking each object `call_ends_command()`, not typed. The sketch below already prints exactly these lines.
- **M3: the property class, read from a file.** The property class holds NAME, PLACE, DOES, NEEDS and OBJECTS. `make_piece(kind, text)` is the one capsule every object is made through. `properties.text.c4` holds the six seed properties. quad loads the file at every start and checks the semicolon rule.
  *Done when* the run shows `quad knows 6 properties` and one line for each, e.g. `cxx.namespace.std.cout  command  std::cout [cout_item 1..] ;`. A seventh block, a command typed without its semicolon, shows `refused …: a command ends with its semicolon`, and the count stays 6.
- **M4: the first program, by fixed steps.** The draft, splicing, the NEEDS pass, the shape check and the writer. The choices are fixed in code for now. The first write uses `satellite.file.new`; later writes clear the file and open it again.
  *Done when* the run shows `wrote program.cxx.c4: 19 objects, 2 commands`, the compile line prints nothing, and `./program` prints `hi`. A second run rewrites the file instead of failing.
- **M5: flipping through, toward a goal.** The fixed choices are replaced by the chooser from section 4, working from `goal.text.c4`. The ascii fill rule is enforced, and quad writes `program.expected.text.c4`.
  *Done when*:
  - goal `hi` shows quad's four hole choices and gives the M4 program;
  - goal lines `one` and `two` give the single chained command;
  - goal `say "hi"` shows `quad: no property I know prints a quote`, and nothing is written.
- **M6: quad judges its program.** TRIED, PASSED and FAILED= on each property. At every start, before writing, quad judges the waiting program from the two result files.
  *Done when* `satl quad.satl`, then the compile line, then `satl quad.satl` shows `PASS` and TRIED=1 PASSED=1 on all six properties. With `std::cout` changed to `std::cuot` in properties.text.c4, the same three steps show `FAIL: did not compile` with g++'s first error line.
- **M7: every property proves itself.** STATUS and ORIGIN. The rule of one unproven property per program. Exercise mode for an empty goal. The first new seed: the `number` class and `cxx.namespace.std.cout.number`. The preference order from section 4.
  *Done when*:
  - with an empty goal, the run shows `testing cxx.namespace.std.cout.number`, and after one compile the property reads STATUS=proven;
  - goal `42` shows `hole cout_item <- cxx.namespace.std.cout.number`;
  - a property typed wrong on purpose ends up STATUS=broken with a FAILED= line.
- **M8: quad works out its grammar.** quad builds the may-follow table from PLACE and holes, using proven properties only, and writes it to `grammar.text.c4`.
  *Done when*:
  - the file holds `std::cout -> <<`, `<< -> "\n"`, `; -> }` and `{ -> return`;
  - an untested property in which an opening quote comes straight after std::cout is shown as `new pair, not proven: std::cout -> "`.
- **M9: the first property quad defines.** The join from section 7, which must pass the grammar guard and then its own test.
  *Done when*:
  - properties.text.c4 holds a block with ORIGIN=quad that nobody typed;
  - after its test that block reads STATUS=proven;
  - goal `hello` shows `hole cout_item <- cxx.namespace.std.cout.string.newline`.
- **M10: functions quad names itself.** Wrap and move, using two new seed shapes from you and a `name` object.
  *Done when* one goal is met both ways, both programs compile cleanly and print the goal, and properties.text.c4 holds `cxx.function.no_forward_declaration.say_0001` and `cxx.function.forward_declaration.say_0001` with ORIGIN=quad.

## 9. Satellite gaps

Nothing blocks M2 to M10. The only thing blocked is the unattended loop. What each gap costs:

| gap (build 0103) | evidence | what quad does instead |
|---|---|---|
| running another program | S110 "no word named run"; MISSING.md row 5 | you run the compile line between turns (M4 on); **blocks the unattended loop** |
| calling a capsule on a name declared as a multiple of 2+ spacesuits | S110; program_check.cpp `suit_it_may_hold` | reach objects as `objects[i]`, or through a `piece` name or parameter |
| asking which type a value holds | help text: "Not built yet"; `x.kind` "not planned" | each object answers `call_kind()` |
| a short name for a long type | none exists | the long `list<multiple<…>>` is spelled out in full wherever objects are held |
| putting a folder listing into a list | S210 (MILESTONES M14) | one `properties.text.c4` (M3) |
| making a folder | S110 "no word named new"; MISSING.md row 1 | every file sits next to quad.satl |
| deleting a file; `file.new` over an existing file | `system.delete` gives S210; `file.new` answers ok = false | clear the file, then open it (M4) |
| errors a program can catch | "not planned"; `find()` on missing text stops the run (S420) | the loader checks with `contains()` first (M3) |
| `&&` and `\|\|` | refused; MISSING.md row 9 | nested ifs |
| one name declared twice in a capsule | S202, even in separate if blocks | declare temporaries once, at the top |
| making an object inside an expression | S110 | declare it on its own line |
| a base constructor that takes arguments | S110 | `piece` takes none; each class sets its own fields |
| command-line arguments | S201 (container help text) | `goal.text.c4` |
| questions about one character (is it a digit, a letter) | L2, "not planned" | `"abc…".contains(c)`, needed only for reading C++ (later) |

MISSING.md was written for the earlier QUAD and is partly out of date. These now work: including another .satl file, break and continue, and the string methods `.at`, `.substring`, `.split`, `.contains` and `.starts_with`.

## 10. Sketch of the core classes

This is a **sketch**, not quad's code. A longer copy of it, with the same classes plus `symbol`, `open_quote`, `close_quote` and `ascii`, and a `main` that builds three properties, ran on satl 004 rev 08 build 0103. It printed the three choices below and then the two M2 lines. `…` marks trimmed parts.

```
satellite.include(satellite)

// Every C++ object extends piece, so all of them answer the same capsules.
// piece takes no constructor arguments: satellite cannot extend one that does (S110).
satellite.spacesuit piece()
{
    satellite.protected
    {
        satellite.variable.string text = ""       // exactly what goes into the C++ file
        satellite.variable.string kind = "piece"  // which class it is, as a word
        satellite.variable.string before = " "    // the gap written in front of it
    }
    satellite.public
    {
        satellite.capsule call_text()
        {
            satellite.return(text)
        }
        satellite.capsule call_kind()
        {
            satellite.return(kind)
        }
        satellite.capsule call_ends_command()
        {
            satellite.return(satellite.bool.false)
        }
        // call_before, and call_takes (answers ""): the same shape
    }
}

// An ordinary object. symbol and header have the same shape; open_quote,
// close_quote and ascii also set before = "" where no gap is allowed.
satellite.spacesuit word(piece)
{
    satellite.constructor(satellite.variable.string text_input)
    {
        text = text_input
        kind = "word"
    }
}

// "\n" as ONE object: quote, backslash, n, quote.
satellite.spacesuit newline(piece)
{
    satellite.constructor()
    {
        text = "\"\\n\""
        kind = "newline"
    }
}

// THE SPECIAL CLASS: the only object that ends a command.
satellite.spacesuit semicolon(piece)
{
    satellite.constructor()
    {
        text = ";"
        kind = "semicolon"
        before = ""
    }
    satellite.public
    {
        satellite.capsule call_ends_command()
        {
            satellite.return(satellite.bool.true)
        }
    }
}

// A gap: takes whole properties whose PLACE is `takes`, least to most (0 = no limit).
satellite.spacesuit hole(piece)
{
    satellite.protected
    {
        satellite.variable.string takes = ""
        satellite.variable.number least = 1
        satellite.variable.number most = 1
    }
    satellite.constructor(satellite.variable.string takes_input, satellite.variable.number least_input, satellite.variable.number most_input)
    {
        takes = takes_input
        least = least_input
        most = most_input
        kind = "hole"
        text = "[" + takes_input + "]"
    }
    satellite.public
    {
        satellite.capsule call_takes()
        {
            satellite.return(takes)
        }
        // call_least, call_most: the same shape
    }
}

satellite.spacesuit property()
{
    satellite.protected
    {
        satellite.variable.string name = ""
        satellite.variable.string place = ""     // the kind of hole it may fill
        satellite.variable.string does = ""      // what it adds to the output (a list from M9)
        satellite.container.list<satellite.variable.string> needs = {}
        // YOUR DEFINITION: the objects in order, each position any one object type
        satellite.container.list<satellite.container.multiple<word, symbol, header, open_quote, ascii, close_quote, newline, semicolon, hole>> objects = {}
        // record, status, origin, made_of: added in M6 and M7
    }
    satellite.constructor(satellite.variable.string name_input, satellite.variable.string place_input, satellite.variable.string does_input)
    {
        name = name_input
        place = place_input
        does = does_input
    }
    satellite.public
    {
        // a piece parameter takes every object class
        satellite.capsule call_add(piece p)
        {
            objects.append(p)
        }
        satellite.capsule call_place()
        {
            satellite.return(place)
        }
        satellite.capsule call_does()
        {
            satellite.return(does)
        }
        satellite.capsule call_size()
        {
            satellite.return(objects.size)
        }
        satellite.capsule call_object(satellite.variable.number n)
        {
            satellite.return(objects[n])
        }
        // call_name, call_need: the same shape
    }
}

// Can a property that DOES `does` make the front of what is still `wanted`?
satellite.capsule fits(satellite.variable.string does, satellite.variable.string wanted)
{
    satellite.statement.if (wanted == "")
    {
        satellite.return(satellite.bool.false)
    }
    satellite.statement.if (does == "holes")
    {
        satellite.return(satellite.bool.true)
    }
    satellite.statement.if (does == "line_end")
    {
        satellite.return(wanted.starts_with("\n"))
    }
    satellite.statement.if (does == "ascii")
    {
        satellite.statement.if (wanted.starts_with("\n"))
        {
            satellite.return(satellite.bool.false)
        }
        satellite.return(satellite.bool.true)
    }
    satellite.return(satellite.bool.false)
}

// FLIPPING THROUGH: the first property whose PLACE is what the hole takes and
// whose DOES can make the front of what is wanted. 0 when none fits.
satellite.capsule choose(satellite.container.list<property> known, satellite.variable.string takes, satellite.variable.string wanted)
{
    satellite.variable.number n = 1
    satellite.statement.while (n <= known.size)
    {
        satellite.statement.if (known[n].call_place() == takes)
        {
            satellite.statement.if (fits(known[n].call_does(), wanted) == satellite.bool.true)
            {
                satellite.return(n)
            }
        }
        n = n + 1
    }
    satellite.return(0)
}

// In satellite.main, with the three cout properties in `known`:
//   choose(known, "command", "hi\n")   -> cxx.namespace.std.cout
//   choose(known, "cout_item", "hi\n") -> cxx.namespace.std.cout.string
//   choose(known, "cout_item", "\n")   -> cxx.namespace.std.cout.newline
// and for M2, the 8 objects appended to a list<multiple<...>> named `line`:
    satellite.variable.string joined = ""
    satellite.variable.number commands = 0
    satellite.variable.number i = 1
    satellite.statement.while (i <= line.size)
    {
        satellite.statement.if (i > 1)
        {
            joined = joined + line[i].call_before()
        }
        joined = joined + line[i].call_text()
        satellite.statement.if (line[i].call_ends_command() == satellite.bool.true)
        {
            commands = commands + 1
        }
        i = i + 1
    }
    satellite.console.display(joined)                    // std::cout << "hi" << "\n";
    satellite.console.display(line.size.string + " objects, " + commands.string + " ends a command")
```

## 11. Questions only you can answer

Each has a recommended answer.

1. Who runs g++ between turns? MISSING.md row 5 says running programs must not be papered over with a shell script. Recommended: type the two compile lines yourself through M5. Decide on a check.sh that decides nothing only once typing gets tiresome. Put satellite.system.run(command), which would answer the exit code and output, on satellite's own milestone list, because the unattended loop needs it.
2. Is a LIST of multiples an acceptable reading of 'held in a satellite.container.multiple<every object type>'? A multiple holds one value (std::variant), so each position is your multiple and the list gives the order. Recommended: yes, keep your spelling. satellite.container.list<piece> also works, but it loses the closed list of every object type.
3. Should the opening and closing quote be one class or two? Recommended: two classes (open_quote, close_quote), because different objects may follow each one. The text still shows as the same character.
4. The semicolon 'at the end of every command': is it fine that #include lines end at their line end, that } ends a block, and that the two ; inside for( ; ; ) will be a different class? Recommended: yes. The semicolon means 'ends a command', not 'ends a line'.
5. Should compiler warnings count as failure? Recommended: yes. Use g++ -std=c++17 -Wall -Wextra, and treat any message from g++ as FAIL, so quad never learns habits the compiler complains about.
6. What should the C++ file quad writes be called? PLAN says every quad file ends in .c4. Recommended: program.cxx.c4, compiled with g++ -x c++ (probed, works). The alternative is program.cpp, as a named exception to the .c4 rule.
7. How should properties quad invents be named? Recommended: extend your cxx. dotted tree, e.g. cxx.namespace.std.cout.string.newline, and mark who made each one with ORIGIN=quad, rather than a separate quad. prefix or numbered names.
