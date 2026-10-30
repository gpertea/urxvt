# Development log

Newest entries first. Existing entries are retained as written.

## 2026-10-30  Stock tmux manual mouse recovery

Branch: companion/meta `devel`. Documentation commit: `9ce7732`.
Managed source branches and installed binaries are unchanged.

- Done: added [a recovery guide](tmux-mouse-recovery.md), linked it from README
  and native cleanup notes, and documented pane targets and a prefix-R binding.
  Kept the companion tmux patch optional for automatic pane cleanup.
- Measured: installed stock tmux 3.4 reproduced mouse fragments in Bash after
  SIGKILL. `send-keys -R -t cleanup:0.0` stopped further fragments after Ctrl+U
  cleared old input. Shell PID, another pane's mouse mode, and tmux's own mouse
  option were preserved. Documentation ASCII, local links, and whitespace pass.
- Not done: automatic recovery using stock tmux; package builds or installation.
- Next: record reproducible urxvt packaging; keep custom tmux optional.

## 2026-10-30  Native foreground-job mouse cleanup

Branches: companion/meta `devel`; managed source `fix/mouse-cleanup` and `gdev`.
Source commit: `fdc69e5f`. Branch instructions clarified in companion `f358c89`.
Local plan: [audit plan](../audit/plan_26-10-30_08-19_fix-mouse-cleanup.md), untracked.

- Done: automatic mouse/focus cleanup on observed job handoffs, saved state
  for stopped/resumed jobs, dead-group pruning, held-terminal cleanup, and a
  tested companion tmux 3.4 patch for pane-local jobs and retained dead panes.
  Integrated and pushed source changes to managed `gdev`.
- Measured: 224 reset/cleanup cases passed across standalone/daemon and
  Perl/no-Perl, plus existing keyboard, resource, OSC/color, clipboard, and
  tmux checks. Multiline paste retained Perl confirmation and bracketed paste.
  Source inputs and zero-fuzz application of the tmux patch matched builds.
  Absolute Perl paths and fallback rejection correct the gap in prior daemon
  checks, which did not establish active Perl hooks.
- Not done: packaging/install, universal attribution of mode-setting output
  queued across unobserved job transitions, and hidden SSH-side job detection.
  The controlled queued-output probe still reproduces that documented limit.
- Next: record and prepare reproducible urxvt/tmux package builds.

## 2026-10-29  Mouse reset recovery

Branches: companion `devel`; source `fix/mouse-reset` and `gdev`.
Source commits: `4f1c2419` (resource lifecycle), `9b38c069` (hard reset/tests).
Local approved plan: [audit plan](../audit/plan_26-10-29_16-39_fix-mouse-reset.md), untracked.

- Done: clear live/saved input-reporting state and X motion selection on RIS;
  preserve DECSTR behavior; document recovery; integrate into cumulative branch.
- Measured: baseline failed 28/30 standalone cases; patched builds passed
  120/120 cases across standalone/daemon and Perl/no-Perl. Existing keyboard,
  OSC/color, resource, clipboard, and tmux checks passed in both builds.
- Not done: package reconstruction/build/install and automatic exit cleanup.
- Next: record Debian packaging and source/build provenance before a release.

## 2026-10-29  Prior state reconstructed from git history

Branches: companion `devel` through `b2df10d`; source cumulative `gdev`
at `87454260`, with upstream-aligned `devel` at `fd64bc61`.

- Done: source `a92e1183` added xterm compatibility; `87454260` added native
  clipboard/OSC 52/tmux integration. Companion commits `6a86e44`, `d7fa3f2`,
  `9bc1c51`, `5920293`, and `b2df10d` documented feature contracts, the mouse
  investigation, and cumulative-branch conventions.
- Measured: the investigation reproduced stale reports after SIGKILL and
  failing reset recovery in baseline and fork builds; xterm recovered on RIS.
- Not done: no Debian packaging, release tag, or exact saved-package source
  revision was recorded in the source repository.
- Next: implement and validate the inherited hard-reset defect.
