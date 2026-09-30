# Clipboard and tmux integration

## Scope

The `clipboard-tmux` feature builds on the default xterm compatibility profile.
It provides one desktop clipboard model and the generic mouse policy needed for
tmux without trying to identify tmux from process names, titles, or local
process trees.

The implemented layer is:

- a completed urxvt selection owns independent PRIMARY and CLIPBOARD snapshots;
- Button 2 always pastes CLIPBOARD locally, including during DEC mouse mode;
- Button 1, Button 3, border interactions, and wheel events continue to reach
  applications when mouse reporting is active;
- native OSC 52 writes let tmux copy mode update the outer X clipboard locally
  or across SSH;
- OSC 52 queries are ignored, so terminal output cannot read the clipboard.

The implementation does not claim that mouse mode identifies tmux. The same
routing applies to every application which enables terminal mouse reporting.

## Defaults and resources

For local configuration changes and activation steps, see
[transition and upgrade advice](transition-upgrade.md).

All new behaviors are enabled by default in this fork:

```text
URxvt.selectionToClipboard: true
URxvt.middleClickPasteClipboard: true
URxvt.osc52Write: true
URxvt.osc52MaxBytes: 16384
```

The corresponding command-line controls are
`-/+selection-to-clipboard`, `-/+middle-click-paste-clipboard`,
`-/+osc52-write`, and `-osc52-max-bytes number`.

The maximum accepted `osc52MaxBytes` value is 24000. OSC control sequences must
fit in urxvt's fixed 32768-byte input buffer after Base64 encoding. The lower
default leaves room for framing and avoids large clipboard changes by accident.

The existing `selection-to-clipboard` Perl extension can be removed from
`perl-ext-common`; native selection mirroring makes it redundant. Leaving the
extension enabled produces the same final clipboard text but performs duplicate
work.

Button 2 ownership is deliberate. When `middleClickPasteClipboard` is enabled,
Button 2 press, motion, and release are unavailable to application mouse mode
and Perl button hooks. The pasted bytes still pass through the normal paste hook,
bracketed paste handling, and `confirm-paste`. Disable the resource to restore
historical PRIMARY paste and application Button 2 reporting.

## OSC 52 contract

The accepted form is `OSC 52 ; Pc ; Pd ST`, with BEL also accepted as the
terminator. Selectors `c`, `p`, and `s` address CLIPBOARD, PRIMARY, and the
configured selection. With selection unification enabled, `s` updates both.
An empty selector is treated as `s`; secondary and cut-buffer selectors are
rejected.

`Pd` must be canonical RFC 4648 Base64 and decode to valid UTF-8 text without an
embedded NUL. Empty data creates an owned empty selection. Malformed Base64,
invalid UTF-8, unsupported selectors, disabled writes, and oversized payloads
leave the previous selections unchanged.

`Pd` equal to `?` is a read request and receives no response. Clipboard reads
are intentionally absent rather than controlled by the older broad `insecure`
resource.

Any process capable of writing output to the terminal can request an OSC 52
clipboard update. This includes programs running through SSH. Set
`URxvt.osc52Write: false` for sessions where output is not trusted.

The protocol behavior follows the xterm OSC 52 description:

- <https://invisible-island.net/xterm/ctlseqs/ctlseqs.html>

## tmux configuration

tmux 2.6 and newer default `set-clipboard` to `external`. Current tmux also
adds the `Ms` capability automatically when the outer `TERM` matches `xterm*`.
Because this fork exports `TERM=xterm-256color`, a clean current tmux setup
normally needs no configuration change.

The supplied explicit configuration is:

```text
set -s set-clipboard external
set -g mouse on
```

It is available at `contrib/tmux/urxvt-integration.conf`. The Ubuntu package
also installs it as
`/usr/share/doc/rxvt-unicode/urxvt-integration.conf`. To use it from an existing
`~/.tmux.conf`, source it or copy the two settings. Confirm support inside tmux
with:

```sh
tmux show -s set-clipboard
tmux info | grep 'Ms:'
```

The first command should report `external`; the second should show an OSC 52
string rather than `[missing]`. Restart the tmux server after changing its
configuration. Existing pane, resize, status-line, copy-mode, and wheel mouse
bindings remain tmux policy.

Use `set-clipboard external` for a single tmux layer. It lets tmux send copied
text outward while ignoring clipboard writes from applications inside tmux.
Nested tmux requires the outer tmux layer to use `set-clipboard on`, which also
allows inner applications to create tmux buffers and has a broader security
boundary. That is not enabled by the supplied configuration.

References:

- <https://github.com/tmux/tmux/wiki/Clipboard>
- <https://man.openbsd.org/tmux.1>

## Deferred history and scrollbar work

The outer urxvt scrollbar cannot represent tmux history accurately from output
or mouse modes. Each tmux pane has its own history and copy-mode position. A
correct outer slider needs a versioned protocol carrying active pane, history
size, copy-mode state, and scroll position, plus expiry when the state becomes
stale.

Stateless conversion of scrollbar actions to wheel events is also deferred. It
can be misrouted to an application inside a pane and does not guarantee entry
into tmux copy mode. Custom tmux `user-keys[]` can provide deterministic page
commands, but they require user configuration and belong in a separately
reviewable history-integration branch.

## Validation

Pure C++ tests cover Base64 padding, canonical encoding, size limits, and UTF-8
validation. Xvfb tests cover independent X selection ownership, local Button 2
paste with mouse reporting and bracketed paste, strict no-fallback behavior,
retained Button 1 and wheel reports, OSC 52 selectors and terminators, rejected
input preserving old content, option disablement, and a real isolated tmux
server copying through its `Ms` capability.

The remaining manual checks are desktop clipboard-manager persistence, actual
SSH transport to representative Linux and macOS hosts, nested tmux if enabled,
and the user's preferred tmux copy-mode key table.
