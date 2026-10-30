# Native mouse cleanup

## Sources and implementation

The source base directory is `/home/gpertea/work/urxvt/rxvt-unicode/`, the
independent checkout of `gpertea/rxvt-unicode`. Paths below are relative to it.
The baseline is `9b38c069570d0e02f68057850c471af3336d7815`; the cleanup revision
is fdc69e5fbea82fb132f08cc22d4d80a7e126925e on `fix/mouse-cleanup`, integrated into source `gdev`.

At that revision, `src/command.C`, `mouse_cleanup_check`, observes foreground
process groups through the master PTY. `src/rxvt.h` defines `mouse_pgrp`,
`mouse_modes`, and `PrivMode_mouse_input`. Live and saved mouse/focus modes
belong to the observed job. A foreground handoff releases those modes; a
stopped job's cached modes are restored when it resumes. Dead groups are
discarded. No process names, titles, TERM values, or shell cleanup hooks are
used to identify jobs.

The guard runs before X input and through `mouse_cleanup_ev` every 100 ms while
state exists. `mouse_reset_state` cancels mouse gestures, updates
`vt_emask_mouse`, and schedules an X flush. It leaves bracketed paste under
application/shell control. `src/main.C`, `child_cb`, releases reporting when
the terminal's initial child exits, including `-hold` windows.
`src/screen.C`, `scr_poweron`, also discards cached job state during RIS.

The new `mouseAutoReset` resource defaults to true. Its command-line controls
are `-mouse-auto-reset` and `+mouse-auto-reset`. Disabling it preserves legacy
mode inheritance between jobs. If the PTY cannot report a foreground group,
the guard preserves existing reporting rather than disabling a live program.

## tmux pane cleanup

The outer urxvt PTY observes a tmux client, not each pane's foreground job.
The companion patch is `contrib/tmux/mouse-auto-reset-3.4.patch`, documented in
`contrib/tmux/mouse-auto-reset.md`. Its target is the official tmux 3.4 archive:

```text
tmux-3.4.tar.gz SHA256
551ab8dea0bf505c0ad6b7bb35ef567cdde0ccb84357df142c254f35a23e19aa
```

The patched tmux source base used for validation is
`/tmp/urxvt-mouse-cleanup-build/tmux-3.4/`. There, `input.c`,
`input_check_mouse`, applies the same job ownership policy to each pane's PTY.
`input-keys.c`, `server-client.c`, and `window.c` check state before delivering
mouse/focus input. Retained dead panes also release reporting. The window
option `mouse-auto-reset` defaults to on; it is independent of tmux's
session-level `mouse` option. Existing pane selection and clipboard behavior
remain available.

Pane cleanup requires the patched tmux server. Updating urxvt alone does not
provide it. Neither component edits shell configuration or installs hooks.
No system binary or existing terminal/tmux server was replaced by this work.

## Validation methods

`tests/mouse-cleanup-integration.py` reproduces live mouse reporting followed
by normal exit, SIGTERM, SIGKILL, and suspension in real Bash. Test shell hooks
record readiness and `READLINE_LINE`; they do not clear terminal modes.
The fixture's SIGCONT handler restores raw termios only, without re-enabling
mouse modes. Resume checks therefore exercise the native mode restoration.

Cases cover two suspended jobs, hard reset discarding cached state, focus
cleanup, tracking/encoding combinations, resource and CLI precedence, held
terminals, dead panes, tmux client death, mouse pane selection, and multiline
clipboard/bracketed paste. The default Perl paste-confirmation dialog remains
enabled and is accepted by the test.

`tests/mouse-reset-integration.py` retains separate manual-reset regression
cases, with automatic cleanup explicitly disabled for their stale-mode
precondition. `make check` also retains the existing keyboard, TERM, OSC/color,
clipboard ownership, OSC 52, and tmux integration tests.

Test libraries use absolute `PERL5LIB` and `URXVT_PERL_LIB` paths because
urxvtd changes its working directory to `/`. Both regression drivers reject
Perl initialization fallback. Earlier daemon checks using relative paths did
not establish that Perl hooks were active; the new checks correct that gap.

Both complete `make check` runs passed: 224 top-level reset/cleanup cases
across standalone/daemon and Perl/no-Perl builds, plus the retained keyboard,
TERM, OSC/color, clipboard, and tmux checks. Matrix cases also exercise all
urxvt tracking/encoding combinations and the subset supported by tmux 3.4.

```text
Path           Perl      Reset  Native  Pane  Total
-------------  --------  -----  ------  ----  -----
Standalone     Disabled     30      14    12     56
Daemon/client  Disabled     30      14    12     56
Standalone     Enabled      30      14    12     56
Daemon/client  Enabled      30      14    12     56
```

The companion tmux patch applied to a pristine 3.4 archive with zero fuzz,
and the resulting seven source files matched the tested build exactly.
Committed source/test inputs matched both urxvt build trees. Runtime Perl
fallback was rejected in the successful daemon runs.

The builds are `/tmp/urxvt-mouse-cleanup-build/noperl/` and
`/tmp/urxvt-mouse-cleanup-build/perl/`, using libev 4.33 and libptytty 2.0.
Each enabled 256 colors, with Perl explicitly enabled or disabled. Tests used
private HOME directories, X displays, daemon sockets, and tmux sockets.
The full checks set `TMUX_MOUSE_CLEANUP_BINARY` to the patched executable and
put it first in PATH for the legacy tmux clipboard tests. File/binary hashes,
compiler/dependency versions, and results are in the untracked local file
`audit/mouse-cleanup-validation/provenance.json`.


## Measured limits

The reset-only baseline still leaks mouse reports after SIGKILL. A new urxvt
build with an unpatched tmux server also retains the pane-level failure.
Those are separate from the patched direct and pane paths.

Foreground sampling cannot attribute all queued output. A controlled probe
paused the terminal/parser while a job enabled mouse reporting and exited,
then resumed parsing after the shell had regained the PTY. Mouse fragments
still reached Bash, both directly and in a patched pane. The PTY stream did
not identify the departed writer. This case remains unresolved; automatic
cleanup covers observed foreground-job transitions rather than every possible
mode-setting output race.

A surviving SSH connection similarly hides changes on its remote PTY. A
patched tmux server handles its own local pane jobs; a pane running SSH still
cannot infer an inner remote job's lifetime. Unsupported foreground queries,
unobserved transitions, and remote transport boundaries are not reported as
fixed. The hard-reset recovery patch remains available for these limits.

Local evidence is untracked under `audit/`: `check-mouse-cleanup-*.log`,
`mouse-cleanup-baseline.log`, `mouse-cleanup-pane-baseline.log`, and
`mouse-cleanup-queued-limit.json`. The controlled probe is
`audit/queued-mode-probe.py`.
