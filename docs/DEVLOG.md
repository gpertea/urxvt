# Development log

Newest entries first. Existing entries are retained as written.

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
