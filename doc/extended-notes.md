---
author:
- massifrg at gmail dot com
date: 'March 16, 2026'
title: Extended notes
---

# Introduction

In Pandoc types version 1.23.1, notes can't have any extra information but
their contents. That's because the `Note` `Inline` is made of `[Block]`,
a list of blocks that represent its contents.

No identifier, classes or attributes, that you usually find in `Inlines` and
`Blocks` that have an `Attr`.

Supporting [multiple types of notes](https://github.com/jgm/pandoc/issues/1425)
is difficult at least, and also referencing the same note more than once has
its [issues](https://github.com/jgm/pandoc/issues/1603).

This document describe some extensions that add support for

- multiple references to the same note

- endnotes

These extra features are enabled by **extensions that are disabled by default**,
so Pandoc behaves as usual if they are not enabled.

If you notice a different behavior when those extensions are disabled, please
let me know.

## Adding data to notes

Those problems could be mitigated if only the `Note` inline carried some data,
something like `Note Attr [Block]`, `Note (Maybe Text) [Block]` (optional
identifier), etc.
But changing the Pandoc types may disrupt some existing workflows, and it would
require many fixes to the code of readers and writers.

In the case of custom styles, in particular paragraph styles, a convention lets
you give a style to a `Para`, even if it has nothing but its inline contents,
just by surrounding it with a `Div` that has a `custom-style` attribute set to
the name of a certain style.

In analogy with the convention on custom styles, we can embed a `Note` in a
`Span` inline that carries the data that the `Note` can't have.
Here's a couple of examples in Markdown, that anticipate what will be shown
below:

```markdown
This text has a footnote[[^1]]{noteref="note1"} that is referenced
twice[[^1]]{noteref="note1"}, but also an endnote[[^2]]{.endnote}.

[^1]: Footnote text.

[^2]: Endnote text.
```

In the case of the footnote, the `Span` around it carries a `noteref`
attribute, with a value that can be used to link its two references in
the text.

The endnote, instead, has an `endnote` class, that can be used to treat it
differently in writers or filters, whose output format supports a distinction
between footnotes and endnotes.

That data is impacting the readability of the Markdown source, but there are
ways to make that extra information invisible but still present, see below.

## Multiple references to the same note

In version 3.9 of Pandoc, if you reference a note twice, you won't get any
error, but it will result in a duplication of the note's text:

```sh
pandoc -f markdown -t markdown
```

```markdown
This text has a footnote[^1] that is referenced twice[^1],
and another footnote[^2] referenced only once.

[^1]: Double-reference footnote.

[^2]: Single-reference footnote.
```

results in:

```markdown
This text has a footnote[^1] that is referenced twice[^2], and another
footnote[^3] referenced only once.

[^1]: Double-reference footnote.

[^2]: Double-reference footnote.

[^3]: Single-reference footnote.
```

as if there were 3 distinct notes, instead of 2.

### Extension `multiref_notes`

If you enable the `multiref_notes` extension on the reader,

```sh
pandoc -f markdown+multiref_notes -t markdown
```

you get:

```markdown
This text has a footnote[[^1]]{noteref="1"} that is referenced
twice[[^2]]{noteref="1"}, and another footnote[^3] referenced only once.

[^1]: Double-reference footnote.

[^2]: Double-reference footnote.

[^3]: Single-reference footnote.
```

Same result as above, but the two references of note 1 are now embedded
in a `Span` with a `noteref` attribute equal to "1".

Now enable the same extension on the writer too:

```sh
pandoc -f markdown+multiref_notes -t markdown+multiref_notes
```

and you get:

```markdown
This text has a footnote[^1] that is referenced twice[^1], and another
footnote[^2] referenced only once.

[^1]: Double-reference footnote.

[^2]: Single-reference footnote.
```

which is exactly the original document

- without any text duplication of the notes' text

- without any modification to the document source

- with a modification of the intermediate AST, where multi-referenced
  notes are embedded in a `Span`

### Extension `keep_noterefs`

Now consider a modified version of the example:

```markdown
This text has a footnote[^double] that is referenced twice[^double],
and another footnote[^single] referenced only once.

[^double]: Double-reference footnote.

[^single]: Single-reference footnote.
```

And convert it with:

```sh
pandoc -f markdown+multiref_notes -t markdown+multiref_notes
```

The references and notes' texts are still OK, but notes are renumbered:

```markdown
This text has a footnote[^1] that is referenced twice[^1], and another
footnote[^2] referenced only once.

[^1]: Double-reference footnote.

[^2]: Single-reference footnote.
```

You can use the `keep_noterefs` to keep the references as they were in the
original document:

```sh
pandoc -f markdown+multiref_notes+keep_noterefs -t markdown+multiref_notes+keep_noterefs
```

which gives back the original document:

```markdown
This text has a footnote[^double] that is referenced twice[^double], and
another footnote[^single] referenced only once.

[^double]: Double-reference footnote.

[^single]: Single-reference footnote.
```

In this case the `multiref_notes` is not necessary, and you would get the
same result with:

```sh
pandoc -f markdown+keep_noterefs -t markdown+keep_noterefs
```

That's because the `multiref_notes` adds the `Span` with the `noteref`
attribute only to notes that are referenced more than once, while the
`keep_noterefs` does the same for *every* note.

If you make a conversion with the command above, with `keep_noterefs`
both on the reader and the writer, you'll get a way to sort the notes
of your document, while retaining the reference names you decided.

## Endnotes

A new `endnotes` extension lets you discriminate between footnotes and
endnotes, which is useful when you work with formats that support that
distinction.

In the terms of the Pandoc AST, an endnote is just a `Note` embedded in
a `Span` of class `endnote` (always the same trick).

### Supported formats

Currently endnotes are supported on DOCX, ODT and Markdown formats, both
in the reader and in the writer.

Here's an example:

```markdown
This text has an endnote[[^1]]{.endnote} and a footnote[^2].

[^1]: Endnote text.

[^2]: Footnote text.
```

If you convert it to DOCX or ODT with no extension enabled, you'll get
a document with two footnotes.

With the following commands, you'll get a document with a footnote and
an endnote, instead:

```sh
pandoc -f markdown -t docx+endnotes -o test.docx test.md
pandoc -f markdown -t odt+endnotes -o test.odt test.md
```

With a round-trip conversion:

```sh
pandoc -f markdown -t docx+endnotes -o - test.md | pandoc -f docx+endnotes -t markdown
```

you'll get back the original document, with a `Span` of class `endnote` around the
endnote, as expected.

### Endnotes support in Markdown

Those `Span` inlines around the endnotes' references impact markdown's readability.

A convention on references, while enabling the `endnotes` extension, lets you write
the previous document this way:

```markdown
This text has an endnote[^EN1] and a footnote[^1].

[^EN1]: Endnote text.

[^1]: Footnote text.
```

When you convert it to Markdown without extensions,

```sh
pandoc -f markdown+endnotes -t markdown
```

you'll get:

```markdown
This text has an endnote[[^1]]{.endnote} and a footnote[^2].

[^1]: Endnote text.

[^2]: Footnote text.
```

As a result, there's the endnote `Span` around the first note.
That's because the "EN" prefix makes the Markdown reader (with
`endnotes` enabled) consider the first note as an endnote.


If you enable the `endnotes` extension on the writer too,

```sh
pandoc -f markdown+endnotes -t markdown+endnotes
```

you'll get:

```markdown
This text has an endnote[^EN1] and a footnote[^1].

[^1]: Footnote text.

[^EN1]: Endnote text.
```

Please note that another effect of the extension on the writer is that
the *endnote(s) is (are) listed after the footnote(s)*.

### Prefixes

The "EN" prefix for endnotes is just a default value, that you can
modify with the `--endnotes-prefix` option.

```sh
pandoc --endnotes-prefix=endnote -f markdown+endnotes -t markdown+endnotes
```

When --id-prefix is also specified, the endnote prefix will come first
(this is to make endnotes still recognizable).

```sh
pandoc --id-prefix=prefix -f markdown+endnotes -t markdown+endnotes
```

After converting the previous example, you get:

```markdown
This text has an endnote[^ENprefix1] and a footnote[^prefix1].

[^prefix1]: Footnote text.

[^ENprefix1]: Endnote text.
```

### Multiple references and endnotes together

`multiref_notes`, `keep_noterefs` and `endnotes` can work together.
Here's an example:

```markdown
This document has both endnotes[^EN1] and footnotes[^fn1].

Footnotes[^fn1] are listed before endnotes[^EN1].

Here's another footnote[^fn2] and another endnote[^EN2].

[^EN1]: Yes, there are endnotes in this document.

[^fn1]: An example footnote, that is referenced twice.

[^fn2]: Another footnote.

[^EN2]: Another endnote.
```

Convert it with:

```sh
pandoc -f markdown+endnotes+multiref_notes -t markdown+endnotes+multiref_notes
```

and you'll get this:

```markdown
This document has both endnotes[^EN1] and footnotes[^1].

Footnotes[^1] are listed before endnotes[^EN1].

Here's another footnote[^2] and another endnote[^EN2].

[^1]: An example footnote, that is referenced twice.

[^2]: Another footnote.

[^EN1]: Yes, there are endnotes in this document.

[^EN2]: Another endnote.
```

If you convert it with:

```sh
pandoc -f markdown+endnotes+keep_noterefs -t markdown+endnotes+keep_noterefs
```

you'll get:

```markdown
This document has both endnotes[^EN1] and footnotes[^fn1].

Footnotes[^fn1] are listed before endnotes[^EN1].

Here's another footnote[^fn2] and another endnote[^EN2].

[^fn1]: An example footnote, that is referenced twice.

[^fn2]: Another footnote.

[^EN1]: Yes, there are endnotes in this document.

[^EN2]: Another endnote.
```

which has the same references used in the original document, but the notes'
texts are in the right order.
