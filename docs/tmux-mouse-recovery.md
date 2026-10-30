# Recover a tmux pane flooded with mouse reports

Use this when a mouse-enabled application exits and moving the mouse leaves
fragments such as `35;18;10M` in the shell. Stock tmux can reset the affected
pane directly. Neither patched tmux nor patched urxvt is required for this
manual recovery.

## Reset a specific pane from another terminal

Find the session, window, and pane:

```sh
tmux list-panes -a -F '#{session_name}:#{window_index}.#{pane_index}'
```

Reset the affected target, replacing `mysession:0.0` with its actual address:

```sh
tmux send-keys -R -t 'mysession:0.0'
```

This example targets window 0, pane 0 in `mysession`. Run the command on the
host running that tmux server. For a named socket, use `tmux -L NAME ...`, or
use `tmux -S /path/to/socket ...` for an explicit socket path, with both commands.

The command resets tmux's internal pane terminal state; it does not type a
shell command or restart the shell. It is a full terminal-state reset, so it
also resets display attributes and clears the visible pane. It leaves tmux's
own `mouse` option and other panes' terminal modes unchanged.

Press Ctrl+U in the recovered shell to discard fragments already on its
command line. The reset stops further reports but does not erase shell input
already received. Do not press Enter on that polluted command line.

## Recover from inside the affected pane

Press the tmux prefix (Ctrl+B by default), then `:`, and enter:

```tmux
send-keys -R
```

For a shortcut, add this line to `~/.tmux.conf`:

```tmux
bind-key R send-keys -R
```

To activate the binding immediately without restarting tmux:

```sh
tmux bind-key R send-keys -R
```

Then press the tmux prefix followed by Shift+R to reset the current pane.
This assigns or replaces the prefix-table `R` binding.

## Validation and scope

Validated on 2026-10-30 with the installed, unpatched `/usr/bin/tmux` 3.4.
The test harness was `tests/mouse-cleanup-integration.py` from the independent
source checkout `/home/gpertea/work/urxvt/rxvt-unicode/`, revision `fdc69e5f`.
It ran in a private Xvfb display and tmux socket with a temporary HOME.

A mouse-reporting application was killed with SIGKILL. Subsequent real mouse
movement reproduced `35;18;10M` in Bash's `READLINE_LINE`. After
`tmux send-keys -R -t cleanup:0.0` and clearing the old input with Ctrl+U,
further mouse movement left `READLINE_LINE` empty. The shell PID was unchanged,
a second pane retained its enabled `mouse_any_flag`, and the session's `mouse`
option remained `on`.

This is manual recovery, not automatic exit detection. It resets the tmux pane
addressed by the command; nested tmux or a remote terminal layer may require
resetting the inner pane on the host that owns it. Automatic cleanup and its
separate limits are described in [native mouse cleanup](native-mouse-cleanup.md).

Reference: [tmux manual, send-keys -R](https://man.openbsd.org/tmux.1#send-keys).
