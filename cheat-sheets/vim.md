# vim cheat sheet

A highly opinionated vim cheat sheet.

## Copy / Cut / Paste (normal mode)

* selecting:
  * `v`: start selecting characters
  * `V`: selects the whole line
* action:
  * `d`: to cut
  * `y`: to copy
  * `p`: to paste after the cursor
  * `gp`: to paste after the cursor and then move the cursor at the end of the pasted text
  * `P`: to paste before the cursor
  * `gP`: to paste before the cursor and then move the cursor at the end of the pasted text

## History

* `u`: undo last change
* `Ctrl-R`: redo last undone change

## Motions

### word / WORD

* `w`: move to the start of the next word
* `e`: move to the end of the current or next word
* `b`: move to the start of the current or previous word

> **Note**: upper case next word motions (e.g. `W`) will consider `"word"` as being 1 WORD, instead of 3 words.

### Inline occurence

* `f`: move to the next character occurence in the current line
* `t`: move one character before the next character occurence in the current line
* `;`: repeats the previous inline motion

> **Note**: upper case inline motions (e.g. `F`) will move to previous occurence.

## Splits

* `Ctrl-W v`: creates a new vertical split
* `Ctrl-W s`: creates a new horizontal split
* `Ctrl-W q`: closes the current split
* `Ctrl-W h`: switches the right split
* `Ctrl-W l`: switches the left split

## Misc

* `Ctrl-N`: autocomplete word

## Macro

A macro is a registered list of keys, which can be replayed.

* recording a macro:
  * `q<character>`: start recording a marco in the register `<character>`
    * example: `qa`, starts recording macro in register `a`
  * `<keys>`: the list of keys to register
    * example: `A,^[`, appends `,` at the end of the line (`^[` is Escape to quit _Insert Mode_)
  * `q`: stops recording
* replaying a macro:
  * `@<character>`: replays macro from reigster `<character>`
* editing a macro:
  * `:new`: create a new buffer
  * `:put <character>`: puts content from register `<character>`
  * `<edit macro>`: for example add `j` to go to the next line
  * `"<character>`: target register `<character>`
  * `yy`: copy the whole line into targeted register
* check all registers: `:reg`
