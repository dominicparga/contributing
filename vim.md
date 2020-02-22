# Vim

Vim is a way of navigating and editing text-files.
What makes vim special is its separation of __viewing__ text and __editing__ text.
For this, vim supports different modes with different key-usage.

- The __normal__-mode allows navigating through the text and executing handy commands, like "marking everything between brackets".
- The __insert__-mode allows adding new text.
- The __visual__ allows marking text, e.g. for deleting it.

There is a great tutorial for vim, which is probably already installed on your machine.
You can execute `vimtutor` in terminal and go.
Although the tutorial could be done in half an hour, it takes much longer to fully understand all commands and many commands are just not that important for most of the time.
This page provides a summary of the most important commands, which will cover more or less your whole daily vim-usage.

You can easily select vim in several IDEs, like [visual-studio-code][vscode] (and configure the hell out of it, which is not covered here).
With the overview of this page, it should be pretty easy to get into it. `:)`


## Table-of-contents

1. [Vim-inputs in general](#inputs-in-general)
1. [Normal-mode](#normal-mode)
    1. [Basic cursor-movement](#basic-cursor-movement)
    1. [Jumping words](#jumping-words)
1. [Insert-mode](#insert-mode)
1. [Visual-mode](#visual-mode)


## Vim-inputs in general <a name="inputs-in-general"></a>

Inputs can be distinguished in 2 categories:

- targetless, like `move cursor one character right`
- targeted, like `delete next word`

When targeted input, the input contains the command (like `d` for __delete__), then a cursor-movement (`e` = __end__ of word) or a target (`iw` = __word__ where cursor is currently __in__).


Further, most commands can be repeated automatically by adding a number before the command.
While `w` jumps a word (see below), input `3w` will jump 3 words.


### General commands <a name="general-commands"></a>

|      input     |  mode  | action |
|:--------------:|:------:|:------:|
| <kbd>esc</kbd> | normal, visual, insert | Go to normal-mode. |
| <kbd>:w</kbd> | normal | Save file to disk. |
| <kbd>:q</kbd> | normal | Exit file/vim, but ask user before exiting, if file hasn't been saved. |
| <kbd>:q!</kbd> | normal | Force exit, independent whether the file has been saved or not. |
| <kbd>:x</kbd> | normal | short for `:wq` |
| <kbd>u</kbd> | normal | Undo changes. |
| <kbd>strg</kbd> + <kbd>r</kbd> | normal | Undo undo. |


### Basic cursor-movement <a name="basic-cursor-movement"></a>

|    input     | example |  mode  | action |
|:------------:|:--------|:------:|:------:|
| <kbd>h</kbd> | `li|ne 1` -> `l|ine 1` | normal | Move cursor left. |
| <kbd>l</kbd> | `li|ne 1` -> `lin|e 1` | normal | Move cursor right. |
| <kbd>j</kbd> | `li|ne 1` -> `li|ne 2` | normal | Move cursor down. |
| <kbd>k</kbd> | `li|ne 1` -> `li|ne 0` | normal | Move cursor up. |
| <kbd>0</kbd> | `Some text |in a line.` -> `|Some text in a line.` | normal | Move cursor to the start of line. |
| <kbd>$</kbd> | `Some text |in a line.` -> `Some text in a line.|` | normal | Move cursor to the end of line. |
| <kbd>strg</kbd> + <kbd>u</kbd> | normal | Let cursor jump several lines up. |
| <kbd>strg</kbd> + <kbd>d</kbd> | normal | Let cursor jump several lines down. |

The current view can be changed.

|    input     |  mode  | action |
|:------------:|:------:|:------:|
| <kbd>strg</kbd> + <kbd>e</kbd> | normal | Let view move one line up. |
| <kbd>strg</kbd> + <kbd>y</kbd> | normal | Let view move one line down. |


### Copy-and-paste <a name="copy-and-paste"></a>

|      input     |  mode  | action |
|:--------------:|:------:|:------:|
| <kbd>y</kbd> | visual | Copy marked text. |
| <kbd>yy</kbd> | normal | Copy the cursor's current line. |
| <kbd>p</kbd> | normal, visual | Paste copied text at cursor. If some text is marked, it will be removed! If some text is marked, it will be removed! |
| <kbd>P</kbd> | normal, visual | Paste copied text after cursor's current character. If some text is marked, it will be removed! |

Note: Keep in mind that deleting text does store the deleted text in th clipbard.


### Jumping words <a name="jumping-words"></a>

|    input     | action | example |
|:------------:|:-------|:--------|
| <kbd>w</kbd> | Jump to the next first character of a word. | `he|llo, vim` -> `hello, |vim` |
| <kbd>W</kbd> | Jump to the next first character after a space. | `he|llo, vim` -> `hello, |vim` |
| <kbd>e</kbd> | Jump to the next last character of a word. | `he|llo, vim` -> `hell|o, vim` |
| <kbd>E</kbd> | Jump to the next last character before a space. | `he|llo, vim` -> `hello|, vim` |
| <kbd>b</kbd> | Jump to the previous first character of a word. | `hello, |vim` -> `hello|, vim` |
| <kbd>B</kbd> | Jump to the previous first character before a space. | `hello, |vim` -> `|hello, vim` |


### Jumping by search <a name="jumping-by-search"></a>

|    input     | action | example |
|:------------:|:-------|:--------|
| <kbd>f</kbd> + <kbd>any character (e.g. `i`)</kbd> | Jump to the next occurence of the provided character (inclusive). | `he|llo, vim` -> `hello, v|im` |
| <kbd>t</kbd> + <kbd>any character (e.g. `i`)</kbd> | Jump to the next occurence of the provided character (exclusive). | `he|llo, vim` -> `hello, |vim` |
| <kbd>F</kbd> + <kbd>any character (e.g. `l`)</kbd> | Jump to the previous occurence of the provided character (inclusive). | `hello, v|im` -> `hel|lo, vim` |
| <kbd>T</kbd> + <kbd>any character (e.g. `l`)</kbd> | Jump to the previous occurence of the provided character (exclusive). | `hello, v|im` -> `hell|o, vim` |


### Jumping lines <a name="jumping-lines"></a>

|    input     | action | example |
|:------------:|:-------|:--------|
| <kbd>42gg</kbd> | Jump to start of line 42. | `lin|e 5102` -> `|line 42` |
| <kbd>gg</kbd> | Jump to start of line 0. | `lin|e 5102` -> `|line 0` |
| <kbd>G</kbd> | Jump to start of last line. | `lin|e 5102` -> `|last line` |


### Changing mode <a name="changing-mode"></a>

|    input     | example |  mode  | action |
|:------------:|:--------|:------:|:------|
| <kbd>i</kbd> | `hell|o, vim` -> `hell|o, vim` | normal -> insert | Go to insert-mode in front of current character. |
| <kbd>I</kbd> | `hell|o, vim` -> `|hello, vim` | normal -> insert | Go to insert-mode in front of current line. |
| <kbd>a</kbd> | `hell|o, vim` -> `hello|, vim` | normal -> insert | Go to insert-mode behind current character. |
| <kbd>A</kbd> | `hell|o, vim` -> `hello, vim|` | normal -> insert | short for `$a` |
| <kbd>s</kbd> | `hell|o, vim` -> `hell|, vim` | normal -> insert | Remove cursor-character and go to insert-mode at this character's position. |
| <kbd>S</kbd> | `hell|o, vim` -> `|` | normal -> insert | Remove line's content (without the line-break) and go to insert-mode. |
| <kbd>o</kbd> | `hell|o, vim` -> `hell|o, vim \n |` | normal -> insert | Add a new line __below__ the cursor's current line, go to this line and enter insert-mode. |
| <kbd>O</kbd> | `hell|o, vim` -> `hell|o, vim \n |` | normal -> insert | Add a new line __above__ the cursor's current line, go to this line and enter insert-mode. |
| <kbd>v</kbd> | `hell|o, vim` -> `hell|o, vim` | normal -> visual | Go to visual-mode. After entering the visual-mode, you can move the cursor around and delete or copy this marked text to get back to normal-mode. |


### Targets <a name="targets"></a>

For examples, see their usage when [deleting text](#deleting-text) or [marking text](#marking-text).

|     input     | meaning |
|:-------------:|:--------|
| <kbd>i</kbd> + <kbd>some character</kbd> | Inner text between characters like brackets or quotations. |
| <kbd>a</kbd> + <kbd>some character</kbd> | Inner text between characters like brackets or quotations in addition to the specified characters. |


### Deleting text without specified target <a name="deleting-text-without-target></a>

|    input      | example |  mode  | action |
|:-------------:|:--------|:------:|:------|
| <kbd>dd</kbd> | `line0 \n li|ne1 \n line2` -> `line0 \n |line2` | normal | Cut out the cursor's current line. |
| <kbd>ddp</kbd> | `line0 \n li|ne1 \n line2` -> `line0 \n line2 \n |line1` | normal | Handy command-order for swapping lines. |
| <kbd>D</kbd> | `line0 \n li|ne1 \n line2` -> `line0 \n li| \n line2` | normal | Cut out the cursor's current line, starting from the cursor. |
| <kbd>C</kbd> | `line0 \n li|ne1 \n line2` -> `line0 \n li| \n line2` | normal | short for `Da` |
| <kbd>x</kbd> | `line0 \n li|ne1 \n line2` -> `line0 \n li|e1 \n line2` | normal | Cut out the cursor's current character. |


### Deleting text with target <a name="deleting-text-with-target"></a>

Usually, deletions are initiated with `d`, followed by a cursor-movement-command (see above) or a target (see above).
You can delete text with `c` entering insert-mode afterwards.
In the following, some examples are given, working for both `d` and `c`.

|    input      | example |  mode  | action |
|:-------------:|:--------|:------:|:------|
| <kbd>de</kbd> | `he|llo, vim` -> `he|, vim` | normal | Delete to the next last character of a word (inclusive). |
| <kbd>dt</kbd> + <kbd>any character</kbd> | `he|llo, vim` -> `he|, vim` | normal | Delete to the next occurence of the provided character (exclusive). |
| <kbd>df</kbd> + <kbd>any character</kbd> | `he|llo, vim` -> `he| vim` | normal | Delete to the next occurence of the provided character (inclusive). |
| ... | ... | ... | ... |
| <kbd>diw</kbd> | `he|llo, vim` -> `|, vim` | normal | Delete the current word to the next first character of a word. |
| <kbd>diW</kbd> | `he|llo, vim` -> `| vim` | normal | Delete the current word to the next first character after a space (but keep the spaces). |
| ... | ... | ... | ... |
| <kbd>di</kbd> + <kbd>some character</kbd> | `hello, "v|im"` -> `hello, "|"` | normal | Delete all characters inside the provided character. Is the character a bracket, delete all characters inside these brackets. |
| <kbd>da</kbd> + <kbd>some character</kbd> | `hello, "v|im"` -> `hello, |` | normal | Delete all characters inside the provided character and the surrounding characters in addition. Is the character a bracket, delete all characters inside these brackets __and__ the brackets. |
| ... | ... | ... | ... |


### Replacing text <a name="replacing-text"></a>

|    input      | example |  mode  | action |
|:-------------:|:--------|:------:|:------|
| <kbd>r</kbd> + <kbd>any character</kbd> | `he|llo, vim` -> `he|klo, vim` | normal, visual | Replace cursor's character by the provided character. If multiple characters are marked, replaces every marked character by the provided one. |
| <kbd>R</kbd> + <kbd>any character</kbd> | `he|llo, vim` -> `hekkkkk|vim` | normal | Enter kind of a replace-mode, overwriting current text. |


### Others

|    input      | example |  mode  | action |
|:-------------:|:--------|:------:|:------|
| <kbd>.</kbd> | `he|llo, vim` -> `he|lo, vim` -> `he|o, vim` | normal | Repeats the last executed command. |
| <kbd>strg</kbd> + <kbd>a</kbd> | `1` -> `2` | normal | Increments a number under the cursor. |
| <kbd>strg</kbd> + <kbd>x</kbd> | `2` -> `1` | normal | Decrements a number under the cursor. |


[vscode]: https://code.visualstudio.com/
