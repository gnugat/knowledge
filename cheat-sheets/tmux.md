# tmux cheat sheet

A highly opinionated [tmux](https://github.com/tmux/tmux/wiki) cheat sheet.

## Ubuntu 24.04 installation

Install the latest version:

```console
sudo apt install tmux
```

## Usage

Sessions:

* `tmux new-session -s <name>`: starts a new session
* `tmux kill-session -t <name>`: kills the session

Windows:

* `Ctrl + b, c`: creates new window
* `Ctrl + b, p`: switches to previous window
* `Ctrl + b, n`: switches to next window
* `Ctrl + b, <window-number>`: switches to specific window
* `Ctrl + b, w`: lists all windows

Panes:

* `Ctrl + b, %`: splits in two horizontal panes
* `Ctrl + b, "`: splits in two vertical panes
* `Ctrl + d`: closes pane
* `Ctrl + b, q`: lists pane numbers
* `Ctrl + b, <pane-number>`: switches to specific pane
* `Ctrl + b, <arrow-key>`: switches to specific pane
* `Ctrl + b, !`: breaks pane into its own window
* `Ctrl + b, : join-pane -s <window-number>`: brings window to the pane
