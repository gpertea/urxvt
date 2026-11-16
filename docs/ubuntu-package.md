# Ubuntu 24.04 package

## Source and build

The source base directory is `/home/gpertea/work/urxvt/rxvt-unicode/`, the
managed `gpertea/rxvt-unicode` repository. Source revision `57d33fb9` on `gdev`
contains all cumulative fork patches and the Debian packaging. Paths below
are relative to that source base unless explicitly absolute.

`debian/README.source` identifies the Ubuntu `9.31-3build2` packaging archive
and its SHA256. Seven retained vendor patches apply with zero fuzz. The OSC
terminator fix is already in the managed source and is omitted from the
vendor patch series. Debian integration includes app-defaults, desktop files,
alternatives, Perl extensions, and setgid `root:utmp` terminal/daemon binaries.

The build command is:

```sh
contrib/build-ubuntu-package /tmp/urxvt-package-release-20261116
```

`contrib/ubuntu-package.md` lists dependencies and verification requirements.
The builder exports a clean committed `gdev`, adds only the missing libev
directory from a checksum-pinned Ubuntu 9.31 source archive, and invokes
`dpkg-buildpackage -b -us -uc`. The build host is Ubuntu 24.04.4 amd64.

The package version is `9.31-3build2+xtermcompat3`. It includes xterm
compatibility, native clipboard/OSC 52 integration, resource lifetime fixes,
hard-reset recovery, and automatic cleanup on observed foreground-job changes.
The build enables Ubuntu's full feature configuration, including Perl, Xft,
pixbuf, startup notification, transparency, and 256 colors. `debian/rules`
strips binaries, disables automatic debug-symbol packages, and skips DWARF
compression to prevent a shared `.dwz` debug file in the release package.

`/usr/share/doc/rxvt-unicode/build-provenance` in the package records the exact
source commit and Git export/libev archive hashes. The adjacent `.buildinfo`
artifact records build flags and dependency versions. These records identify
build inputs; bit-for-bit reproducibility across environments is unmeasured.

## Artifact

The release is
`~/packages/rxvt-unicode_9.31-3build2+xtermcompat3_amd64.deb`.
Its SHA256 is:

```text
9f3c018d77deb63b7e077450688656254cd86eee5a4d16d78193e518c0716767
```

Matching `.buildinfo`, `.changes`, and `.sha256` files are beside the package.
No `xtermcompat3` debug-symbol package was produced. The package has not been
installed; the installed urxvt version remains `9.31-3build2+xtermcompat2`.
An apt simulation resolves the replacement as a single package upgrade,
without additional dependencies or removals on this workstation.

## Verification method

The normal package build runs `make check`, including the keyboard/OSC unit
tests and X integration checks. Binary package verification uses `dpkg-deb`
metadata and payload inspection, file checksums, `file`, and `readelf`.
Checks cover architecture/version, root/utmp permissions, stripped executables,
absence of debug files and debug links, and embedded source/export provenance.

Extracted `urxvt`, `urxvtd`, and `urxvtc` are copied under the regression
drivers' build-tree names and verified byte-for-byte against package contents.
Standalone and daemon copies retain setgid `root:utmp` permissions. A private
mount namespace binds the extracted Perl library at the compiled library path;
this prevents the older installed `urxvt.pm` from taking precedence over
`PERL5LIB`. Tests run as the ordinary user with private HOME, Xvfb displays,
daemon sockets, and tmux sockets. Interpreter fallback is rejected.

The executed drivers are `tests/xterm-compat-integration`,
`tests/clipboard-tmux-integration`, `tests/mouse-reset-integration.py`, and
`tests/mouse-cleanup-integration.py`, with both Python drivers also run using
`--daemon`. Package checks use stock tmux 3.4. The optional pane cleanup patch
is shipped as documentation; no custom tmux binary is built or installed.

The queued-output and remote SSH boundaries described in
[native cleanup limits](native-mouse-cleanup.md#measured-limits) still apply.

## Results, 2026-11-16

The release build's `make check` passed its keyboard/OSC unit checks, X
integration suites, and 88 mouse cases without failures or skips. Extracted
release binaries separately passed the keyboard/clipboard/OSC integration
suites and 88 mouse cases: 30 reset plus 14 cleanup cases in standalone mode,
and the same 44 cases in daemon/client mode. The package checks used stock
tmux 3.4; they do not claim automatic pane cleanup with stock tmux.

All payload MD5 checksums and the embedded Git export hash matched. All three
terminal binaries were stripped, without debug sections or debug links. No
debug payload files, `.ddeb`, or `-dbgsym` package were produced. Archive
metadata retained the expected root/utmp setgid permissions. The `.deb`,
`.buildinfo`, and `.changes` copies in `~/packages/` passed their SHA256 checks.

Local test logs, binary hashes, the verification driver, and extracted package
are untracked under the companion repository's `audit/ubuntu-package-20261116/`
and `audit/verify-ubuntu-package.py`. The verification summary is
`audit/ubuntu-package-verification.log`; the build log is
`/tmp/urxvt-package-release-20261116/build.log`.
