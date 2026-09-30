# Transition to the custom Ubuntu 24.04 package

The latest local build is `rxvt-unicode_9.31-3build2+xtermcompat2_amd64.deb`.
It enables xterm compatibility, native selection mirroring, middle-button
CLIPBOARD paste, and write-only OSC 52 by default. It targets Ubuntu 24.04
on amd64. Debug symbols are optional.

See [xterm compatibility](xterm-compat.md) and
[clipboard and tmux integration](clipboard-tmux.md) for the feature contracts.
The older xterm document's deferred OSC 52 and tmux items describe the first
feature stage; the clipboard document describes their subsequent implementation.

## Local configuration changes, 2026-09-30

Changes were applied only on the local workstation. Packages were previously
copied to `~/packages/` on gvlin, srv16, gw, gxlin, and gglin; their configuration
files were not edited by this task.

In `~/.Xresources`, remove the explicit Home, End, KP_Home, and KP_End key
mappings. These force fixed CSI sequences and override the fork's automatic
normal/application cursor-mode handling. Keep the explicit
`URxvt*termName: xterm-256color` and the existing appearance settings.

Add these explicit defaults:

```text
URxvt.xtermCompat: true
URxvt.selectionToClipboard: true
URxvt.middleClickPasteClipboard: true
URxvt.osc52Write: true
URxvt.osc52MaxBytes: 16384
```

Remove the redundant `selection-to-clipboard` Perl extension. The resulting
active extension list preserves the existing disabled paste-confirmation choice:

```text
URxvt.perl-ext-common: default,selection,selection-popup,-confirm-paste
```

In `~/.tmux.conf`, use explicit mouse and clipboard settings:

```text
set -g mouse on
set -s set-clipboard external
```

Retain `history-limit 30000`, `escape-time 50`, and existing PageUp/PageDown
bindings. Add the binding for the active emacs copy-mode table:

```text
bind -T copy-mode PageDown send-keys -X page-down
```

The vi-table binding remains available. Unprefixed PageUp/PageDown continue
to select tmux history navigation instead of being delivered to applications.
Leave tmux's inner terminal as `tmux-256color`; outside tmux, this fork uses
`xterm-256color`. `external` permits tmux copies to update the outer clipboard.
Nested tmux needs a separate configuration decision; see the clipboard guide.

The active VNC session uses `~/gscripts/vnc-session` and
`~/.config/vnc-session/jwm.xml`. Move X resource loading before
`start_urxvt_daemon` in that script. Its existing display-specific `RXVT_SOCKET`
is exported to JWM and terminal clients. This script change is local to the
gscripts checkout, not included as code in this companion repository.

No changes are needed in `~/.vnc/xstartup`: it already loads resources before
starting the daemon. No changes are needed in `~/.jwmrc` or the active JWM
configuration: their `urxvtc` commands inherit the compatibility defaults.

## Activation and verification

Removing a line from the resource file does not remove its existing value
from the X server when using `xrdb -merge`. Run this inside the target desktop:

```sh
xrdb -load ~/.Xresources
xrdb -query | rg 'URxvt'
```

`-load` replaces the entire X resource database with this file. Existing
terminal windows retain their settings. The local display `:1` was reloaded
and a new client of its running daemon was verified: TERM remained
`xterm-256color`, and Home in application cursor mode emitted `ESC O H`.
No daemon or VNC restart was needed for these configuration edits.

The daemon takes daemon options, while terminal options belong on the client:

```sh
urxvtd -q -f -o
urxvtc -xterm-compat
```

`-xterm-compat` is optional with this package and the explicit resource above.
Do not pass it to urxvtd. Start daemon and clients with the same `RXVT_SOCKET`.

Check outside tmux:

```sh
dpkg-query -W rxvt-unicode
printf '%s\n' "$TERM" "$RXVT_SOCKET"
```

Check inside tmux:

```sh
tmux show -s set-clipboard
tmux show -g mouse
tmux info | rg 'Ms:'
```

Expect `external`, `mouse on`, and an OSC 52 string for `Ms`, not `[missing]`.
No additional terminal override is needed when `Ms` is present. Reload an
existing server's settings with `tmux source-file ~/.tmux.conf`; reconnect
clients if terminal capability configuration changes. Keep running sessions
unless a specific check establishes that a restart is needed.

## Package replacement and rollback

Installing the package requires no VNC restart. Only a daemon still running
an older executable needs replacement to use newly installed terminal code.
Compare `/proc/<pid>/exe` with `/usr/bin/urxvtd`, using elevated read access
if required. On 2026-09-30, both local daemons matched the installed executable
by SHA256; no restart was needed for package installation.

Stopping a daemon closes its terminal windows. Save terminal work before any
deliberate daemon restart. A VNC restart is a broader action and is not required
solely to replace urxvtd.

Original configuration files were backed up under the companion repository:
`audit/config-backup_26-09-30_18-54_01a0f446-5bee-72b3-ae30-668c66171f38/`.
Restore `Xresources`, `tmux.conf`, or `vnc-session` to their original paths to
undo the corresponding edit, then reload resources or tmux settings as above.
The backups and approved implementation plan remain untracked.

## Validation of the local edits

- `bash -n` passed for the active VNC script and legacy xstartup.
- JWM configuration parsing passed; both JWM files were left unchanged.
- An isolated tmux server loaded the edited configuration and reported
  `mouse on`, `set-clipboard external`, and the emacs PageDown binding.
- The live display resource database contains the explicit native defaults
  and no removed Home/End overrides.
- A temporary client of the running daemon passed the TERM and application
  cursor-mode Home checks described above; its test window was closed.

Existing user terminal windows and tmux sessions were not restarted.
