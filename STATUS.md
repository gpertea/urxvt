# Project status

## Current state

Managed source base: `/home/gpertea/work/urxvt/rxvt-unicode/`.
Source `gdev` is checked out at `57d33fb9`, containing all completed fork
patches through `fdc69e5f` plus Ubuntu packaging and the build recipe.
Source `devel` remains at upstream `fd64bc61`. Companion/meta uses `devel`.

Built and verified Ubuntu 24.04 amd64 package:
`~/packages/rxvt-unicode_9.31-3build2+xtermcompat3_amd64.deb`.
No debug symbols or debug-symbol package. Provenance, `.buildinfo`, `.changes`,
and SHA256 records identify the build inputs and artifact. Installed urxvt
remains `9.31-3build2+xtermcompat2`; live terminals and tmux servers are unchanged.

## Works now

The release build passed `make check`. Extracted release binaries separately
passed keyboard/clipboard/OSC integration and 88 mouse cases: 60 reset and
28 automatic-cleanup cases across standalone and daemon/client modes. Tests
used stock tmux and a private mount namespace for the packaged Perl library.
See [package build, checks, and artifact hash](docs/ubuntu-package.md).

Build again from a clean committed source `gdev` with
`contrib/build-ubuntu-package OUTPUT_DIRECTORY`, using a fresh directory
outside the source checkout. Dependencies are in `contrib/ubuntu-package.md`.
Stock tmux manual recovery remains `tmux send-keys -R -t SESSION:WINDOW.PANE`;
see [the recovery guide](docs/tmux-mouse-recovery.md).

## Decisions taken

- [Companion devel; managed source cumulative gdev](docs/decisions/001-fork-branches.md).
- [Native foreground-job cleanup with optional tmux pane support](docs/decisions/002-native-mouse-cleanup.md).

## Waiting on people

1. None for package delivery.

## Next

1. Install the verified package on target workstations when requested.

## Resuming

Source is on `gdev`; the feature/fix branches retain their review history.
Package verification and provenance are in [the package record](docs/ubuntu-package.md).
Local evidence is untracked under `audit/ubuntu-package-20261116/`; the release
build is `/tmp/urxvt-package-release-20261116/`. Binaries are not tracked in Git.

The urxvt package includes the optional tmux patch as documentation, not a
custom tmux binary. Automatic pane cleanup still requires a patched server;
manual recovery works with stock tmux. Unobserved queued output and SSH
boundaries remain limits; see [native cleanup notes](docs/native-mouse-cleanup.md).
