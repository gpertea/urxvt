# Mouse reset recovery: implementation and validation

## Sources

The source base directory is `/home/gpertea/work/urxvt/rxvt-unicode/`.
All source paths below are relative to that independent checkout of
`gpertea/rxvt-unicode`.

- Baseline `87454260b6169681db705a6a48443707b34ba235`: cumulative xterm and
  clipboard/tmux features before this work.
- `4f1c2419`: resource-database lifecycle prerequisite.
- `9b38c069`: mouse reset implementation, regression tests, and user docs.

The two new commits are on `fix/mouse-reset` and integrated into `gdev`.

## Implemented behavior

At `9b38c069`, `src/screen.C`, `scr_poweron`, clears mouse tracking modes
9, 1000, 1002, and 1003; encodings 1005, 1006, and 1015; focus mode 1004;
and bracketed-paste mode 2004. It clears the corresponding bits in both
`priv_modes` and `SavedModes`. It also resets mouse gesture bookkeeping and
`vt_emask_mouse`, then updates the X event selection with `vt_select_input`.

RIS (`ESC c`) therefore clears reporting left enabled by an application that
exited without cleanup. Ordinary `reset` recovers the tested xterm and
rxvt-unicode terminfo paths. Subsequent applications can enable reporting again.
The existing DECSTR soft-reset behavior is unchanged.

At `4f1c2419`, `src/xdefaults.C`, `extract_resources`, retains the database
handle created when command-line resources are merged into an empty database.
`src/rxvttoolkit.C`, `rxvt_display::~rxvt_display`, detaches an explicitly
destroyed database before display close. Both changes use the existing
`XLIB_ILLEGAL_ACCESS` path. They address a resource-option failure and shutdown
double destruction encountered by the existing suite on clean Xvfb displays.

## Validation

At `9b38c069`, `tests/mouse-reset-integration.py` exercises actual X events,
DECRQM replies, private-mode save/restore, clipboard input, and Bash Readline.
The unchanged baseline failed 28 of 30 standalone cases; the two passing
controls were existing DECSTR behavior and Bash recovery through tmux.

All 30 cases passed in each patched configuration:

```text
Executable path       Perl      Passed  Failed
--------------------  --------  ------  ------
Standalone rxvt       Disabled      30       0
Standalone rxvt       Enabled       30       0
Isolated rxvtd/rxvtc  Disabled      30       0
Isolated rxvtd/rxvtc  Enabled       30       0
```

Cases cover all four tracking modes with legacy, UTF-8, SGR, and urxvt
encoding; live and saved mode clearing; re-enabling reporting; focus events;
bracketed and ordinary clipboard paste; a held button across RIS; the actual
X PointerMotion event mask; and recovery after SIGTERM/SIGKILL with
`TERM=xterm-256color` and `TERM=rxvt-unicode-256color`. Bash cases verify
mouse fragments before `reset` and empty `READLINE_LINE` afterward, directly
and through isolated tmux.

`make check` also passed the existing C++ keyboard/OSC 52 checks and X tests
for key sequences, resource precedence, TERM, OSC color replies, clipboard
ownership/paste, and tmux clipboard integration in both builds. The daemon
reset runs were executed separately after that check; the committed Makefile
now runs both standalone and daemon reset suites from `make check`.

Builds enabled 256 colors, with `--enable-perl` or `--disable-perl`, and used
libev 4.33 and libptytty 2.0. Source/test inputs were compared with the checkout.
Test HOME directories, X displays, daemon sockets, and tmux sockets were isolated.
The optional X event-mask check ran with xwininfo installed in both builds.

Local commands, from each build directory:

```sh
make check
PERL5LIB=src URXVT_PERL_LIB=src/perl xvfb-run -a python3 tests/mouse-reset-integration.py --daemon src/rxvt
```

The build directories are
`/tmp/urxvt-mouse-investigation.lC0jDP/patched-noperl/` and
`/tmp/urxvt-mouse-investigation.lC0jDP/patched-perl/`.
Untracked companion evidence is in `audit/check-mouse-reset*.log`,
`audit/mouse-reset-baseline-final.log`, and
`audit/mouse-reset-validation/provenance.json`.

## Limits

This fixes reset recovery, not automatic cleanup when an application dies.
The probes use small mouse-enabled applications, not Claude/Codex internals.
No replacement package was built or installed. Existing terminal processes
continue to run their installed executable. The exact source revision of the
older saved package remains unrecorded.
