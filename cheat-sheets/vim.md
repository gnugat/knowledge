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

## Splits

* `Ctrl-W v`: creates a new vertical split
* `Ctrl-W s`: creates a new horizontal split
* `Ctrl-W q`: closes the current split
* `Ctrl-W <arrow>`: switches the the split in the arrow's direction
