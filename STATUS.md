# Project status

## Current state

Source checkout: `/home/gpertea/work/urxvt/rxvt-unicode/`.
`gdev` and `fix/mouse-reset` contain the two original feature commits plus
resource-lifecycle fix `4f1c2419` and mouse-reset fix `9b38c069`.
All 120 reset cases passed across standalone/daemon and Perl/no-Perl builds.
Source `devel` remains at `fd64bc61`. Installed packages and live terminals
have not been changed. Debian packaging is not yet recorded in Git.

## Works now

In each tested build directory, `make check` passed the existing suite and
30 standalone reset cases. The isolated daemon command also passed 30 cases:
`PERL5LIB=src URXVT_PERL_LIB=src/perl xvfb-run -a python3 tests/mouse-reset-integration.py --daemon src/rxvt`.
The current Makefile includes both reset runs in `make check`.
See [validation details](docs/mouse-reset-fix.md).

## Decisions taken

- [Keep upstream devel separate from cumulative fork gdev](docs/decisions/001-fork-branches.md).
- Hard reset clears input reporting; automatic exit cleanup remains separate.

## Waiting on people

1. None for the completed reset patch.

## Next

1. Recover or reconstruct Debian packaging and record source/build provenance
   before building a replacement package.
2. Treat automatic shell/application cleanup after unclean exits as separate
   work if requested.

## Resuming

Source is on `fix/mouse-reset`; the same commits are integrated into `gdev`.
Read [the fix notes](docs/mouse-reset-fix.md) and [the development log](docs/DEVLOG.md).
Local logs and the approved plan are untracked under `audit/`. Temporary
builds are under `/tmp/urxvt-mouse-investigation.lC0jDP/` and may expire.
