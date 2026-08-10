# Ogol

**Onboard General Operations Language** — pronounced *OH-gol*, which is Logo backwards, whose syntax
it borrows.

A small interactive language written in [sysl](https://sysl.sh), meant to live in a microcontroller's
flash and be talked to over a serial line: type a line, it runs, you see what happened. This
repository is the **language** and nothing else. It reads text and answers with text, and knows
nothing about a board, a register or a terminal — what drives it is a separate program.

```
ogol> print 2 + 3 * 4
14
ogol> set count 0
ogol> repeat 3 [ set count count + 1 print count ]
1
2
3
ogol> to double :n output n * 2 end
ogol> print double count
6
```

## What it is, in one paragraph

Logo's grammar with the brackets left off: every name has a known number of arguments, so
`print double count` reads without a comma or a parenthesis anywhere. Seven names do the work —
`print`, `set`, `repeat`, `if`, `ifelse`, `stop`, `output` — with eleven more for lists, the five
arithmetic and six comparison operators, and `to … end` for everything else. Values are numbers,
words, booleans and lists.

## Lists

`[a b c]` is a list, and its elements are **not evaluated** — which is what lets a list hold words
that name nothing, and what lets `repeat 3 [ print 1 ]` hand three unevaluated words to something
that decides what they mean. A block is not a separate kind of thing: it is a list that `repeat`,
`if` or `ifelse` ran.

```
ogol> print [hello there]
hello there
ogol> print [a [b c] d]
a [b c] d
ogol> set xs [1 2 3]
ogol> print first xs
1
ogol> print butfirst xs
2 3
ogol> print fput 0 xs
0 1 2 3
```

A list prints without its own brackets and a nested one keeps them: `print` is asking for the
contents, and inside a list the brackets are the only thing saying where an element ends.

| | |
|---|---|
| `first` `last` | one end or the other |
| `butfirst` `bf` `butlast` `bl` | everything but that end — this is how a list is walked |
| `item n` | the nth, **counting from 1** |
| `fput` `lput` | a new list with one more on the front or the back |
| `list a b` | the two of them as a list |
| `emptyp` `listp` | whether it is empty, and whether it is a list at all |

**Every one of these reads a word as well as a list**, because a word is a sequence of characters
exactly as a list is a sequence of elements: `first "hello` is `h` and `butfirst "hello` is `ello`.
That is Logo's rule and it halves the vocabulary without costing any clarity.

A number is not a sequence here. Logo would read `first 123` as `1` by treating the number as its own
text; Ogol keeps numbers and words apart on purpose, so that is a mistake and says so.

**There is no `count`, and its absence is deliberate.** Logo's name for a list's length collides with
one of the most natural variable names there is — the first example in this file is `set count 0` —
and dropping Logo's sigils means a built-in name cannot also be a variable. `emptyp` answers the
question people actually ask of a list; how to spell the length is a decision still to be taken.

## What is different from Logo, and why

**No sigils.** Logo writes `make "x 5` to set a name and `:x` to read it. Ogol writes `set x 5` and
`x`. A bare name is a call, and a name nobody has heard of is a call of no arguments — which is what
a variable is. That one rule replaces both marks.

**A colon survives in exactly one place**: the title line of a definition. `to double :n output n * 2
end` has to be readable in a single pass, and with a bare `n` there would be nothing to say where the
parameters stop and the body starts.

**A line is a hard boundary.** Logo lets a command reach across as many lines as its arguments need,
which is why a Logo console can sit waiting for a word that is never coming and look, from the
outside, exactly like a board that has crashed. Here a statement ends at the end of its line. The
only two things that may span one are a bracketed block and a definition, and both are visible in the
text — so a console can tell *unfinished* from *wrong*, and say which.

**A minus sign is read by the space around it.** `count -1` is two things and `count - 1` is one;
`n-1` is one, because that is how everybody writes it, and `n- y` is refused as having no plausible
reading. The safety net is arity: the two readings always differ in argument count by exactly one, so
the wrong one is refused rather than quietly answered.

```
ogol> set level 3
ogol> set level level -1
'set' needs 2 arguments, got 3 arguments -- '-1' reads as a negative number, and '- 1' as a subtraction
```

**A procedure sees its own parameters and the globals, and nothing of whoever called it.** Logo's
scope is dynamic; this is the departure that makes a procedure readable on its own.

**Numbers, words and booleans are three kinds rather than one.** Logo has the word and reads it as a
number where it needs one. A language whose purpose is to write to a register is better off deciding
that at the point of the mistake.

## Using it

```
dependencies {
  ogol { git = "github.com/sysl-lang/ogol", version = "0.1.0" }
}
```

```sysl
import sh.sysl.ogol.*

var i = ogol()

run(i, "print 6 * 7") match
    Ok(_) -> ()
    Err(f) -> print(describe(f))
```

An interpreter is a value and carries the session: procedures and variables set by one call to `run`
are there for the next. A `Fault` is either `Bad(message)` — a refusal, with the sentence to print —
or `More`, which means the text stopped in the middle of a block or a definition and a console should
ask for another line rather than complain.

`requires { alloc = true }`. The board this is aimed at links a heap whether or not a program touches
it, so the capability costs nothing already spent.

## Tests

```
sysl test .
```

Fifty of them, and all of them are written through the transcript — what a person typing would see —
rather than through the syntax tree, because the transcript is what is promised.

## Where it is going

The register model: a table generated from the vendor's SVD and linked into flash, so that a field is
named rather than masked and a read comes back decoded. Then flash-backed definitions that survive a
power cycle. Neither is here yet.

## Licence

ISC — see `LICENSE`.
