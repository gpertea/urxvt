# Mouse reporting after an application is killed

## Sources and scope

Investigation date: 2026-10-29. The source base directory is
`/home/gpertea/work/urxvt/rxvt-unicode/`, an independent checkout.
Source paths below are relative to that directory.

Compared revisions from `gpertea/rxvt-unicode`:

- `origin/devel`: `fd64bc6183514641312da3242d6217fd9e14ad12`, the
  unmodified baseline, without either fork feature commit.
- `origin/clipboard-tmux`: `87454260b6169681db705a6a48443707b34ba235`,
  including the xterm compatibility and clipboard/tmux changes.

Runtime comparisons used source builds of both revisions, the installed
`rxvt-unicode` package `9.31-3build2+xtermcompat2`, and xterm `390-1ubuntu3`.
The [xterm control-sequence specification](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html)
defines mode 1003 as reporting all mouse motion, mode 1006 as SGR encoding,
mode 1015 as urxvt numeric encoding, and explicit private-mode reset controls.

## Findings

A child process enabled mouse reporting, focus reporting, and bracketed paste,
then received SIGKILL. Its parent remained alive. Mode queries confirmed that
1003 remained enabled; moving the mouse then delivered reports to the parent.
This occurred in all four terminal implementations tested, including xterm.

Separate interactive Bash tests reproduced numeric fragments in
`READLINE_LINE`, including `35;18;8M`, after the child was killed. The same
failure occurred directly and inside an isolated tmux session. These fragments
are application mouse input reaching the shell after the application has gone.

There is also a recovery defect inherited from the baseline urxvt source:

- At both revisions, `src/command.C`, `process_escape_seq`, handles RIS
  (`ESC c`) by calling `scr_poweron`.
- At both revisions, `src/screen.C:462`, `scr_poweron`, rebuilds screen state
  without clearing `priv_modes` or `vt_emask_mouse`. Mouse reporting survives
  RIS. Focus and bracketed-paste modes also survived the runtime probe.
- At baseline `src/command.C:2901` and fork `src/command.C:2949`, the DECSTR
  (`ESC [ ! p`) reset list omits 1002, 1003, 1004, and 2004. It resets mouse
  encodings 1005, 1006, and 1015 while leaving all-motion reporting enabled.
  The probe observed continued reports using the legacy encoding afterward.
- `reset` with `TERM=xterm-256color` sends the system terminfo reset controls.
  Those controls failed to stop reports in all three urxvt binaries tested.
  Xterm's RIS and `reset` stopped reports. Xterm's DECSTR alone retained mouse
  reporting, so soft reset is distinct from hard reset in this comparison.

The source comparison found no fork changes to these reset paths or the
private-mode enable/disable implementation. The clipboard patch reserves
Button 2 for local paste; it does not enable all-motion reporting. The xterm
profile changes the default TERM, but the baseline also reproduced the failure.

## Measured behavior

Each raw-byte result below was observed with both 1006 and 1015 encoding.
"Reports" means input bytes arrived after moving the mouse without pressing
a button. "None" means no mouse bytes arrived in the capture interval.

```text
Terminal           After kill  After RIS  After reset  Explicit disable
-----------------  ----------  ---------  -----------  ----------------
Installed fork     Reports     Reports    Reports      None
Baseline fd64bc61  Reports     Reports    Reports      None
Fork 87454260      Reports     Reports    Reports      None
Xterm 390          Reports     None       None         None
```

The explicit disable tested was:

```sh
printf '\033[?9;1000;1002;1003;1004;1005;1006;1015l'
```

It stopped mouse input in all eight raw-byte cases and all eight Bash cases.
In the Bash/tmux cases, `reset` also cleared the pane's stale mouse mode,
including when the outer terminal was urxvt. Direct Bash/urxvt cases retained
mouse input after `reset`. The direct baseline shell used its default
`TERM=rxvt-unicode`; the raw-byte reset comparison forced `xterm-256color`.

## Methods and limitations

The raw-byte probe queried 1003, 1004, the selected encoding, and 2004 using
DECRQM, and captured input after independent reset actions. The Bash probe
captured `READLINE_LINE` through a Readline binding without executing its
contents. Both used Xvfb and xdotool. Source builds disabled Perl and enabled
256 colors. The installed fork was also tested with its compiled Perl support.

Local, untracked evidence is under the companion repository's
`audit/mouse-results/`: eight raw-byte result files, eight Bash result files,
and `provenance.json` with revisions, package versions, and SHA256 hashes.
Probe scripts are `audit/mouse-repro.py` and `audit/shell-mouse-repro.py`.

The probes used a small mouse-enabled process rather than Claude or Codex.
They establish the terminal failure and the inherited recovery defect, but
do not establish which application, exit signal, or cleanup path produced a
particular live incident. No terminal implementation was changed or installed.
