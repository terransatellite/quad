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

**Open:** what each property holds.

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

### M2 — the property class

Not started. It waits on the answer to "what does each property hold".
