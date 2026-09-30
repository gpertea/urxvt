# Xterm compatibility profile

## Purpose

The `xtermCompat` profile gives applications the common keyboard and color
behavior associated with `xterm-256color`. It is enabled by default in this
fork. A normal local shell, tmux session, Linux HPC login, or macOS SSH target
should not require rxvt terminfo installation or personal key remapping.

This is a compatibility contract, not a claim that urxvt implements every
xterm private extension. The behavior was reviewed against the xterm control
sequence documentation, the xterm manual, the ncurses `xterm-256color`
definition, and the GNU Readline initialization rules.

References:

- <https://invisible-island.net/xterm/ctlseqs/ctlseqs.html>
- <https://www.invisible-island.net/xterm/manpage/xterm.html>
- <https://invisible-island.net/ncurses/terminfo.src.html>
- <https://www.gnu.org/software/bash/manual/html_node/Readline-Init-File.html>

## Activation and environment

For existing workstation configurations and package replacement, see
[transition and upgrade advice](transition-upgrade.md).

`URxvt.xtermCompat: true` is the default. `-xterm-compat` enables the profile
and `+xterm-compat` disables it.

When `termName` is not set explicitly, the profile exports
`TERM=xterm-256color` from a 256-color build and `TERM=xterm` from a build
without 256-color support. An explicit `termName` resource or `-tn` argument
always wins. Disabling the profile restores the compiled urxvt TERM default.

An rxvt-specific `RXVT_TERMINFO` build path is not exported while the profile
is active, including when `termName` was set explicitly. An inherited
`TERMINFO` is left untouched. `COLORTERM` remains `rxvt` or `rxvt-xpm`; the
profile does not claim full truecolor support.

SSH passes TERM to the remote host. Current Linux and macOS systems normally
provide an `xterm-256color` entry, so remote applications can start without an
rxvt-unicode terminfo entry. If a remote system does not contain that standard
entry, its system administrator must install a suitable ncurses database.

## Keyboard contract

The profile emits the standard xterm sequences for cursor keys, Home, End,
Insert, Delete, Page Up, Page Down, and F1 through F35. Modified forms use the
xterm parameter `1 + Shift + 2*Meta + 4*Control`, producing values 2 through
8. Meta means the modifier selected by the urxvt `modifier` resource; it is not
hard-coded to Mod1.

Unmodified arrows, Home, and End use CSI in normal cursor mode and SS3 in
application cursor mode. Modified forms always use `CSI 1;<modifier>`.
Unmodified F1 through F4 use SS3. F5 through F12 and editing keys use their
normal tilde forms. Physical F13 through F35 use xterm's extended numeric
parameters. Shift+F1 and similar combinations remain parameterized forms of
their base key, as emitted by reference xterm.

Caps Lock, Num Lock, and ISO Level 3 do not change the xterm modifier number.
They retain their existing effects on text and keypad translation. Application
keypad mode and Shift's existing keypad override are also preserved.

### X resources and hooks

Existing explicit key mappings remain authoritative. For keys managed by this
profile, a modifier-qualified mapping such as `keysym.C-Left` matches that
exact logical Shift/Control/Meta combination. An unqualified mapping such as
`keysym.Home` applies only to unmodified Home. This prevents a common Home or
End customization from suppressing all xterm modified forms.

The same rule covers `M-`, `A-`, the configured raw modifier, and keypad
navigation keysyms. Lock, Num Lock, and ISO Level 3 state do not prevent an
otherwise exact match. With the profile disabled, the historical urxvt subset
matching rule is unchanged.

The key press hook still sees the computed profile sequence first. If the hook
does not consume the event, an exact key resource can consume it. Otherwise,
Shift+Insert keeps the local selection paste action and Shift+PageUp/PageDown
keep local scrollback behavior. Ordinary character input, XIM, ISO 14755,
`meta8`, and Control+Meta clipboard shortcuts keep their existing behavior.

### Readline and other consumers

The terminal produces standard xterm bytes; Readline, tmux, curses, and remote
programs interpret those bytes. A clean account can use the system inputrc and
does not need a `~/.inputrc` for basic Home, End, and modified cursor behavior.
Personal inputrc rules that deliberately bind old rxvt-only byte sequences are
application configuration and may still select those old meanings.

The profile does not edit XKB maps, `~/.Xmodmap`, `~/.Xresources`,
`~/.inputrc`, or tmux configuration. It uses the modifier masks delivered by
the active X keyboard map.

## OSC color queries

OSC 10 and OSC 11 queries return `rgb:rrrr/gggg/bbbb` while the profile is
enabled, even when the configured Xft color contains alpha. The reply reports
the configured RGB components and preserves the query's BEL or ST terminator.
Disabling the profile retains urxvt's `rgba:` reply for translucent colors.

## Capability audit

Implemented by the first patch or already present in urxvt:

- truthful default TERM selection for 256-color and non-256-color builds;
- normal and application cursor keys and xterm modifier parameters;
- editing keys and F1 through F35;
- explicit key resource precedence and existing local Shift actions;
- 16-color and 256-color operation already present in configured builds;
- semicolon-form `38;2`/`48;2` RGB operation already present in urxvt;
- OSC 10/11 reusable RGB query replies;
- bracketed paste, focus reporting, and SGR 1006 mouse reporting already
  implemented by urxvt.

Accepted differences:

- `COLORTERM` remains an urxvt identifier;
- xterm and urxvt have different non-profile resources, extensions, and
  default user-interface behavior;
- simultaneous distinct Alt and Meta cannot be represented because urxvt
  configures one Meta modifier.

Known unsupported or deferred behavior:

- OSC 52 clipboard exchange, although some `xterm-256color` databases expose
  the extended `Ms` capability;
- SGR faint and invisible rendering;
- colon-form ISO 8613-6 RGB parameters;
- xterm private controls not confirmed by source and runtime probes;
- tmux mouse/history policy and packaging automation.

Applications must not infer support for every extended capability solely from
the TERM value. Deferred items require separate implementation and tests.

## Acceptance checks

Automated validation builds both color configurations and tests normal and
application cursor encoding, editing and function keys, modifier values 2
through 8, physical extended function keys, exact key resources, local Shift
actions, option precedence, TERMINFO handling, and OSC query termination. The
Xvfb/xdotool tests compare stable samples with a reference xterm.

Before a release, manually verify remaining keypad and lock combinations, XIM,
`meta8`, hooks, a clean HOME with the system inputrc, a local Readline shell,
tmux, and SSH sessions to representative Linux and macOS hosts. The user's real
configuration files are never rewritten.
