# Separate upstream tracking and cumulative fork branches

Date: 2026-10-29.

In the source repository `gpertea/rxvt-unicode`, keep `devel` aligned with
upstream devel. Integrate fork changes into `gdev`, and base feature/fix
branches on `gdev`. Use the cumulative branch for future package builds.
Keep focused branches available for review and upstream patch preparation.

`gdev` was created at `87454260`, which already included `xterm-compat`
commit `a92e1183` and `clipboard-tmux` commit `87454260`. No cherry-picking
or rewriting of those commits was needed.

The companion repository `gpertea/urxvt` continues to use `devel`. Its plans,
findings, and handoff records stay outside the independent source checkout.
