# tmux cheat sheet

A highly opinionated [tmux](https://github.com/tmux/tmux/wiki) cheat sheet.

## Ubuntu 24.04 installation

Install the latest version:

```console
sudo apt install tmux
```

## Modes

* default: **Normal Mode**
* `Ctrl` + `b`, `:`: **Command Mode**
* `Ctrl` + `b`, `[`: **Copy Mode**

## Usage

**Sessions**:

* `tmux new-session -s <name>`: starts a new session
* `tmux list-sessions`: lists all sessions
* `tmux kill-session -t <name>`: kills the session

**Windows**:

* `Ctrl` + `b`, `c`: creates new window
* `Ctrl` + `b`, `p`: switches to previous window
* `Ctrl` + `b`, `n`: switches to next window
* `Ctrl` + `b`, `<window-number>`: switches to specific window
* `Ctrl` + `b`, `w`: lists all windows

**Panes**:

* `Ctrl` + `b`, `%`: splits in two horizontal panes (left and right)
* `Ctrl` + `b`, `"`: splits in two vertical panes (top / bottom)
* `Ctrl` + `d`: closes pane
* `Ctrl` + `b`, `q`: lists pane numbers
* `Ctrl` + `b`, `<pane-number>`: switches to specific pane
* `Ctrl` + `b`, `<arrow-key>`: switches to specific pane
* `Ctrl` + `b`, `!`: breaks pane into its own window
* `: join-pane -s <window-number>`: brings window to the pane

**Copy Mode**:

* `Ctrl` + `b`, `[`: Enter Copy Mode
* `Space`: Start selection
* `Enter`: Copy selection
* `Ctrl` + `b`, `]`: Pastes contents of *buffer_0*
* `q`: Quits Copy Mode (back to Normal Mode)
* `: show-buffer`: Displays *buffer_0* contents
* `: list-buffers`: Lists all buffers
* `: delete-buffer-b 1`: Deletes *buffer_1*
* `: save-buffer <filename>`: Saves buffer contents to file
