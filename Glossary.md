
### Assembler

An `Assembler` is a program that takes specially-written text files as an input and converts them into binary data. These text files allow us to write out changes in a way that are easier to read and edit than the binary data they turn into.

#### 64tass

([main page](https://sourceforge.net/projects/tass64/))([manual](http://tass64.sourceforge.net/))

`64tass` is an [assembler](#assembler) written for the 65xx family of processors. The SNES uses a version of the 65C816 chip. 64tass is my preferred assembler for SNES-related things due to its rich set of features.

#### Event Assembler

[Nintenlord's `E`vent `A`ssembler](https://feuniverse.us/t/event-assembler/1749) is an [assembler](#assembler) that is used for GBA(FE) hacking. EA has fewer features than [64tass](#64tass) and is set up exclusively for the GBA's memory map, so we don't use it.

### Buildfile (file)

See: [Build Script](#build-script)

### Buildfile (method)

The `Buildfile Method` is a project organizational structure that uses a [build script](#build-script) and an [assembler](#assembler) to make changes to a ROM, rather than making edits to a ROM over time with tools.

### Build Script

This is the main file of the [Buildfile Method](#buildfile-method). It is the main input for the [assembler](#assembler).

The `Build Script` is a text file written in the [assembler](#assembler)'s syntax that manages major file [inclusions](#include-directive) and ROM space used by components.

### Directive

A `Directive` is a special kind of command for the [assembler](#assembler) that has some kind of effect that couldn't be mimicked without special behavior, like [including](#include-directive) other files.

#### Include (directive)

Instead of putting everything into one file, parts of the project are split up across multiple files, often across multiple folders. To tell the [assembler](#assembler) to read one of these files, we use the `include` [directive](#directive), which acts as if all of the file was where the `directive` was.

[64tass](#64tass)-specific ([manual section](http://tass64.sourceforge.net/#including)): `64tass` offers multiple variants of the `include` directive:

##### .include

([manual section](http://tass64.sourceforge.net/#d_include))

`.include <filename>`
`.include "relative/path/to/file.asm"`

This is for text files written in the assembler syntax, and acts as if the text in the file was where the `.include` line is.

##### .binclude

([manual section](http://tass64.sourceforge.net/#d_binclude))

`[<label>] .binclude <filename>`
`someLabel .binclude "relative/path/to/file.asm"`

This is similar to [.include](#include) but it either places everything into some [label](#label)'s [scope](#scope) if preceded by a label or into an unreachable scope if left bare. You can access parts of it using the `.` operator, like `someLabel.someDefinition`.

##### .binary

([manual section](http://tass64.sourceforge.net/#d_binary))

`.binary <filename>[, <offset>[, <length>]]`
`.binary "relative/path/to/file.bin"`
`.binary "relative/path/to/file.bin", 42`
`.binary "relative/path/to/file.bin", 42, 1337`

This is similar to [.include](#include) but it inserts the contents of the file as bytes into the output.

If specified, `<offset>` is a number that dictates how many bytes into the file to start reading from. A negative number means to begin reading some number of bytes from the end of the file.

If specified, `<length>` is a number that dictates at most how many bytes are copied into the output.

### Scope

([manual section](http://tass64.sourceforge.net/#symbols))

`Symbols` are how we refer to locations and values in the [assembler's](#assembler) syntax. Symbols get their values through assignment or by being used as a [label](#label). Example:

```
AssignedSymbol := 1234 ; Here, `AssignedSymbol` is given the value `1234`

LabelSymbol ; Without any assignment operators, the symbol is a label.
```

Other things can then use these symbols instead of their values, such as

```
ReallySpecificNumber = 42

GetReallySpecificNumber

  lda #ReallySpecificNumber
  rtl
```

The assembler maintains a hierarchy where symbol names are valid called a `scope`. For example:

```
SomeScope .block

  SomeLabel

.endblock

.word SomeLabel ; This will error, because `SomeLabel` is in the scope `SomeScope`
                ; and not the global scope.

.word SomeScope.SomeLabel ; This is how you'd refer to `SomeLabel`.

```

Scopes are useful because they allow you to organize things and reuse common names:

```
FirstFunction

  _Loop
  dec x
  bpl _Loop

  rtl

SecondFunction

  _Loop
  inc x
  bpl _Loop

  rtl
```

#### Label

([manual section](http://tass64.sourceforge.net/#symbols))

A `label` is how we give a name to some address whose precise location we don't want to calculate ourselves. For example:

```
* := 1000

FirstFunction ; `FirstFunction` is a label

  lda #1
  rtl

SectionFunction ; `SecondFunction` is a label

  lda #2
  rtl
```

We could figure out `SecondFunction`'s location ourselves, but that's a lot of work and it would change if we messed with anything that came before it.
