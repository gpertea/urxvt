# Project scope

This is the companion (chaperone) repository for agent-assisted work on
`gpertea/rxvt-unicode`.

- Companion: `gpertea/urxvt`, rooted here. Keep plans, reviewed design notes,
  agent instructions, and supporting workflow material here.
- Code: `rxvt-unicode/`, an independent checkout of `gpertea/rxvt-unicode`.
  Keep feature code, tests, build changes, and necessary user documentation there.
- Never add the nested checkout as tracked files or a submodule. Keep agent
  transcripts, planning notes, and audit logs out of the code repository.

# Branches and releases

- Use `devel` as the working and GitHub default branch in both repositories,
  tracking `origin/devel`.
- Create temporary feature branches in `rxvt-unicode/` as needed; integrate
  completed work into that fork's `devel` branch.
- Use the fork's `devel` branch to build and publish installable `.deb` release
  packages. Keep packaging and CI changes in the code repository.
- Prepare focused feature PRs for upstream when needed; exclude companion
  material and unrelated fork changes.

# Documentation status

`URXVT_XTERM_TMUX_FORK_HANDOFF.md` and `urxvt-xterm-compat.patch` are unreviewed
local inputs, excluded from Git. Do not commit them or apply the patch blindly.
Review their claims against the code and rewrite accepted requirements concisely
into `docs/` before committing that documentation.

Proposed feature areas are xterm interoperability, clipboard/OSC 52 support,
and tmux mouse/history integration. These drafts are not implementation approval.

# Working rules

- Inspect status, branch, remotes, and applicable instructions in both repositories
  before changes. Use explicit paths, including `git -C rxvt-unicode` for code work.
- Preserve existing user changes. Keep changes minimal and feature tracks
  independently reviewable. Stage explicit paths in the intended repository.
- Define success criteria before implementation; verify with relevant checks.
  Report results, unresolved questions, and tradeoffs without speculation.
- Keep communication and documentation concise. Use ASCII in generated code and
  documentation. Add brief, lower-case comments for non-obvious code; use `## `
  for comment lines in shell, R, and other languages using `#` comments.
- Push committed substantial work or setup to `origin` in both repositories
  before ending the task, and verify the remote refs.
- When an implementation plan is approved, record it before implementation in
  `audit/plan_<YY-MM-DD_hh-mm>_<sessionID>.md`; use the Git branch or process ID
  if the session ID is unavailable. Keep `audit/` untracked unless requested.
- After multi-step implementation, offer an `audit/work_...md` log containing only
  successful commands and successful non-trivial actions.
- Put reviewed companion documentation in `docs/`. For short tables, prefer
  fenced ASCII tables, generate padding with a formatter, and verify alignment.

## Codebase Knowledge Graph

- Use the project-scoped `codebase-memory-mcp` tools for code requests that
  benefit from structural context: architecture, symbol discovery, callers,
  callees, dependencies, impact analysis, dead code, refactor scope, and
  unfamiliar code paths.
- The code index root is `/home/gpertea/work/urxvt/rxvt-unicode`, the
  independent source checkout. Do not index the companion root: its ignore
  rules exclude the source checkout.
- At first use, call `list_projects`. If that source checkout is absent or
  stale, call `index_repository` with its exact root, then check `index_status`.
- Prefer `search_graph`, `trace_path`, and `get_code_snippet` for discovery.
  Use `get_architecture`, `query_graph`, and `detect_changes` when relevant.
- After finding candidate paths, call `check_index_coverage` before relying on
  graph conclusions. Verify exact source with normal file tools.
- Use `rg` directly for literals, errors, configuration, documentation,
  non-code files, or when graph results are insufficient.
- Keep activation project-scoped. Never add global MCP config, hooks,
  instructions, or skills for this server.
- If the MCP is unavailable in the current session, state that a new Codex
  session is required and continue with local tools. Never claim graph evidence
  without calling the graph tools.
