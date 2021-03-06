# verso|recto - A Different Approach to Literate Programming

> Literate programming is the art of preparing programs for human readers.

_Norman Ramsey, on the [noweb homepage](https://www.cs.tufts.edu/~nr/noweb/)._

Literate Programming (LP) tries to address a common problem with software systems: by reading the
code one can discover _how_ a thing is done, but not _why_ it is done that way. Every program has a
theory of operation embedded within its logic, but this is often hidden. Comments within the code
and documentation of APIs are helpful but insufficient for this task. Often the documentation
generation systems provided with a programming language do not provide a way to contextualize the
code they describe within the larger system. For programs which rely on a more advanced mathematical
background, it is also often difficult to embed equations or other markup in the documentation.

Existing LP tools such as WEB, noweb, and Org Babel attempt to address this by taking the following
approach:

1. Embed the program source code within a prose document which explains the author's thinking.
2. Provide mechanisms for abstracting code into abstract chunks, which can be recombined in
   different orders than they are defined and referenced throughought the document.
3. Process the combined document in one of two ways: either `tangle` the source code out of the
   document into a computer-friendly version, or `weave` the document into a form ready for humans
   to read.

Overall this is an improvement on inline documentation and can provide much more context to the
reader than mainstream approaches:

- The source code is embedded within a document, making it inaccessible to language-specific tooling
  such as editors, compilers, and static analysis tools. In order to use these the code must first
  be tangled.
- Similarly, to build your literate program the end user must have the appropriate tools installed
  in addition to the language tooling needed to compile the source.
- Most programmers have spent years working directly in the machine-friendly source representation
  rather than a literate style, introducing a barrier to entry to writing literate programs. Porting
  an existing project to a literate representation is nontrivial.

`verso|recto` takes a different approach. It considers the source file as a first-class citizen to
be referenced by the documentation. Rather than embedding source code in documents, or documents in
source code, lightweight annotations are used to mark sections of interest within the code which can
be easily referenced by prose documents. There is no `tangle` step, and source files remain fully
valid inputs for compilers, editors, and other tools. There is no need to maintain line numbers or
formatting between literate sources and what the compiler sees.

## What traditional LP systems get right

A lot of things! Here are a few ideas that the traditional systems pioneered and `verso|recto`
borrows:

- Support a many-to-many relationship between LP documents and source files.
  - In `noweb`, a document may contain several root chunks which can each be sent to different
    source files, and multiple LP documents can be fed into the `tangle` tool at once (they are
    concatenated).
- The more modern tools support multiple programming and markup languages.
- `noweb` offers a pipelined architecture which makes it easy to insert processing stages to meet a
  user's needs.
- They generate indices and cross references within the human-readable output.

## Using `verso|recto`

`verso|recto` is driven by annotations within your source files, defining regions which can be
referenced by other documents.

### Annotating a file

Annotations are quite simple. To mark a region of code for reference, simply add a pair of comments
around the region with the symbols `@!` and `!@`, followed by a unique ID. The ID can be any string
of non-whitespace characters, though it should be both unique within your project and valid in the
source file you're annotating.

### Referencing annotations

In order to reference an annotation in another file, add a line containing the symbol `@@` followed
by the ID of the annotation (e.g. `@@12345`). When the file is woven using the `recto` command (see
the next section), the line will be replaced with the contents of the annotation. You can add any
markup you like around the line to provide formatting.

### Weaving a document for human consumption

The `verso` command will read all of the annotations from the files specified on the command line,
extract their annotations, and output the result to stdout. In turn the `recto` command will read
its annotations from stdin. This makes the two programs easy to use together via pipes:

```
verso main.rs lib.rs | recto build chap1.tex chap2.tex blog/home.md
      ^       ^              ^     ^         ^         ^
      +-------+              |     +---------+---------+
      |                      |                         |
      |                      |                         |
      +--- Source files      +--- Output directory     +--- Prose files
```

Each of the woven files is written to the output directory, provided as the first argument, in the
same relative location as given on the command line. So, for example, the file `blog/home.md` above
will be written to `build/blog/home.md` when it is woven.

## The Name

> Recto and verso are respectively, the text written or printed on the "right" or "front" side and
> on the "back" side of a leaf of paper in a bound item such as a codex, book, broadsheet, or
> pamphlet.

_[Wikipedia - Recto and Verso](https://en.wikipedia.org/wiki/Recto_and_verso)_

_(Note that, conveniently, the `verso` program goes on the left of the pipe while `recto` goes on
the right.)_

## Future Work

- Add support for allow overlapping annotations.
- Add support for custom formatting of annotation properties within the woven output.
- Paralellize file processing in Verso, and both reading from `stdin` and file reading in Recto.
- Add `--annotations-from` option to specify a source other than stdin for annotations.
