# Project status

## Current state

Managed source base: `/home/gpertea/work/urxvt/rxvt-unicode/`.
Source `gdev` and `fix/mouse-cleanup` contain `fdc69e5f`: native automatic
mouse/focus cleanup plus a companion tmux 3.4 pane patch. All 224 regression
cases passed across standalone/daemon and Perl/no-Perl builds. Source `devel`
remains at `fd64bc61`; this companion/meta repository stays on `devel`.
Installed packages and live terminals/servers are unchanged. Packaging is not
yet recorded in Git. Unobserved queued output and SSH boundaries remain limits.

## Works now

Both isolated builds passed `make check`, including legacy reset tests,
automatic cleanup, and patched tmux panes. Set `TMUX_MOUSE_CLEANUP_BINARY` to
the new tmux executable; place it first in PATH for legacy clipboard tests.
See [commands, results, and limits](docs/native-mouse-cleanup.md).
The new checks use absolute Perl library paths and reject interpreter fallback.

## Decisions taken

- [Companion devel; managed source cumulative gdev](docs/decisions/001-fork-branches.md).
- [Native foreground-job cleanup with pane-side tmux support](docs/decisions/002-native-mouse-cleanup.md).

## Waiting on people

1. None for source/test delivery.

## Next

1. Record reproducible Debian packaging and provenance for both urxvt and the
   companion tmux build before preparing replacement packages.

## Resuming

Source is on `fix/mouse-cleanup`, integrated and pushed to source `gdev`.
Read [the native cleanup notes](docs/native-mouse-cleanup.md) and
[the development log](docs/DEVLOG.md). Local plan, logs, and provenance are
untracked under `audit/`; builds are under `/tmp/urxvt-mouse-cleanup-build/`.
Pane cleanup requires a patched tmux server; the outer urxvt change alone
cannot observe pane jobs. No shell cleanup hooks are required for the tested
native local/pane paths.
