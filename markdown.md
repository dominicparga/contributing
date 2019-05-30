# Markdown

If you stuck with syntax (e.g. how to include images?), checkout this [cheatsheet][www_md_cheatsheet].
Below comes a table of contents.

---

## Table of Contents <a name="toc"></a>

1. [Table of Contents](#toc)
1. [Links](#links)
1. [Super Duper Hardcore Mega Fancy Title](#fancy-title)
    1. [Code styles](#code-styles)
    1. [Style mentions](#style-mentions)

This TOC's source code is shown below.
Please notice that every item starts with number 1.
Also cool is the inline html tag, that allows title names being independent of the tag for the TOC.

Typically, the table of contents doesn't appear in the TOC, but here is content referred to the chapter.

```markdown
1. [Table of Contents](#toc)
1. [Links](#links)
1. [Super Duper Hardcore Mega Fancy Title](#fancy-title)
    1. [Code styles](#code-styles)
    1. [Style mentions](#style-mentions)

...

---

## Super Duper Mega Fancy Title <a name="fancy-title"></a>

...
```

---

## Links <a name="links"></a>

Use links [directly](https://github.com/dominicparga) or with a [reference][www_github_dominicparga] in the end of the file.

I prefer always using references for consistency.
Prefixing link-references with __www__ could help with readability, e.g. `www_github_dominicparga` referring to [me :3][www_github_dominicparga].

```markdown
Use links [directly](https://github.com/dominicparga) or with a [reference][www_github_dominicparga] in the end of the file.

...

[www_github_dominicparga]: https://github.com/dominicparga
```

---

## Super Duper Mega Fancy Stuff <a name="fancy-title"></a>

Writing markdown is a lot easier, especially if you are vim-user, if every sentence has its own line without any linewidth limit.

```diff
+ This is a sentence.
+ This is a second sentence written in a separate line. :)

- This is a bad example. This is another sentence in the same line.
```

> __Note__: Here stands a note.
> `diff` is a quite useful code-style.
> As you see here, even notes can follow the multiline rule.
> Notes' lines start with `>`.

### Code-styles <a name="code-styles"></a>

Every code block should define a style.
Some examples (given in a code block):

```text
- text
- python
- bash
- zsh
- java
- rust
- diff
- markdown
```

Highlighted monospace text can be written like `this`.

### Style mentions <a name="style-mentions"></a>

You can look at this files raw version to see the written style, but in addition:

- Never add two __headings__ right after each other, but add at least one sentence in between.
  Just looks more professional.
- Add __sub-headings__ only if *__more__* than one sub-heading is added.
  Otherwise, sub-headings make no sense.
- It helps using `__bold__` for __bold__ text and `*italic*` for *italic* text.
  Mixing them could be done like `*__bold and italic__*`, giving *__bold and italic__*, but you should be consistent.
- Horizontal lines (add `---` to an empty line) could help with readability.

[www_md_cheatsheet]: https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet
[www_github_dominicparga]: https://github.com/dominicparga
