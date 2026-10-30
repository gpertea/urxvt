# Native cleanup follows foreground-job ownership

Date: 2026-10-30.

Associate mouse tracking, mouse encodings, focus reporting, and their saved
values with observed foreground jobs. Enable this policy by default in urxvt.
Keep a resource/CLI opt-out for deliberate inheritance between jobs. Preserve
stopped jobs' state for resume and discard dead jobs' state.

Implement pane cleanup in tmux through a companion source patch. A surviving
multiplexer or SSH connection does not expose inner job transitions through
the outer terminal PTY. Avoid application-name guesses and automatic full
screen resets. Preserve bracketed-paste policy and existing clipboard hooks.

Treat delayed/unobserved mode-setting output as an explicit observability
limit. Validation must reproduce that boundary rather than claim universal
exit detection. See [implementation and measurements](../native-mouse-cleanup.md).
